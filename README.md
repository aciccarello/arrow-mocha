## ES6 arrows vs Mocha

ES6 arrow functions have static, lexical `this` binding. This is a great
feature, but it prevents them from playing nicely with Mocha, because
Mocha provides the test context using dynamic `this` binding:

```js
before(function() {
  this.myObj = new MyAwesomeThing()
})
it('some test', function() {
  console.log('myObj is:', this.myObj) // MyAwesomeThing {}
})
```

This will not work for arrow functions:

```js
it('some test', () => console.log('myObj is:', this.myObj)) // undefined, `this` points to the global object
```

This little library provides a set of patched Mocha functions (`it`,
`before`, `after`, `beforeEach` and `afterEach`) that pass the context
as the first argument to your arrows:

```js
before(t => {
  t.myObj = new MyAwesomeThing()
})
it('some test', t => console.log('myObj is:', t.myObj)) // MyAwesomeThing {}
```

This is done by wrapping each arrow into the usual function, obtaining the
context through `this` and passing it to the first argument of the arrow.

## Example

```js

// This line is important, otherwise the magic will not work!
// Is is commented because it breaks GitHub syntax highlighting.
//
// import { it, before, after, beforeEach, afterEach } from 'arrow-mocha'


describe('The functions imported on the previous line decorate the corresponding Mocha functions', () =>
{
  describe('so that the Mocha test context gets passed to the first argument', () => {
    before(t => {
      t.some = 'value'
    })
    it('like this', t => assert.equal(t.some, 'value'))
  })


  describe('this works for async tests too:', () => {
    const delay = (ms, value) => new Promise(resolve => {
      setTimeout(resolve, ms, value)
    })
    before(t => {
      return delay(10, 'value').then(v => {
        t.another = v
      })
    })
    describe('when a test/hook returns a Promise', () => {
      it('the context is passed to the first argument', t => delay(10).then(() => {
        assert.equal(t.another, 'value')
      }))
    })
  })


  describe('when an async test/hook requires a callback,', () => {
    const delay = (ms, fn) => {
      setTimeout(fn, ms)
    }
    before((t, done) => {
      return delay(10, () => {
        t.third = 'value'
        done()
      })
    })
    describe('declare it as the second argument;', () => {
      it('the context will be passed to the first arg', (t, done) => delay(10, () => {
        assert.equal(t.third, 'value')
        done()
      }))
    })
  })
})
```

## Licence

MIT

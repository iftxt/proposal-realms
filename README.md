# ECMAScript spec proposal for Realms API

## Status

### Current Stage

 * __Stage 1__

### Champions

 * @dherman
 * @caridy
 * @erights

### Spec Text

You can view the spec rendered as [HTML](https://rawgit.com/caridy/proposal-realms/master/index.html).

# Realms

## History

* worked on this during ES2015 time frame, so never went through stages process
* got punted to later (rightly so!)
* goal of this proposal: resume work on this, reassert committee interest via advancing to stage 1
* original idea from @dherman: [What are Realms?](https://gist.github.com/dherman/7568885)

## Intuitions

* sandbox
* iframe without DOM
* principled version of Node's `'vm'` module
* sync Worker

## Use cases

* security isolation (with synchronous but coarse-grained communication channel)
* plugins (e.g., spreadsheet functions)
* in-browser code editors
* server-side rendering
* testing/mocking (e.g., jsdom)
* in-browser transpilation

## Examples

### Example: simple realm

```js
let realm = new Realm();

let outerGlobal = window;
let innerGlobal = realm.global;

let f = realm.evalScript("(function() { return 17 })");

f() === 17 // true

Reflect.getPrototypeOf(f) === outerGlobal.Function.prototype // false
Reflect.getPrototypeOf(f) === innerGlobal.Function.prototype // true
```

### Example: simple subclass

```js
class EmptyRealm extends Realm {
  constructor(...args) { super(...args); }
  init() { /* do nothing */ }
}
```

### Example: DOM mocking

```js
class FakeWindow extends Realm {
  init() {
    super.init(); // install the standard primordials
    let global = this.global;

    global.document = new FakeDocument(...);
    global.alert = new Proxy(fakeAlert, { ... });
    ...
  }
}
```

### Example: language hooks

```js
const r = new Realm({
  evalHook(sourceText) {
    return compile(sourceText);
  },
  importHook(referrerNamespace, specifier) {
    ...
  }
});
const result = r.eval('eval("1")'); // calls compile('1'), and evaluate the returned value
const ns = r.eval('import("foo")'); // where referrerNamespace is null, and specifier is "foo"
```

### Example: custom direct evaluation

```js
const r = new Realm({
  isDirectEvalHook(func) {
    return func === r.customEval;
  },
});
r.customEval = function (sourceText) {
  return 3;
}
const source = `
  let x = 1;
  (function foo() {
    let x = 2;
    eval('x');      // yields 2
    (0,eval)('x');  // yields 3
  })();
`;
r.eval(source);
```

These example demonstrate how to fully customize the direct and indirect evaluations.

### Example: JS dialects with ES6 Realms

 * https://gist.github.com/dherman/9146568

## Shim

A shim implementation of the Realm API can be found [here](shim/README.md).

And you can play around with the Shim [here](https://rawgit.com/caridy/proposal-realms/master/shim/examples/simple.html).

## Contributing

### Updating the spec text for this proposal

The source for the spec text is located in [spec/index.emu](spec/index.emu) and it is written in
[ecmarkup](https://github.com/bterlson/ecmarkup) language.

When modifying the spec text, you should be able to build the HTML version in
`index.html` by using the following command:

```bash
npm install
npm run build
open index.html
```

Alternative, you can use `npm run watch`.

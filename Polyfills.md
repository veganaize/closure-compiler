# Polyfills

Closure Compiler defines polyfills for newer classes and methods present in ES6+ versions of the ECMAScript spec. The compiler does not polyfill ES5 methods like [`Object.defineProperty`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty). It also does not polyfill browser methods that are not part of the ECMAScript standard.

See https://github.com/google/closure-compiler/wiki/Supported-features for a complete list of available polyfills.

## Using polyfills

SIMPLE and ADVANCED optimization levels in Closure compiler contain support for automatically adding polyfills. The compiler will only add polyfills that are present in the input language but not the output language. The compiler detects what polyfills are needed based on the input code, and it will avoid injecting unnecessary polyfills.

Injection of polyfills may be disabled with `--rewrite_polyfills=false`. Polyfills are also not injected if using BUNDLE or WHITESPACE_ONLY.

Polyfill injection is also not automatically triggered by "guarded" usages of classes or methods. For example, the compiler will not inject the `Symbol` polyfill given `if (typeof Symbol === 'function') { use(Symbol('foo')); }`, so it is safe to write code conditionally using `Symbol` if you don't want the binary size impact of including the `Symbol` polyfill.

Polyfill implementations may be found in https://github.com/google/closure-compiler/tree/master/src/com/google/javascript/jscomp/js/es6.

## Polyfill isolation

Polyfill isolation is an output mode that may optionally be used alongside polyfill injection.

Polyfill isolation was motivated by two related issues.
 - sometimes, the existence of Closure polyfills on the page would cause other non-Closure-compiled code to break, because of conflicting assumptions in a polyfill implementation used by third-party code
 - sometimes, Closure-compiled code would break, because of existing polyfills on the page that violated some assumption Closure Compiler makes

These issues were generally seen by projects compiling code for inclusion as a script on third-party websites, along with arbitrary JavaScript not under their control.

Polyfill isolation mode attempts to solve these problems by "isolating" Closure polyfills and code from other code & polyfills. It is not intended to protect against malicious actors; it is instead intended to solve cases where other polyfill implementations are either buggy or (more likely) make conflicting assumptions.

Enable with `--isolate_polyfills` or `options.setIsolatePolyfills(true);`.

This option is available starting in v20200504. A major caveat (as of 2020-05-07) is that support for isolation of the `Symbol` polyfill is not expected until the following release.

### Current limitations

 - polyfilled classes and methods referenced via bracket access will not be polyfilled at runtime. So `window.Promise` is acceptable but `window['Promise']` is not.
 - custom implementations of polyfills are not supported. For example, given `window.Promise = MyPromise; use(new Promise(resolve, reject));`, the `Promise` instantation will refer to the compiler polyfill and not `MyPromise`.
 - as of v20200504, polyfill isolation is supported for all polyfills _except_ `Symbol`. Support for `Symbol` is already implemented at head and in the internal Google release, and it will land in the next release.

These are not technical limitations and could be supported given a compelling use case.

### Other caveats

 - enabling `--isolate_polyfills` will increase the output binary size, even with ADVANCED_OPTIMIZATIONS. This is because the current implementation requires rewriting all potential usages of a polyfill into either a function call or bracket access, and the library code for defining polyfills is more complicated than the default, un-isolated version.
 - polyfill isolation mode uses polyfills for ES2016+ classes/methods even when running on a modern browser with native support. This is because we have not invested in a way to reliably distinguish between native implementations versus polyfills. However, the native implementation of ES6 features (like `Promise` and `String.prototype.startsWith`) is used if and only if Symbol is also native. This is detected by checking `typeof Symbol('x') === 'symbol'`.

### Implementation details

The compiler implements polyfill isolation by first, changing how the compiler's JS runtime libraries inject polyfills, and second, rewriting all potential use sites of polyfills in the compiled code to refer to the compiled polyfills instead of the native object.

Concretely, here's what the output code looks like.

Input:

```
console.log(maybeString.startsWith('x'));
const sym = Symbol('project-id');
```

Output:

```
console.log($jscomp.lookupPolyfilledValue(maybeString, 'startsWith').call(maybeString, 'x'));
const sym = $jscomp.polyfills['Symbol'];
```

Polyfilled globals like `Symbol`, `Promise`, and `globalThis` are defined on a separate object instead of on `window`. Polyfilled methods like `Array.from` or `String.prototype.startsWith` are defined as non-enumerable properties on their owner object under a unique name or Symbol.
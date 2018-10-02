While Closure Library does not currently have any ES6 modules, the Closure Compiler and the Library's Debug Loader are compatible with ES6 modules. For more information on the Debug Loader support see TODO.

# Referencing Closure Files from ES6 Modules

ES6 modules can use `goog.require` like `goog.module`s to reference Closure files:

```
const math = goog.require('goog.math');

export const MY_CONSTANT = math.sum(1, 1, 2, 3, 5, 8, 13, 21);
```

# Referencing ES6 Modules from Closure Files

For those that have projects that are currently Closure files (e.g. `goog.provide`, `goog.module`) and wish to migrate to ES6 modules, it is possible to do so incrementally.

The Closure Library has the function `goog.declareModuleId(/** string */ id)`, which, when called in an ES6 module, associates the ES6 module with a `goog.module`-like id (e.g. something that can be `goog.require`'d, `goog.module.get`d, `goog.forwardDeclare`d, and `goog.requireType`d, and does not create any global symbols). `goog.require`s for these symbols will return the entire module object, as if bound by `import * as`.

```
goog.declareModuleId('example.es6');

export const MY_CONSTANT = 'Hello!';
```

```
goog.module('old.module');

const es6 = goog.require('example.es6');

console.log(es6.MY_CONSTANT);
```

`goog.declareModuleId` should only be used as a temporary measure to help migrate to ES6 modules, and should be removed once the migration is complete. For a more detailed migration guide see [Migrating from goog.modules to ES6 modules](https://github.com/google/closure-compiler/wiki/Migrating-from-goog.modules-to-ES6-modules).
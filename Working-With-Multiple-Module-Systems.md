**This Page is a Work In Progress**

The compiler understands several types of modules including:

 * ES6
 * Common JS
 * AMD (only certain signatures for define calls are supported)
 * Closure-Library (goog.module)

During compilation, the compiler normalizes and inlines all of these disparate module by rewriting them into a form where the remaining passes can understand and optimize them fully.

It is possible to work with multiple module systems in a single compilation.

## ES6 Modules

In ES6 syntax, a module is defined as a file. ES6 modules use `import` and `export` statements to include and expose symbols.

```JavaScript
import MySymbol from 'path/to/module';
export MySymbol;
```

### Importing CommonJS Modules

ES6 modules can import from commonjs modules:

```JavaScript
// Note - currently you can only import '*' from a CommonJS module
// In this case we assume that the commonjs module has a "module.exports = { MySymbol: foo }" statement
import * from 'path/to/commonjs/module';
console.log(MySymbol);
```

### Using Closure-Library Dependencies

*How does/should this work?*

## CommonJS Modules
CommonJS modules are most widely used in NodeJS. CommonJS modules use `var MySymbol = require('module-path')` to import and add properties to a global `module` symbol to export.

```JavaScript
var MySymbol = require('path/to/module');
module.exports.MySymbol = MySymbol;
```

### Importing ES6 Modules

CommonJS modules can import from ES6 modules:

```JavaScript
var MySymbol = require('path/to/module');
```

### Using Closure-Library Dependencies

*How does/should this work?*

## Dependency Management and Entry Points
Currently the compiler requires flags to enable each of these module systems:

ES6 Modules are supported via the language_in flag:

    --language_in=ECMASCRIPT6

CommonJS Modules are enabled via a boolean flag:

    --process_common_js_modules

Closure-Library style code (goog.require/provide) is understood by default.

### Entry Points and Dependency Tree Discovery

Closure-compiler can determine which source files should be included in a compilation by tracing dependencies in each type of module. However, each module currently uses it's own entry point flag to determine starting points for dependency searching. In each case, the flag may be specified multiple times to support multiple entry points.

ES6 Modules

   *no flag currently - this needs added*

CommonJS

  --common_js_entry_module='path/to/module'

Closure Library

  --closure_entry_point='goog.provide.namespace'

*Can we standardize these to a common entry point flag?*
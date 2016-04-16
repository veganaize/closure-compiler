*Note: CommonJS module interop requires the 20160125 release or newer.*

Closure Compiler recognizes and several JS module systems, including:

 * [ES6 Modules](https://github.com/nzakas/understandinges6/blob/master/manuscript/13-Modules.md)
 * Common JS
 * [Closure-Library (goog.module)](https://google.github.io/closure-library/api/namespace_goog.html#module)

During compilation, the compiler normalizes and inlines all of these disparate module by rewriting them into a form where the remaining passes can understand and optimize them fully. You can use multiple modules types in a single compilation.

CommonJS and ES6 Modules are file based and a module is imported by it's path. Paths must be absolute or relative and omit the file extension:

 * `import foo from './folder/source'`
 * `import foo from '/folder/source'`
 * `require('./folder/source')`
 * `require('/folder/source')`

The root folder is the current working directory of the compiler when executed.

## Restrictions

Each module must only use a single type of import or export. Unless otherwise noted, a module may not mix import or export methods within the same file.

## ES6 Modules

ES6 Module support is enabled when the `--language_in` flag is used to specify an input language of ECMASCRIPT6 or higher.

ES6 modules use `import` and `export` statements to include and expose symbols.

```JavaScript
import MySymbol from 'path/to/module';
export MySymbol;
```

### Importing CommonJS Modules into an ES6 Module

```JavaScript
// CommonJS modules have a single export and are therefore imported as namespaces.
// In this case we assume that the commonjs module has a
// "module.exports = { MySymbol: foo }" statement
import * as myModule from 'path/to/commonjs/module';
console.log(myModule.MySymbol);
```

### Importing goog.module code into an ES6 Module

```JavaScript
// The compiler recognizes a special syntax to identify goog.module paths
import Event from 'goog:goog.events';
```

## CommonJS Modules

CommonJS module support is enabled in the compiler when the `--process_common_js_modules` flag is specified.

### Importing ES6 modules into a CommonJS Module

```JavaScript
// ES6 modules have have multiple export properties.
// You must reference the desired property.
// In this case we assume that the ES6 module has an
// "export default MySymbol;" statement
var MySymbol = require('path/to/es6/module').default;
console.log(MySymbol);
```

### Importing goog.module code into a CommonJS Module

```JavaScript
const m = goog.require("my.ns.Thing"); // goog.modules can always be require'd
// (but not goog.provide'd scripts).
```

## goog.module

goog.module support is always enabled in the compiler.

### Importing CommonJS Module code into a goog.module

```JavaScript
const m = require('path/to/cjs/module');
```
> To be confirmed

### Importing ES6 Module code into a goog.module

```JavaScript
var MySymbol = require('path/to/es6/module').default; // as above
```
> To be confirmed

## Dependency Management

The compiler can utilize import and export statements from any of the module systems to sort files and drop unreferenced files. See [Managing Dependencies](https://github.com/google/closure-compiler/wiki/Managing-Dependencies).

## Output Modules

Closure Compiler can split the output of a compilation into multiple files to support dynamic loading. This process uses the `--module` flag, but is not related to the above JS modules as the output does not specify the type of module. See [how do I split my javascript into modules](http://stackoverflow.com/questions/10395810/how-do-i-split-my-javascript-into-modules-using-googles-closure-compiler/10401030#10401030)

## Restrictions of Google modules

Note this is an incomplete summary(by trial and error) of restrictions of goog module system

1. Google module does not allow toplevel throw, Closure Compiler will spit out an error instead of a warning

```js
// file xx.js
goog.module('my.module')
throw 3 ;
``` 

Like code above, Closure compiler will refuse to bundle it
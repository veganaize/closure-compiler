Closure Compiler recognizes and several JS module systems, including:

 * [ES6 Modules](https://github.com/nzakas/understandinges6/blob/master/manuscript/14-Modules.md)
 * Common JS
 * [Closure-Library (goog.module)](https://google.github.io/closure-library/api/namespace_goog.html#module)

During compilation, the compiler normalizes and inlines all of these disparate module by rewriting them into a form where the remaining passes can understand and optimize them fully. You can use multiple versions is possible to work with multiple module systems in a single compilation.

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

### Importing CommonJS Modules from an ES6 Module

```JavaScript
// CommonJS modules have a single export and are therefore imported as namespaces.
// In this case we assume that the commonjs module has a
// "module.exports = { MySymbol: foo }" statement
import * as myModule from 'path/to/commonjs/module';
console.log(myModule.MySymbol);
```

### Importing goog.module code from an ES6 Module

```JavaScript
// The compiler recognizes a special syntax to identify goog.module paths
import Event from 'goog:goog.events';
```

## CommonJS Modules

CommonJS module support is enabled in the compiler when the `--process_common_js_modules` flag is specified.

### Importing ES6 modules from a CommonJS Module

```JavaScript
// ES6 modules have have multiple export properties. You must reference the desired property.
// In this case we assume that the ES6 module has a "export default MySymbol;" statement
var MySymbol = require('path/to/es6/module').default;
console.log(MySymbol);
```

### Importing goog.module code from a CommonJS Module

```JavaScript
// For goog.module, fall back go using goog.require
// ??
```

## goog.module

goog.module support is always enabled in the compiler.

> How do we import other modules within a Closure-Library style file? Must we resort to using the normalized
module identifier? If so, we should find a better way to do this.

## Dependency Management

The compiler can utilize import and export statements from any of the module systems to sort files and drop unreferenced files. See [Managing Dependencies](https://github.com/google/closure-compiler/wiki/Managing-Dependencies).
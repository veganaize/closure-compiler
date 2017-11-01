Closure Compiler recognizes several JS module systems, including:

 * [ES6 Modules](https://github.com/nzakas/understandinges6/blob/master/manuscript/13-Modules.md)
 * Common JS
 * [Closure-Library (goog.module)](https://github.com/google/closure-library/wiki/goog.module:-an-ES6-module-like-alternative-to-goog.provide)

During compilation, the compiler normalizes and inlines all of these disparate module by rewriting them into a form where the remaining passes can understand and optimize them fully. This is equivalent functionality to other module bundlers. You can use multiple modules types in a single compilation.

## Module Resolution Mode

CommonJS and ES6 Modules are file based and a module is imported by its path. How the path is interpreted depends on the resolution mode specified.

The resolution mode is controlled by the `--module_resolution` flag. This flag was introduced after the 20161201 release.

### NODE Resolution Mode
Modules which do not begin with a "." or "/" character are looked up from the appropriate node_modules folder. This mode supports importing a directory as well as treating a JSON file as a module. See the [node module resolution algorithm](https://nodejs.org/api/modules.html#modules_all_together) for full details.

The compiler does not discover and add source files. Users must pass the full set of possible source files to the compiler. In addition, any package.json files should also be provided as source files via the `--js` flag.

Projects which fully make use of modules should make use of the [dependency management flags](https://github.com/google/closure-compiler/wiki/Managing-Dependencies).

Example import statements for NODE mode:

 * `import foo from './folder/source.js'`
 * `import foo from '../folder/source.js'`
 * `import foo from '/folder/source'`
 * `import foo from 'folder/source'` - this import would be looked for in the appropriate node_modules folder
 * `require('./folder/source')`
 * `require('/folder/source')`
 * `require('folder/source')`

### BROWSER Resolution Mode

Modules which do not begin with a "." or "/" character are not supported. Module import statements must always specify the file extension.

The compiler does not discover and add source files. Users must pass the full set of possible source files to the compiler. 

Projects which fully make use of modules should make use of the [dependency management flags](https://github.com/google/closure-compiler/wiki/Managing-Dependencies).

Example import statements for BROWSER mode:

 * `import foo from './folder/source.js'`
 * `import foo from '../folder/source.js'`
 * `import foo from '/folder/source.js'`
 * `require('./folder/source.js')`
 * `require('/folder/source.js')`
 * `require('../folder/source.js')`

## Restrictions

CommonJS and Goog modules are recognized by its export mechanism. ES6 modules are recognized be either the `import` or `export` keyword. A single file can only use one type of export. The following statements must not be mixed in the same file:

 * ES6 export: `export default Foo /* and variants */` or `import foo from '/path/to/module.js'`
 * CommonJS export: `module.exports = Foo` and `export.Foo = Foo`;
 * goog.module: `goog.module('foo'); export.Foo = Foo;`

## ES6 Modules

ES6 Module support is enabled when the `--language_in` flag is used to specify an input language of ECMASCRIPT6 or higher.

ES6 modules use `import` and `export` statements to include and expose symbols.

```JavaScript
import MySymbol from 'path/to/module.js';
export MySymbol;
```

### Uncompiled Support for ES6 Modules

ES6 module loading is still at very early stages of development. Currently, ES6 modules require transpilation to another module format and therefore are not supported by browsers in uncompiled mode. This is not a limitation of the compiler, but one of the environment.

## CommonJS Modules

CommonJS module support is enabled in the compiler when the `--process_common_js_modules` flag is specified.

CommonJS modules are rewritten to be in a single namespace and imports are hoisted. Hoisting imports allows impressive code analysis and optimizations, but does so at the expense of compatibility. Not all CommonJS modules will be compatible with this rewriting.

The compiler recognizes several variants of the Universal Module Definition and will simplify these into just a CommonJS module.

### Uncompiled Support for CommonJS Modules

CommonJS modules define the [require.ensure](http://wiki.commonjs.org/wiki/Modules/Async/A) signature to support asynchronous loading. A module loader [such as InjectJS](http://www.injectjs.com/docs/0.7.x/cjs/require.ensure.html) can be used to fully support CommonJS modules in uncompiled mode. Such loaders are not needed after compilation as the compiler will remove the `require.ensure` call during compilation.

Both the dependency array and the callback function arguments for `require.ensure` must be literals. The compiler does not support using variables as arguments to `require.ensure` calls.

## goog.module

goog.module support is always enabled in the compiler. [Full goog.module documentation](https://github.com/google/closure-library/wiki/goog.module:-an-ES6-module-like-alternative-to-goog.provide)

### Limitations of goog.module

Note this is an incomplete summary(by trial and error) of restrictions of goog module system

1. Google module does not allow toplevel throw, Closure Compiler will spit out an error instead of a warning

```js
// file xx.js
goog.module('my.module')
throw 3 ;
// Closure compiler will not bundle it
``` 

A more real world case:

```js
// file yy.js
goog.module('my.module')
var f = function(){..}
try { f x}
catch(e){throw 'impossible'}
// Closure compiler still refuse to bundle such code
```

Like code above, Closure compiler will refuse to bundle it

### Uncompiled Support for goog.modules

Closure-Library has support for loading Closure dependencies (including goog.modules) uncompiled.

## Module Interoperation

A file can only be recognized as one type of module, however you may use CommonJS or Goog module imports in any file type (including scripts). ES6 'import' statements are restricted by spec to an ES6 module.

It is recommended that a module is imported using the method native to its type. CommonJS modules should be imported using `require` calls, goog.module should be imported with `goog.require` calls. However since ES6 `import` statements can only be used in ES6 modules, scripts and other module types can't use the `import` keyword.

It is possible to mix import statements, however this method is not generally recommended except for the case where importing an ES6 module into another script or module type.

```js
// CommonJS import of ES6 module
var Foo = require('/path/to/es6/module/foo').default;
```

## Dependency Management

The compiler can utilize import and export statements from any of the module systems to sort files and drop unreferenced files. See [Managing Dependencies](https://github.com/google/closure-compiler/wiki/Managing-Dependencies).

## Type References

The preferred method to reference a type from another module is to import it and utilize the imported alias as a type reference:

```
const foo = require('./foo')';

/** @param {foo.Foo} Foo */
function(Foo) {}
```

This type of reference only works in the module root or global scope. Inside a nested scope, the alias will not be dereferenced by the compiler. In these cases, type annotations may specify a module path:

```
/** @param {./foo.Foo} Foo */
function(Foo) {}
```

Using the rewritten module name is highly discouraged. This name is an internal representation by the compiler and may be changed at any time.

## Code Splitting (Output Modules)

Closure Compiler can split the output of a compilation into multiple files to support dynamic loading. This process uses the `--module` flag, but is not related to the above JS modules as the output does not specify the type of module. See [how do I split my javascript into modules](http://stackoverflow.com/questions/10395810/how-do-i-split-my-javascript-into-modules-using-googles-closure-compiler/10401030#10401030)
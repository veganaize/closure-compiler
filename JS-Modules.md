Closure Compiler recognizes several JS module systems, including:

 * [ES Modules](https://github.com/nzakas/understandinges6/blob/master/manuscript/13-Modules.md)
 * Common JS
 * [Closure-Library (goog.module)](https://github.com/google/closure-library/wiki/goog.module:-an-ES6-module-like-alternative-to-goog.provide)

During compilation, the compiler normalizes and inlines all of these disparate module by rewriting them into a form where the remaining passes can understand and optimize them fully. This is equivalent functionality to other module bundlers. You can use multiple modules types in a single compilation.

## Dynamic Import Expressions
Dynamic import expressions (ex: `import('./my-module.js')`) allow ES modules to be loaded asynchronously from any input type - scripts or module.

### Resolving the Import Specifier
When the import specifier is a string literal (ex `import('./my-module.js')`), the compiler will statically resolve the module and enable full type checking on the returned promise. If the specifier can’t be resolved, the compiler will emit a warning. This methodology is similar to how other JS build tools statically analyze dynamic imports. The compiler will update the specifier to reference the correct output bundle.

Dynamic import specifiers can also be arbitrary expressions (ex `import(modulePathVar)`). In this case, the compiler can’t resolve the module. The returned promise will resolve to an unknown type and no warning will be issued. The compiler assumes you know what you are doing here and the imported module is external.

### Transpiling Dynamic Import
Dynamic imports can’t be cleanly transpiled. While there are polyfills which utilize `<script type="module">` insertion in the DOM, most of them require unsafe-eval or data url manipulations that might run afoul of a content security policy. Closure Compiler has opted not to even attempt a polyfill.

Instead, Closure-Compiler will alias (or rename) `import()` expressions during compilation into a function call (ex `import_()`) using the new `--dyanmic_import_alias=import_` command line flag. The presence of this flag instructs the compiler to allow dynamic imports even when the output language is lower than ECMASCRIPT_2020. The compiler will expect you to provide a definition (external or in source) for this aliased function. This way you can choose how you want to polyfill dynamic imports.

## Module Resolution Mode

CommonJS and ES modules are file based and a module is imported by its path. How the path is interpreted depends on the resolution mode specified.

The resolution mode is controlled by the `--module_resolution` flag. This flag was introduced after the 20161201 release.

### NODE Resolution Mode
Modules which do not begin with a "." or "/" character are looked up from the appropriate node_modules folder. This mode supports importing a directory as well as treating a JSON file as a module. See the [node module resolution algorithm](https://nodejs.org/api/modules.html#modules_all_together) for full details.

The compiler does not discover and add source files. Users must pass the full set of possible source files to the compiler. In addition, any package.json files should also be provided as source files via the `--js` flag. You can use the [closure-calculate-chunks utility](https://github.com/chadkillingsworth/closure-calculate-chunks) to discover the correct files to pass to the compiler.

Projects which fully make use of modules should make use of the [dependency management flags](https://github.com/google/closure-compiler/wiki/Managing-Dependencies).

Example import statements for NODE mode:

 * `import foo from './folder/source.js'`
 * `import foo from '../folder/source.js'`
 * `import foo from '/folder/source'`
 * `import foo from 'folder/source'` - this import would be looked for in the appropriate node_modules folder
 * `require('./folder/source')`
 * `require('/folder/source')`
 * `require('folder/source')`

**Note:** package.json files are required to properly resolve node modules. The appropriate package.json files should be passed to the compiler as source files (using the `--js` flag). These files are only used to resolve modules and won't affect the output size of the compipiled js.

### BROWSER Resolution Mode

Modules which do not begin with a "." or "/" character are not supported. Module import statements must always specify the file extension.

The compiler does not discover and add source files. Users must pass the full set of possible source files to the compiler. You can use the [closure-calculate-chunks utility](https://github.com/chadkillingsworth/closure-calculate-chunks) to discover the correct files to pass to the compiler.

Projects which fully make use of modules should make use of the [dependency management flags](https://github.com/google/closure-compiler/wiki/Managing-Dependencies).

Example import statements for BROWSER mode:

 * `import foo from './folder/source.js'`
 * `import foo from '../folder/source.js'`
 * `import foo from '/folder/source.js'`
 * `require('./folder/source.js')`
 * `require('/folder/source.js')`
 * `require('../folder/source.js')`

## Restrictions

CommonJS and Goog modules are recognized by its export mechanism. ES modules are recognized be either the `import` or `export` keyword. A single file can only use one type of export. The following statements must not be mixed in the same file:

 * ES export: `export default Foo /* and variants */` or `import foo from '/path/to/module.js'`
 * CommonJS export: `module.exports = Foo` and `export.Foo = Foo`;
 * goog.module: `goog.module('foo'); export.Foo = Foo;`

## ES Modules

ES module support is enabled when the `--language_in` flag is used to specify an input language of ECMASCRIPT_2015 or higher.

ES modules use `import` and `export` statements to include and expose symbols.

```JavaScript
import MySymbol from 'path/to/module.js';
export MySymbol;
```

For information on interop with Closure files, see [ES modules and Closure interop](https://github.com/google/closure-compiler/wiki/ES6-modules-and-Closure-interop).

## CommonJS Modules

CommonJS module support is enabled in the compiler when the `--process_common_js_modules` flag is specified.

CommonJS modules are rewritten to be in a single namespace and imports are hoisted. Hoisting imports allows impressive code analysis and optimizations, but does so at the expense of compatibility. Not all CommonJS modules will be compatible with this rewriting.

The compiler recognizes several variants of the Universal Module Definition and will simplify these into just a CommonJS module.

### Uncompiled Support for CommonJS Modules

Common JS modules are not natively supported in browsers. Several basic dev server transformations exist to transpile them into ES modules for use in a browser.

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

A file can only be recognized as one type of module, however you may use CommonJS or Goog module imports in any file type (including scripts). ES module 'import' statements are restricted by spec to an ES module.

It is recommended that a module is imported using the method native to its type. CommonJS modules should be imported using `require` calls, goog.module should be imported with `goog.require` calls. However since ES `import` statements can only be used in ES modules, scripts and other module types can't use the `import` keyword.

It is possible to mix import statements, however this method is not generally recommended except for the case where importing an ES module into another script or module type.

```js
// CommonJS import of ES module
var Foo = require('/path/to/es/module/foo').default;
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

Closure Compiler can split the output of a compilation into multiple files to support dynamic loading. This process uses the `--chunk` flag. See [Chunk Output for Dynamic Loading](https://github.com/google/closure-compiler/wiki/Chunk-output-for-dynamic-loading) for details on splitting code into output chunks.
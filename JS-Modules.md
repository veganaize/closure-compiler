*Note: Full module interop requires the 20160517 release or newer.*

Closure Compiler recognizes several JS module systems, including:

 * [ES6 Modules](https://github.com/nzakas/understandinges6/blob/master/manuscript/13-Modules.md)
 * Common JS
 * [Closure-Library (goog.module)](https://google.github.io/closure-library/api/namespace_goog.html#module)

During compilation, the compiler normalizes and inlines all of these disparate module by rewriting them into a form where the remaining passes can understand and optimize them fully. This is equivalent functionality to other module bundlers. You can use multiple modules types in a single compilation.

CommonJS and ES6 Modules are file based and a module is imported by its path. Paths must be absolute or relative and omit the file extension:

 * `import foo from './folder/source'`
 * `import foo from '/folder/source'`
 * `require('./folder/source')`
 * `require('/folder/source')`

When using the grunt/gulp plugins, the root folder is the current working directory of the compiler when executed. Otherwise, the root directory is the root of the filesystem.

## Restrictions

A module is recognized by its export mechanism. A single file can only use one type of export. The following statements must not be mixed in the same file:

 * ES6 export: `export default Foo // and variants`
 * CommonJS export: `module.exports = Foo` and `export.Foo = Foo`;
 * goog.module: `goog.module('foo'); export.Foo = Foo;`

Module exports are also treated as constants in the compiler. This enables many existing optimizations.

## ES6 Modules

ES6 Module support is enabled when the `--language_in` flag is used to specify an input language of ECMASCRIPT6 or higher.

ES6 modules use `import` and `export` statements to include and expose symbols.

```JavaScript
import MySymbol from 'path/to/module';
export MySymbol;
```

### Uncompiled Support for ES6 Modules

ES6 module loading is still at very early stages of development. Currently, ES6 modules require transpilation to another module format and therefore are not supported by browsers in uncompiled mode. This is not a limitation of the compiler, but one of the environment.

## CommonJS Modules

CommonJS module support is enabled in the compiler when the `--process_common_js_modules` flag is specified.

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

While modules may only contain one type of export, they may freely mix import statements. It is recommended that a module is imported using the method native to it's type. CommonJS modules should be imported using `require` calls, ES6 modules should be imported with `import` statements. goog.module should be imported with `goog.require` calls.

It is possible to mix import statements, however this method is not generally recommended except for the case where importing an ES6 module would require transpilation of the importer as well.

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
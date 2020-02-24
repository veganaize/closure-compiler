*Note: These flags were redesigned in the 20160125 release. This document describes that release and newer.*

The browser processes JavaScript files serially. So if you have a lot of JavaScript files you'll need some way to make sure they get executed in the right order. Closure Compiler has similar restrictions in that compilations must process files in the correct order so that types are defined before they are used.

## Specifying Dependencies Between Files

Closure Compiler can use dependency information from the following sources:

 * Closure-Library `goog.require`, `goog.module`, and `goog.provide` statements
 * ES6 Module `import` and `export` statements
 * CommonJS `require` and `module.export` or `exports` statements when the `--process_common_js_modules` flag is specified

For any of the above cases, internally the compiler normalizes the dependency information to always be the Closure Library style `goog.provide` for exporting and `goog.require` for importing. See [JS Modules](https://github.com/google/closure-compiler/wiki/JS-Modules) for more information on the different module types and how they interact.

The remaining portions of this document uses the Closure Library style `goog.provide` and `goog.require` statements, but apply to any of the module systems.

## Using Closure Compiler to auto-sort files

Suppose you have three files.

```js
// File 1 - icecream.js
goog.provide('ice.cream');
```
----
```js
// File 2 - cone.js
goog.provide('waffle.cone');
```
----
```js
// File 3 - shop.js
goog.provide('ice.cream.Shop');

goog.require('ice.cream');
goog.require('waffle.cone');
```
----

You want to compile your ice cream shop, so you pass it to the compiler.

    $ java -jar compiler.jar --js shop.js
    
    shop.js:4: ERROR - required "ice.cream" namespace never provided
    goog.require('ice.cream');
                ^
    
    shop.js:5: ERROR - required "waffle.cone" namespace never provided
    goog.require('waffle.cone');
                ^
    
    2 error(s), 0 warning(s)

What happened? The compiler noticed that shop.js requires ice.cream and waffle.cone, but it did not know where to find those files. You have to pass them in explicitly.

    $ java -jar compiler.jar --js shop.js --js icecream.js --js cone.js
    
    var ice={};ice.cream={};var waffle={};waffle.cone={};ice.cream.Shop={};

Now the compiler knows where ice.cream and waffle.cone are, but we passed them out of order. Closure Compiler will notice that shop.js has to come last, and will re-arrange the files before it compiles them. If you are using the Java API, this behavior is controlled by [DependencyOptions' DependencySorting](http://closure-compiler.googlecode.com/git/javadoc/com/google/javascript/jscomp/DependencyOptions.html) option.

There's one subtle point we should point out: Closure Compiler will rearrange the files, but it will try to maintain the *relative* order of files when there aren't any dependencies between them. In the above example, icecream.js still comes before cone.js in the compiled output, because icecream.js came before cone.js on the commandline.

### Using Wildcards to Specify Source Files

With large source sets of files, the compiler can utilize **java** glob patterns rather than listing each file as an individual flag:

    $ java -jar compiler.jar --js **.js

The compiler uses java style globs:

 * `--js src/*.js` - all js files in the src folder
 * `--js src/**.js` - all js files in the src folder and any sub folders (recursive)
 * `--js !src/**test.js` - exclude files that end in with "test.js" (recursive)

### `goog.provide` and `goog.require` as Magic Primitives

While `goog.require` and `goog.provide` are defined in Closure Library, Closure Compiler considers them magic primitives. The compiler automatically expands them into variable and property declarations. Even if the JS files you give it do not define `goog.provide`/`goog.require` anywhere, the compiler expands these symbols. Even if your JS files define completely bizarre implementations for these functions, the compiler still expands them according to the Closure Library implementation. The logic is hard-coded. 

If you want the code to work uncompiled, you should either include Closure Library's [base.js](https://github.com/google/closure-library/blob/master/closure/goog/base.js) with your code, or you should copy and paste the reference implementation.

### Using `--dependency_mode` to Drop Unreferenced Files

*This flag replaces the now deprecated `--only_closure_dependencies`  and `--manage_closure_dependencies` flags*

Suppose you don't need the ice cream shop. You just want the ice cream. If you have a lot of files, it is difficult to manually figure out if your ice cream had any dependencies. `--dependency_mode` helps with that. When you use `--dependency_mode=PRUNE`, you must specify to the compiler what the entry points of your application are. Beginning at those entry points, it will trace through the files to discover what sources are actually referenced and will drop all other files.

    $ java -jar compiler.jar --js shop.js --js icecream.js --js cone.js \
      --dependency_mode=PRUNE --entry_point=goog:ice.cream
    
    var ice={};ice.cream={};

`--dependency_mode=PRUNE` only keeps files that `goog.provide` a symbol. If you have files that do not `goog.provide` a symbol, the compiler will just skip them when this flag is used. ES6 or CommonJS module rewriting generates an a `goog.provide` statement for every ES6 or CommonJS file which is auto-detected by the presenese of `import` or `export` statements for ES6 and `require` or `module.exports` statements for CommonJS.

## Working with Files that Do Not Specify Dependencies

Suppose you want to compile a file that doesn't contain dependency information, but want the nice sorting and pruning of sources that Closure Compiler provides. There are two common solutions:

- Write a shell script that adds goog.provide/goog.require dependency information to the top of the file.
- Use `--dependency_mode=PRUNE_LEGACY`

`--dependency_mode=PRUNE_LEGACY` will **keep** any files that do not `goog.provide` symbols (whereas `--dependency_mode=PRUNE` will drop them). ES6 and CommonJS modules are rewritten to **always** have `goog.provide` statements.

For example, you would define a new file:

----
```js
// File 4 - shop-app.js
goog.require('ice.cream.Shop');
```
----

Because this file does not `goog.provide` a symbol, `--dependency_mode=PRUNE_LEGACY` will always keep it and all of its dependencies.

    $ java -jar compiler.jar --js shop-app.js --js shop.js --js icecream.js --js cone.js \
      --dependency_mode=PRUNE_LEGACY
    
    var ice={};ice.cream={};var waffle={};waffle.cone={};ice.cream.Shop={};

### Debugging Dependencies

These flag may sometimes appear to be magical. The compiler reads all your code, and then picks a subset of files to include in the compilation. How do you know which files it picked?

Fortunately, there is a flag for that.

    $ java -jar compiler.jar --js shop-app.js --js shop.js --js icecream.js --js cone.js \
      --dependency_mode=PRUNE_LEGACY --output_manifest manifest.MF
    
    ...
    
    $ cat manifest.MF
    icecream.js
    cone.js
    shop.js
    shop-app.js

When you use `--output_manifest` on the commandline, you give the flag an output file. The compiler writes a list of all the inputs processed, in sorted order, to this output file.

### FAQ

**I made a syntax error in one of my library files, and when I used `--dependency_mode=PRUNE`/`--dependency_mode=PRUNE_LEGACY`, Closure Compiler didn't catch it. Why not?**

When you use these flags, Closure Compiler looks at the dependencies and drops library files that are not required. It makes this determination very early in the process. It doesn't even parse the dropped files to check if they're syntactically valid.

If you're writing a library and you're primarily using Closure Compiler to do syntax checking on that library, then you should not use `--dependency_mode=PRUNE` or `--dependency_mode=PRUNE_LEGACY`. You should create separate build rules for syntax checking and for compiling just what's needed in your app.

**I aliased `goog.require` or CommonJS's `require` statement like `var r = goog.require;`. Now dependency sorting doesn't work. Why?**

Closure Compiler does not even parse the code to determine its dependencies. It just does a regular expression search. If your provide and require statements are not at the top of the file in the form that the compiler expects, then it will fail to find them.

There's a reason it was done this way. Loading thousands of files from disk and building parse trees for them takes some time and memory. If you only need a few files, but you're passing in hundreds of library files that will get dropped anyway, we didn't want to slow down your compilation by an order of magnitude by trying to parse all those files.

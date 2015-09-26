### Documents the --manage_closure_dependencies and --only_closure_dependencies options

The browser processes JavaScript files serially. So if you have a lot of JavaScript files you'll need some way to make sure they get executed in the right order.

Closure Library offers [a few python scripts](http://code.google.com/closure/library/docs/calcdeps.html) that help you manage dependencies between files. In the Closure Library world, each JavaScript file has a `goog.provide` call at the top for each major symbol it defines, and a `goog.require` call at the top for each symbol it needs. Closure Library's scripts use that information to determine an ordering.

Now, Closure Compiler has this feature built-in! Read on to find out more.

### Using Closure Compiler to auto-sort files

Suppose you have three files.

    // File 1 - icecream.js
    goog.provide('ice.cream');
----
    // File 2 - cone.js
    goog.provide('waffle.cone');
----
    // File 3 - shop.js
    goog.provide('ice.cream.Shop');
    
    goog.require('ice.cream');
    goog.require('waffle.cone');
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

### Wait -- aren't `goog.provide` and `goog.require` defined in Closure Library?

Yes. But Closure Compiler considers them magic primitives. The compiler automatically expands them into variable and property declarations. Even if the JS files you give it do not define `goog.provide`/`goog.require` anywhere, the compiler expands these symbols. Even if your JS files define completely bizarre implementations for these functions, the compiler still expands them according to the Closure Library implementation. The logic is hard-coded. 

If you want the code to work uncompiled, you should either include Closure Library's [base.js](http://code.google.com/p/closure-library/source/browse/closure/goog/base.js) with your code, or you should copy and paste the reference implementation.

### Using --only_closure_dependencies to drop files that you don't need

Suppose you don't need the ice cream shop. You just want the ice cream. If you have a lot of files, it would be a pain to manually figure out if your ice cream had any dependencies. Fortunately, `--only_closure_dependencies` helps with that. When you use `--only_closure_depenencies`, you have to tell the compiler what the entry points of your application are. It will figure out what sources you need to load that entry point, i.e., the transitive dependencies of ice.cream. It will drop everything else.

    $ java -jar compiler.jar --js shop.js --js icecream.js --js cone.js \
      --only_closure_dependencies --closure_entry_point=ice.cream
    
    var ice={};ice.cream={};

`--only_closure_dependencies` only keeps files that goog.provide a symbol. If you have files that do not goog.provide a symbol, the compiler will just skip them when this flag is on.

### Handling non-Closure dependencies

Suppose you want to compile a file that doesn't use Closure Library, but want the nice sorting and pruning of sources that Closure Compiler provides. There are two common solutions:

- Write a shell script that adds goog.provide/goog.require dependency information to the top of the file.
- Use `--manage_closure_dependencies`

`--manage_closure_dependencies` will **keep** any files that do not goog.provide symbols (whereas `--only_closure_dependencies` will drop them).

For example, you would define a new file:

----
    // File 4 - shop-app.js
    goog.require('ice.cream.Shop');
----

Because this file does not goog.provide a symbol, `--manage_closure_dependencies` will always keep it and all of its dependencies.

    $ java -jar compiler.jar --js shop-app.js --js shop.js --js icecream.js --js cone.js \
      --manage_closure_dependencies true
    
    var ice={};ice.cream={};var waffle={};waffle.cone={};ice.cream.Shop={};

### Debugging Closure Dependencies

These flag may sometimes appear to be magical. The compiler reads all your code, and then picks a subset of files to include in the compilation. How do you know which files it picked?

Fortunately, there's a flag for that.

    $ java -jar compiler.jar --js shop-app.js --js shop.js --js icecream.js --js cone.js \
      --manage_closure_dependencies true --output_manifest manifest.MF
    
    ...
    
    $ cat manifest.MF
    icecream.js
    cone.js
    shop.js
    shop-app.js

When you use `--output_manifest` on the commandline, you give the flag an output file. The compiler writes a list of all the inputs processed, in sorted order, to this output file.

### FAQ

**I made a syntax error in one of my library files, and when I used `--manage_closure_dependencies`/`--only_closure_dependencies`, Closure Compiler didn't catch it. Why not?**

When you use these flags, Closure Compiler looks at the dependencies and drops library files that are not required. It makes this determination very early in the process. It doesn't even parse the dropped files to check if they're syntactically valid.

If you're writing a library and you're primarily using Closure Compiler to do syntax checking on that library, then you should not use `--manage_closure_dependencies` or `--only_closure_dependencies`. You should create separate build rules for syntax checking and for compiling just what's needed in your app.

**I was sick of typing `goog.require`, so I aliased it to `var r = goog.require;`. Now dependency sorting doesn't work. Why?**

Closure Compiler does not even parse the code to determine its dependencies. It just does a regular expression search. If your provide and require statements are not at the top of the file in the form that the compiler expects, then it will fail to find them.

**Regular expressions? ew. Can I send you a patch to use a proper parser instead?**

No. There's a reason we did it this way. It wasn't 100% laziness....just 80-90% laziness. As it turns out, loading thousands of files from disk and building parse trees for them takes some time and memory. If you only need a few files, but you're passing in hundreds of library files that will get dropped anyway, we didn't want to slow down your compilation by an order of magnitude by trying to grok all those files.
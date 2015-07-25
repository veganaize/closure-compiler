When you need to debug code that uses Closure Compiler and Closure Library, consider **debugging uncompiled source code**. This is useful because names are not obfuscated, source code line numbers are intact, so it is easy to interpret error messages or stack traces and use the browser's debugger.
However, some people find that [debugging compiled code](https://github.com/google/closure-compiler/wiki/Debugging-Compiled-Code) works quite well.

From [*Closure: The Definitive Guide*](http://www.amazon.com/Closure-Definitive-Guide-Michael-Bolin/dp/1449381871/) by Michael Bolin, Chapter 1:

> **Code Must Work Without Compilation** Although the Compiler provides many beneficial transformations to its input, the code for the Closure Library is also expected to be able to be run without being processed by the Compiler. This not only ensures that the input language is pure JavaScript, but also makes debugging easier, as it is always possible to use the deobfuscated code.

Running from uncompiled source code requires a different way of loading the
Closure Library. The HTML file that loads and runs the application is different
in the uncompiled case (typically running directly from the source directory) 
versus the compiled case (running from the build directory).

The HTML file in the source directory should be set up for the uncompiled case. Here is an example:

```html
<script src="../../../../javascript/closure-library/closure/goog/base.js"></script>
<script src="../../../build/deps.js"></script>
<script>
  goog.define('goog.LOCALE', 'de');
  goog.define('goog.DEBUG', true);
  goog.require('myProject.myPackage.myApp');
</script>
<script>
  myApp = new myProject.myPackage.myApp();
  myApp.start();
</script>
```

The HTML file calls the Closure Library file `base.js` to define the basic functions needed by Closure Library such as `goog.provide` and `goog.require`. Next
a dependency file `deps.js` is loaded that tells for each class what it's required
classes are. Then the class of interest is loaded with a `goog.require` statement.  Once all that has happened, you can start to use the classes and functions defined in your project.

The location of `closure-library` on your system is probably different than shown in the above example, so you would need to edit accordingly.

The `deps.js` file is created using the
[depswriter.py](https://developers.google.com/closure/library/docs/depswriter) tool. The
`deps.js` file is not needed for compiled code or for the compiler. To update `deps.js` 

```bash
python ../javascript/closure-library/closure/bin/build/depswriter.py \
--root_with_prefix="src ../../../../myProject/src" > deps.js
```

Note that the `root_with_prefix` has the name of the `myProject` source directory,
and the path back to the source directory from `closure-library/closure/bin`.

Setting `goog.LOCALE` (seen in the script above) is optional, if you leave that out then
the default `LOCALE` will be used.  Similarly with `goog.DEBUG`.

In contrast, for **compiled code**, the HTML file simply runs the compiled script which has boiled
together everything needed for the application into a single script:

```html
<script src="myProject.js">
</script>
```

When running from source code, it's always a good idea to compile first, even if you are not running compiled code.  The compiler has excellent checks that catch lots of errors.
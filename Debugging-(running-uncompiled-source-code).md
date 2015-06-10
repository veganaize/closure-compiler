The best way to debug is by running directly from **uncompiled source code**, because source code line numbers are intact, so it is easier to interpret error messages or stack traces and use the browser's debugger.

Advance-compiled code is of course very difficult to interpret because of obfuscation (shortening) of names. Even simple-compiled code is concatenated into very long lines that don't correspond to the source code, and local variables are obfuscated.

Running from uncompiled source code requires a different way of loading the Google
Closure Library. The HTML file that loads and runs a the application is different
in the uncompiled case (running from the source directory) versus the compiled case (running
from the build directory).

The HTML file in the source directory is set up for the uncompiled case. The HTML file calls the Google Closure Library directly, and
a dependency file `deps.js` is loaded that tells for each class what it's required
classes are. Then the class of interest is loaded with a `goog.require` statement:

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

The location of `closure-library` on your system is probably different, so you would need to edit accordingly.

The `deps.js` file is created using the
[depswriter.py](https://developers.google.com/closure/library/docs/depswriter) tool. The
`deps.js` file is not needed for compiled code. To update `deps.js` 

```bash
python ../javascript/closure-library/closure/bin/build/depswriter.py \
--root_with_prefix="src ../../../../myProject/src" > deps.js
```

Note that the `root_with_prefix` has the name of the `myProject` source directory,
and the path back to the source directory from `closure-library/closure/bin`.

Setting `goog.LOCALE` (seen in the script above) is optional, if you leave that out then
the default `LOCALE` will be used.  Similarly with `goog.DEBUG`.

In the **compiled case**, the HTML file simply runs the compiled script which has boiled
together everything needed for the application into a single script:

```html
<script src="myProject.js">
</script>
```

Here are some tips about debugging compiled code that uses Closure Compiler and Closure Library.

The book [*Closure: The Definitive Guide*](http://www.amazon.com/Closure-Definitive-Guide-Michael-Bolin/dp/1449381871/) by Michael Bolin has an entire chapter on this topic: Chapter 16 *Debugging Compiled JavaScript*.  That chapter is well worth reading, what follows here is mostly a summary of that information.

### Check Whether the Error Occurs in Uncompiled Mode
From *Closure: The Definitive Guide* Chapter 16:
> As explained in Chapter 1, one of the design principles of Closure Library code is that it should work in both compiled and uncompiled modes. This means that if you en- counter a bug in your compiled code, your first step should be to test whether you can reproduce the bug in the uncompiled version. If such a bug is reproducible, then it will likely be easier to debug using the uncompiled code, as all of your traditional techniques for JavaScript debugging will still work. Also, double-check to make sure that your code compiles without any warnings, and use the Verbose warning level to maximize the number of checks performed by the Compiler. You may be lucky enough to discover that the Compiler has already found your bug for you!

>When the bug occurs only in compiled mode and there are no warnings from the Compiler, then debugging may be more challenging.

See [Debugging Uncompiled Source Code](https://github.com/google/closure-compiler/wiki/Debugging-Uncompiled-Source-Code)

### Use `--debug` option
Passing the `--debug` option to the compiler results in the compiler doing all the same things it otherwise does, but the names are not obfuscated. (Names will look strange with lots of $ signs, but they
are readable).

### Use `--formatting=PRETTY_PRINT` option
The PRETTY_PRINT compiler option maintains line breaks similar to the original code, making it easier to read.

### Use `--formatting=PRINT_INPUT_DELIMITER`
The PRINT_INPUT_DELIMITER compiler option adds a comment with the name of every new input file as it is processed,
so you can tell what file the compiled code is from.

### Use source maps
The compiler option `--create_source_map` creates a file that maps locations in compiled code to the original unobfuscated names.  Browser developer tools can use a source map while debugging.  See
[How can I debug the JavaScript that the Closure Compiler produces?](https://developers.google.com/closure/compiler/faq#sourcemaps)





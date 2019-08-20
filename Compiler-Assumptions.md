# Introduction

To allow for better optimizations in the general case, the compiler makes various assumptions. This is an attempt to document these assumptions more fully.

## Kinds of assumptions

There are two main kinds of assumptions the compiler makes.

 * Assumptions about how the code you pass to Closure Compiler interacts with code the compiler does not see. Examples include browser APIs, JSON data, and libraries you include but do not compile. See https://developers.google.com/closure/compiler/docs/api-tutorial3#mixed.
 * Assumptions about patterns that are valid JavaScript, but do not work in the compiler. See examples below.

## Details

The most important assumptions are described in 
[Understanding the Restrictions Imposed by the Closure Compiler](https://developers.google.com/closure/compiler/docs/limitations), however there are some additional assumptions.

- Name references are assumed to never throw, as described [here](https://blickly.github.io/closure-compiler-issues/#64)
- Property reads are assumed to be side effect free, as described [here](https://blickly.github.io/closure-compiler-issues/#398). We support getters only if they are side-effect free
- In ADVANCED mode, property sets do not have arbitrary side-effects (it is ok to remove property writes if the property is never read), as described [here](https://blickly.github.io/closure-compiler-issues/#705)
- The original value of Function.length may not be preserved, as described [here](https://blickly.github.io/closure-compiler-issues/#253).
- [[toString and valueOf]] are assumed to be side-effect free.
- instanceof is assumed to never throw

General:
- "undefined", "NaN" and "Infinity" have not been externally redefined.
- Object, Array, String, Number and Boolean have not been redefined. Standard methods on Object, Array, String, Number and Boolean have not been redefined in a way that breaks the original contract.

## Debugging compiled code

Debugging compiled code is difficult. Here are some basic tips:

* The [Closure Compiler debugger](https://closure-compiler-debugger.appspot.com) is useful for testing the compiler on small code samples. It's also helpful for creating smaller repros.
* The Closure Compiler supports source maps
* The `--debug` flag makes name mangling less confusing. The compiler gives variables and properties more meaningful names.
 * The `--formatting=PRETTY_PRINT` or `formatting=PRETTY_PRINT_INPUT_DELIMITER` flags make the output code more readable.
 * The `--print_source_after_each_pass` flag print all the source code to stdout after each compiler pass. It's helpful when determining what compiler pass is responsible for breaking your code.

Ask the [closure-compiler-users mailing list](https://groups.google.com/a/google.com/group/closure-compiler-users) if you're stuck, or if you don't know whether a compiler behavior is a bug or caused by violating a compiler assumption.
# Introduction

To allow for better optimizations in the general case, the compiler makes various assumptions. This is an attempt to document these assumptions more fully.


# Details

The most important assumptions are described in 
[Understanding the Restrictions Imposed by the Closure Compiler](http://code.google.com/closure/compiler/docs/limitations.html), however there are some additional assumptions.

- Name references are assumed to never throw, as described [here](http://code.google.com/p/closure-compiler/issues/detail?id=64)
- Property reads are assumed to be side effect free, as described [here](http://code.google.com/p/closure-compiler/issues/detail?id=398). We support getters only if they are side-effect free
- In ADVANCED mode, property sets do not have arbitrary side-effects (it is ok to remove property writes if the property is never read), as described [here](http://code.google.com/p/closure-compiler/issues/detail?id=705)
- The original value of Function.length may not be preserved, as described [here](http://code.google.com/p/closure-compiler/issues/detail?id=253).
- [[toString And valueOf]] are assumed to be side-effect free.
- ~~Math object methods are side-effect free.~~
- instanceof is assumed to never throw

General:
- "undefined", "NaN" and "Infinity" have not been externally redefined.
- Object, Array, String, Number and Boolean have not been redefined. Standard methods on Object, Array, String, Number and Boolean have not been redefined in a way that breaks the original contract.
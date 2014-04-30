# Links to Closure Compiler design documents and proposals

### Architecture

Closure Compiler consists of 3 essential mechanisms:
- A "!ParserRunner" that uses [Rhino](http://www.mozilla.org/rhino/) to parse JavaScript, and converts it into an abstract syntax tree.
- A "!PassConfig" that chooses compiler passes to transform this syntax tree.
- A "!CommandLineRunner" that translates user-specified options into a "!PassConfig", runs the passes, and then serializes the syntax tree back to JavaScript.

Everything else that Closure Compiler does is implemented as a "!CompilerPass". Compiler passes do many things. Some do type-checking. Some do optimizations. Some check the output of other compiler passes. Some create more complicated symbol tables, which are then fed into other compiler passes.

In all cases, the syntax tree is the source of truth. Every compiler pass should leave the syntax tree in a consistent state (so that if we generated JavaScript from that syntax tree, the code would still run correctly). 

If you are a developer on Closure Compiler, you can install "sanity check" passes to ensure that the syntax tree is always valid between each pass. Some of these sanity checks are run all the time, even for non-developers. For example, the !VarCheck pass is run at the end of every compiler job to ensure that all variables are declared in the syntax tree.

### Designs for specific passes

[The Type System](https://docs.google.com/document/edit?id=1L6g92j9Z-w3uvDP0H-0whb2fOcwnjiXRY8pbG9LkOvQ)

[The Internal Syntax Tree (PDF)](http://code.google.com/p/closure-compiler/downloads/detail?name=closure-compiler-ast.pdf&can=2&q=)

[Closure Compiler Side-effect annotations for Extern definitions](https://docs.google.com/document/pub?id=1oaJGeYqhat13e_gpE47ToQc1jPCN6k59438OoUGeXFc)

Property Renaming [Part 1](http://closuretools.blogspot.com/2011/01/property-by-any-other-name-part-1.html), [Part 2](http://closuretools.blogspot.com/2011/01/property-by-any-other-name-part-2.html), and [Part 3](http://closuretools.blogspot.com/2011/01/property-by-any-other-name-part-3.html)

[[Experimental Type Based Property Renaming|Type-based Renaming]]

[EcmaScript 4 Draft Spec](https://docs.google.com/viewer?a=v&pid=explorer&chrome=true&srcid=0B4ovp2kaZSwWM2Q4ZmI3MjgtMGZhZi00NDEwLTk0N2MtZTEzYjE3YTI4ZjY3) - Our type system semantics are based on this spec.

[EcmaScript 4 Draft Grammar](https://docs.google.com/viewer?a=v&pid=explorer&chrome=true&srcid=0B4ovp2kaZSwWZjFiOGU1MjItZTBhMy00NGIxLWFlMzItY2VhNTMyMmIxYmZj) - Our type expression grammar is based on this spec.

[The Warnings API](https://docs.google.com/document/d/1xPIM-GjEjhW6Y1bzGGdfHYgWM6dBGC56_XcPu1hSqbU/edit)
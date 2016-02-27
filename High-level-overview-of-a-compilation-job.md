This document gives a short overview of Closure Compiler's code, and the main classes that are involved in a compilation. It is intended for people who are getting started with the compiler, and want to make changes to it and understand how it works.

For each compilation, an instance of [Compiler](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/Compiler.java) is created. The compiler class contains several `compile` methods, and they all end up calling [`compileInternal`](https://github.com/google/closure-compiler/search?utf8=%E2%9C%93&q=%22private+void+compileInternal%28%29%22).

The [CommandLineRunner](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/CommandLineRunner.java) takes the command-line arguments and creates an instance of [CompilerOptions](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/CompilerOptions.java). The compiler uses the options object to determine which passes will be run (some checks and some optimizations). This happens in [DefaultPassConfig](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/DefaultPassConfig.java). The two important methods in this class are `getChecks` and `getOptimizations`.

Before running the checks, the compiler parses the code and creates an abstract-syntax tree (AST). The structure of the AST is described in [Node.java](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/rhino/Node.java) and [Token.java](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/rhino/Token.java).

[PhaseOptimizer](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/PhaseOptimizer.java) takes the list of passes created in the pass config and runs them. Running the checks is simple, we just go through the list of checks and run each check once. Some optimization passes run once, and others run in a loop until they can no longer make changes. During an optimization loop, the compiler tries to avoid running passes that are no longer making changes. If you are experimenting with the compiler and want to see code before each pass, add
```java
System.out.println("Before pass: " + this.name + "\n" + compiler.toSource(root));
```
at the beginning of the `process` method of [`NamedPass`](https://github.com/google/closure-compiler/search?utf8=%E2%9C%93&q=%22class+NamedPass+implements+CompilerPass%22). If you want to see how long each pass takes, and how each pass changes code size, pass the flag `--tracer_mode=ALL` to the compiler.

After all checks and optimizations are finished, the AST is converted back to JavaScript source. See [CodeGenerator](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/CodeGenerator.java) and [CodePrinter](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/CodePrinter.java).

This is basically it. Below, we briefly describe some of the common compiler passes.

[NodeTraversal](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/NodeTraversal.java) has utility methods to traverse the AST. All compiler passes use a traversal to go through the code, rather than hand-written recursion.

The type-checking code lives in [TypedScopeCreator](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/TypedScopeCreator.java), [TypeInference](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/TypeInference.java) and [TypeCheck](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/TypeCheck.java). The code for the new type checker (still under development) lives in [GlobalTypeInfo](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/GlobalTypeInfo.java) and [NewTypeInference](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/NewTypeInference.java).

To see some of the optimizations, start at [getMainOptimizationLoop](https://github.com/google/closure-compiler/search?utf8=%E2%9C%93&q=%22PassFactory%5C%3E+getMainOptimizationLoop%22&type=Code) and look at the passes used there.

Also, the [debugger](http://closure-compiler-debugger.appspot.com/) is really useful. You can paste some JS, turn individual passes on and off, and see the AST, the generated code, and the compiler warnings.
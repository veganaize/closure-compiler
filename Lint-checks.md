With the deprecation of [Closure Linter](https://developers.google.com/closure/utilities/), the Closure Compiler team has added several "lint" style checks to the compiler. You can enable these with `--jscomp_warning=lintChecks` but this may produce a lot of noise if your project includes generated code. For example, you probably don't care much if goog.require statements are sorted correctly in a generated file. So you can also build the standalone linter binary:

    mvn install -DskipTests -pl com.google.javascript:closure-compiler-linter

Note that because the lint checks operate on the abstract syntax tree of your program, there are no checks for formatting issues such as indentation. For formatting, we recommend [`clang-format`](https://clang.llvm.org/docs/ClangFormat.html)
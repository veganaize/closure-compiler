# Calling the compiler from Java to have more control

# Introduction

placeholder page


# Details

Topics:

- Setting default options
     `CompilationLevel.SIMPLE_OPTIMIZATIONS.setOptionsForCompilationLevel(options);`
     `WarningLevel.VERBOSE.setOptionsForWarningLevel(options);`
- Pulling in the default externs
     See [CommandLineRunner.java](http://code.google.com/p/closure-compiler/source/browse/src/com/google/javascript/jscomp/CommandLineRunner.java) getDefaultExterns


References:

http://blog.bolinfest.com/2009/11/calling-closure-compiler-from-java.html

http://closure-compiler.googlecode.com/git/javadoc/com/google/javascript/jscomp/CompilerOptions.html
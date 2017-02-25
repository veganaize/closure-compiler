# Calling the compiler from Java to have more control

### Details

- Setting default options
     `CompilationLevel.SIMPLE_OPTIMIZATIONS.setOptionsForCompilationLevel(options);`
     `WarningLevel.VERBOSE.setOptionsForWarningLevel(options);`
- Pulling in the default externs
     See [CommandLineRunner.java](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/CommandLineRunner.java) getDefaultExterns


### References

http://hyegar.com/2017/01/09/calling-google-closure/

http://blog.bolinfest.com/2009/11/calling-closure-compiler-from-java.html
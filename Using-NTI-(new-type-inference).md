The compiler option `--new_type_inf` activates the "new type inference" version of the compiler (NTI).  This is still under development (as of May 2015), but is already useful.  It catches more error conditions, which can help find bugs in your code.  But you will probably need to make a fair number of changes to eliminate the warnings that it generates.

Note that the closure library has not yet been tuned to work with NTI, so you will get a large number (around 1000) of warnings from closure library code that you use.

The majority of warnings in your code will probably be `attempt to use nullable type`.  The compiler is warning that you are using a variable that can be `null` in a situation where it must be non-null. These warnings are fairly easy to fix, but it does take a bit of editing of your code.  The best solution is to prefer a non-nullable type whenever possible, so in your code make most properties and parameters and return types be defined as non-nullable.  When you *must* use a nullable type you simply test whether the object is `null` before using it.

You will see many other warnings besides `attempt to use nullable type`.  It may be puzzling what to do to get rid of the warning.  You can ask on the [closure-compiler email list](https://groups.google.com/forum/#!forum/closure-compiler-discuss) if you are stumped.

Some NTI warnings are just plain wrong -- these are bugs in the compiler that have not yet been fixed.  You can usually tell because the warning just doesn't make sense.   But not always!  Sometimes the warning is rather cryptic.  To help you distinguish good warnings from bad warnings look in the [list of issues and select the NTI label](https://github.com/google/closure-compiler/issues?q=is%3Aopen+is%3Aissue+label%3ANTI).  

You can help move the NTI compiler project along by reporting bugs you find in the compiler (as long as the bug has not yet been reported).
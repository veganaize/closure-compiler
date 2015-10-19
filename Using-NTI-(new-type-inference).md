The compiler option `--new_type_inf` activates the "new type inference" version of the compiler (NTI).  This is still under development (as of May 2015), but is already useful.  It catches more error conditions, which can help find bugs in your code.  See [issue #826](https://github.com/google/closure-compiler/issues/826) for an example of a situation that NTI compiles correctly where the old compiler did not. But you will probably need to make a fair number of changes to eliminate the warnings that NTI generates.

Note that the closure library has not yet been tuned to work with NTI, so you will get a large number of warnings (around 1000) from closure library code that you use.  

It may be puzzling what to do to get rid of a warning.  You can ask on the [closure-compiler email list](https://groups.google.com/forum/#!forum/closure-compiler-discuss) if you are stumped.

Because NTI is still under development you will only want to use it occasionally, to work thru bugs in your own code.  Then switch NTI off for your regular work to avoid seeing warnings that cannot yet be fixed.

### compiler bugs

NTI is still under development, so there are compiler bugs.  You may encounter NTI warnings are just plain wrong -- these are bugs in the compiler that have not yet been fixed.

You can usually spot a compiler bug because the warning just doesn't make sense.   But not always!  Sometimes the warning is rather cryptic.  To help you distinguish good warnings from bad warnings look in the [list of issues and select the NTI label](https://github.com/google/closure-compiler/issues?q=is%3Aopen+is%3Aissue+label%3ANTI).  

You can help move the NTI compiler project along by reporting bugs you find in the compiler (as long as the bug is not a duplicate of an already reported issue).


### warning "attempt to use nullable type"

The majority of warnings in your code will probably be "attempt to use nullable type".  The compiler is warning that you are using a variable that can be `null` in a situation where it must be non-null. These warnings are fairly easy to fix, but it does take a bit of editing of your code.  The best solution is to prefer a non-nullable type whenever possible: make properties and parameters and return types be defined as non-nullable. Then you can just use that object without testing whether it is null.  A non-nullable type begins with an exclamation mark.  Here is an example:

```javascript
/**
* @param {!HTMLInputElement} inputElem
* @constructor
*/
myNamespace.myClass = function(inputElem) {
  /**
  * @type {!HTMLInputElement}
  * @private
  */
  this.inputElem_ = inputElem;
};
```

When you *must* use a nullable type you simply test whether the object is `null` before using it. Here is an example.

```javascript
/**
* @param {HTMLInputElement} inputElem
* @constructor
*/
myNamespace.myClass = function(inputElem) {
  /**
  * @type {!HTMLInputElement}
  * @private
  */
  this.inputElem_ = inputElem != null ? inputElem : 
       /** @type {!HTMLInputElement} */(document.createElement('input'));
};
```

You can also use `goog.isDefAndNotNull()` to test if the variable is `null`.

### warning "dangerous use of the global `this` object"

You might see this warning if you use functions like `goog.array.forEach` which have an argument for `this` to apply to a passed in function.  There are a couple of issues about this:

1. <https://github.com/google/closure-compiler/issues/834>
2. <https://github.com/google/closure-compiler/issues/994>

It is possible to avoid this warning by not passing the `this` argument to `goog.array.forEach`, see issue #994 for an example.

In some cases adding the annotation `@this {myType.Foo}` can solve this warning. See [an email on the closure compiler list of Oct 19, 2015](https://groups.google.com/d/msg/closure-compiler-discuss/22FsLdUCWbs/t1dq0-nWAgAJ) for an example of this.  That example involves defining a function operating on `this` inside a constructor.

### mis-spelled property names and `@struct`

*This is not related to NTI, but falls in category of "wanting the compiler to find bugs in your code".*

Suppose an object has a property named `address`.  If you misspell that property by doing:
```javascript
this.adress = "10 Main St.";
```
the compiler will NOT complain.  It assumes you are adding a new property to `this`.
The fix is to annotate the constructor with `@struct`.

>Assigning has different behavior than property access, because it creates a property. If you want to 
prevent accidental property creation by assignment, you should annotate the constructor with @struct

See <https://github.com/google/closure-compiler/issues/861>

Note that `@constructor` implies `@struct` in recent versions of the compiler. See the following [note from compiler release v20151015](https://groups.google.com/d/msg/closure-compiler-discuss/IcQfig9Bd1M/lwERBi4DAgAJ) about using `@struct`

> ` @struct` is now supported for `@interface` and `@record` declarations in
the "old" type inference (The NTI has this behavior by default).
`goog.defineClass` `@interface`s are `@struct` by default (like
`@constructor`)
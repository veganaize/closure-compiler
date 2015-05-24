The compiler option `--new_type_inf` activates the "new type inference" version of the compiler (NTI).  This is still under development (as of May 2015), but is already useful.  It catches more error conditions, which can help find bugs in your code.  But you will probably need to make a fair number of changes to eliminate the warnings that it generates.

Note that the closure library has not yet been tuned to work with NTI, so you will get a large number of warnings (around 1000) from closure library code that you use.  

It may be puzzling what to do to get rid of a warning.  You can ask on the [closure-compiler email list](https://groups.google.com/forum/#!forum/closure-compiler-discuss) if you are stumped.

Because NTI is still under development you will only want to use it occasionally, to work thru bugs in your own code.  Then switch NTI off for your regular work to avoid seeing warnings that cannot yet be fixed.

## compiler bugs

NTI is still under development, so there are compiler bugs.  You may encounter NTI warnings are just plain wrong -- these are bugs in the compiler that have not yet been fixed.  You can usually tell because the warning just doesn't make sense.   But not always!  Sometimes the warning is rather cryptic.  To help you distinguish good warnings from bad warnings look in the [list of issues and select the NTI label](https://github.com/google/closure-compiler/issues?q=is%3Aopen+is%3Aissue+label%3ANTI).  

You can help move the NTI compiler project along by reporting bugs you find in the compiler (as long as the bug is not a duplicate of an already reported issue).


## warning "attempt to use nullable type"

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
       /** @type {!HTMLInputElement} */(document.createElement('input')); ;
};
```

You can also use `goog.isDefAndNotNull()` to test if the variable is `null`.

## mis-spelled property names and `@struct`

Suppose an object has a property named `address`.  If you misspell that property by doing:
```javascript
this.adress = "10 Main St.";
```
the compiler will NOT complain.  It assumes you are adding a new property to `this`.
The fix is to annotate the constructor with `@struct`.

>Assigning has different behavior than property access, because it creates a property. If you want to 
prevent accidental property creation by assignment, you should annotate the constructor with @struct

See <https://github.com/google/closure-compiler/issues/861>

## `goog.forwardDeclare()` helps with circular references

NTI changes when `goog.require()` is needed. The old way is described in the book *Closure, The Definitive Guide* by Michael Bolin, p. 50:

>Note that `goog.require()` is not exactly the same as `import` in Java. In Closure, defining a function that takes a parameter of a particular type does not necessitate a `goog.require()` call to ensure the type has been loaded. For example, the following code does not require `goog.math.Coordinate`:

```javascript
/**
* @param {number} radius
* @param {goog.math.Coordinate} point
* @return {boolean} whether point is within the specified radius
*/
example.radius.isWithinRadius = function(radius, point) {
  return Math.sqrt(point.x * point.x + point.y * point.y) <= radius;
};
```
With NTI this no longer works, you will get warnings like "Type annotation references non-existent type".  Using `goog.forwardDeclare()` solves the problem.  In the above example you would add

```javascript
goog.forwardDeclare(goog.math.Coordinate);
```

This will compile if somewhere else in the code you have `goog.require(goog.math.Coordinate)`.

See <https://github.com/google/closure-compiler/issues/838>






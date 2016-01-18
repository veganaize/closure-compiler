The compiler option `--new_type_inf` activates the "new type inference" version of the compiler (NTI).  This is still under development (as of December 2015), but is already useful.  It catches more error conditions, which can help find bugs in your code.  See [issue #826](https://github.com/google/closure-compiler/issues/826) for an example of a situation that NTI compiles correctly where the old compiler did not. But you will probably need to make a fair number of changes to eliminate the warnings that NTI generates.

Note that the closure library has not yet been tuned to work with NTI, so you will get a large number of warnings (around 1000) from closure library code that you use.  

It may be puzzling what to do to get rid of a warning.  You can ask on the [closure-compiler email list](https://groups.google.com/forum/#!forum/closure-compiler-discuss) if you are stumped.

### Common warnings when you first start using NTI.

1. Stricter property checks. The current type checker is very loose about missing properties; with NTI you will see many more missing-property warnings.
2. NTI warns when the operands of primitive operators (such as -, <, etc) have the wrong type.
3. NTI is stricter about typing generics.

### Compiler bugs

NTI is still under development, and it may produce some warnings that are wrong -- these are bugs in the compiler that have not yet been fixed. To help you distinguish good warnings from bad warnings look in the [list of issues and select the NTI label](https://github.com/google/closure-compiler/issues?q=is%3Aopen+is%3Aissue+label%3ANTI).  

You can help move the NTI compiler project along by reporting bugs you find in the compiler (as long as the bug is not a duplicate of an already reported issue).

### Stricter property checks

Suppose you see a property access `x.prop1`, and the type of `x` is *A*. In the old type checker, *A* may not have `prop1`, but if the property exists on any subtype of *A* you will not get a warning. As a consequence, you will never get a missing-property warning on *Object* (unless the property is not defined anywhere) because it is a supertype of all object types. This is too loose, and many people have asked us for stricter checks. In NTI, if `prop1` is not defined on *A*, you get a warning, even if it is defined on a subtype.

Also, the current type checker does not warn about property accesses on _*_, because it is a supertype of all types, including *Object*. But NTI warns on all property accesses when the receiver is not an object, so it warns here.

### Function vs method types

A function does not specify a receiver type, a method does. In NTI, you cannot pass a method to a context that expects a function, because it can then be called without a receiver type. For example, the current type checker doesn't warn about the following program, but NTI (correctly) does.

```
class Foo {
  constructor() {
    /** @type {number} */
    this.prop = 123;
  }
  setX(/** number */ x) {
    this.prop = x;
  }
}

function f(/** function(number) */ fun) {
  fun(123);
}

f((new Foo).setX); // Warning here
```

### Typedefs are not namespaces

A typedef is supposed to create a new type name for an existing type. The main use is to avoid retyping long names over and over. However, the old type checker was loose about typedefs and did not warn about cases that were not meant to be supported. Specifically, people would use the typedef name as a value, and assign properties to it. This is not supported; a typedef name is meant to be used in type annotations only.

```
/** @typedef {number} */
var N;

/** @type {string} */
N.prop = 'asdf'; // warning, adding properties to typedefs is not allowed.
```

### Warning "dangerous use of the global `this` object"

You might see this warning if you use functions like `goog.array.forEach` which have an argument for `this` to apply to a passed in function.  Relevant issue: <https://github.com/google/closure-compiler/issues/994>. It is possible to avoid this warning by not passing the `this` argument to `goog.array.forEach`.

In some cases adding the annotation `@this {myType.Foo}` can solve this warning. See [an email on the closure compiler list of Oct 19, 2015](https://groups.google.com/d/msg/closure-compiler-discuss/22FsLdUCWbs/t1dq0-nWAgAJ) for an example of this.  That example involves defining a function operating on `this` inside a constructor.

### Misspelled property names and `@struct`

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

Note that classes defined with `goog.defineClass` or the `class` keyword should automatically imply `@struct`.

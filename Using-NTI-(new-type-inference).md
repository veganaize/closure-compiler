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

Also, the current type checker does not warn about property accesses on _*_, because it is a supertype of all types, including *Object*. But NTI warns on a property access when the receiver is not an object, so it warns here.

### Warnings about uninferred constants

If you define a constant variable or property, NTI will attempt to infer the constant's type and use that type in any scope that references the constant. If NTI cannot infer the type, it will warn. Since each function scope is typechecked in isolation, const inference must happen early, with limited type information. So, if you think a const should have been inferred but you are getting a warning, keep in mind that a limited inference is used. For example, take a look at this [snippet](https://closure-compiler-debugger.appspot.com/#input0%3Dfunction%2520f()%2520%257B%250A%2520%2520var%2520x%2520%253D%2520123%253B%250A%2520%2520%252F**%2520%2540const%2520*%252F%250A%2520%2520var%2520c%2520%253D%2520x%253B%250A%2520%2520var%2520%252F**%2520string%2520*%252F%2520s%2520%253D%2520c%253B%250A%2520%2520return%2520function%2520()%2520%257B%2520return%2520c%253B%2520%257D%250A%257D%26input1%26conformanceConfig%26externs%26refasterjs-template%26includeDefaultExterns%3D1%26CHECK_SYMBOLS%3D1%26CHECK_TYPES%3D1%26CHECK_TYPES_NEW_INFERENCE%3D1%26CLOSURE_PASS%3D1%26LANG_IN_IS_ES6%3D1%26MISSING_PROPERTIES%3D1%26PRESERVE_TYPE_ANNOTATIONS%3D1%26PRETTY_PRINT%3D1%26TRANSPILE%3D1):
we don't know the type of `c` during const inference, but we do know it when typechecking `f`.

The old type checker would not warn about this, but it would silently type the constant as unknown when it cannot infer the type. Moreover, the const inference in the old type checker is dependent on the order of type checking the function scopes, which is error prone. In this [example](https://closure-compiler-debugger.appspot.com/#input0%3D%252F**%2520%2540const%2520*%252F%250Avar%2520ns%2520%253D%2520%257B%257D%253B%250A%250Afunction%2520g()%2520%257B%250A%2520%2520var%2520%252F**%2520null%2520*%252F%2520n1%2520%253D%2520ns.myconst%253B%250A%257D%250A%250Afunction%2520f()%2520%257B%250A%2520%2520%252F**%2520%2540constructor%2520*%252F%250A%2520%2520ns.Foo%2520%253D%2520function()%2520%257B%257D%253B%250A%250A%2520%2520%252F**%2520%2540const%2520*%252F%250A%2520%2520ns.myconst%2520%253D%2520goog.asserts.assert(new%2520ns.Foo)%253B%250A%257D%250A%250Afunction%2520h()%2520%257B%250A%2520%2520var%2520%252F**%2520null%2520*%252F%2520n2%2520%253D%2520ns.myconst%253B%250A%257D%26input1%26conformanceConfig%26externs%3Dvar%2520goog%253B%250Agoog.asserts%253B%250Agoog.asserts.assert%253B%26refasterjs-template%26includeDefaultExterns%3D1%26CHECK_SYMBOLS%3D1%26CHECK_TYPES%3D1%26CLOSURE_PASS%3D1%26LANG_IN_IS_ES6%3D1%26MISSING_PROPERTIES%3D1%26PRESERVE_TYPE_ANNOTATIONS%3D1%26PRETTY_PRINT%3D1%26TRANSPILE%3D1),
`ns.myconst` is inferred in scope `h` but not in scope `g`.

### Function vs method types

A function does not specify a receiver type, a method does. In NTI, you cannot pass a method to a context that expects a function, because it can then be called without a receiver type. For example, the current type checker doesn't warn about the following program, but NTI (correctly) does.

```js
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

```js
/** @typedef {number} */
var N;

/** @type {string} */
N.prop = 'asdf'; // warning, adding properties to typedefs is not allowed.
```

Also, NTI warns about recursive typedefs. (Again, because a typedef cannot create a new type, just an alias for an already existing type.) Recursive typedefs were never supported, but were silently unchecked by the old type checker.

### No support for parameterized `Object<K,V>`

The old type checker supports type parameters for the *Object* type, even though its definition is not generic. This is a hack to support map-like objects. Now, with *IObject* and ES6 maps, special-casing *Object* to be generic is not necessary, and NTI does not support it.

### Warning "dangerous use of the global `this` object"

You might see this warning if you use functions like `goog.array.forEach` which have an argument for `this` to apply to a passed in function.  Relevant issue: <https://github.com/google/closure-compiler/issues/994>. It is possible to avoid this warning by not passing the `this` argument to `goog.array.forEach`.

In some cases adding the annotation `@this {myType.Foo}` can solve this warning. See [an email on the closure compiler list of Oct 19, 2015](https://groups.google.com/d/msg/closure-compiler-discuss/22FsLdUCWbs/t1dq0-nWAgAJ) for an example of this.  That example involves defining a function operating on `this` inside a constructor.

### Misspelled property names and `@struct`

*This is not related to NTI, but falls in category of "wanting the compiler to find bugs in your code".*

Suppose an object has a property named `address`.  If you misspell that property by doing:
```js
this.adress = "10 Main St.";
```
the compiler will NOT complain.  It assumes you are adding a new property to `this`.
The fix is to annotate the constructor with `@struct`.

>Assigning has different behavior than property access, because it creates a property. If you want to 
prevent accidental property creation by assignment, you should annotate the constructor with @struct

See <https://github.com/google/closure-compiler/issues/861>

Note that classes defined with `goog.defineClass` or the `class` keyword should automatically imply `@struct`.

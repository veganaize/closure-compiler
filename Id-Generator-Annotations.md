# @idGenerator {unique|consistent|stable|mapped|xid}

## Introduction

ID generator annotations indicate that a function of type `{function(string):string}` or `{function(!Object<string,?>):!Object<string,?>}` will be used to produce short, unique string IDs. When compiled, string literal arguments in invocations of such functions are directly replaced with the calculated IDs.  A common use case for this feature is creating obfuscated HTML IDs.

## String id generators

```js
/**
 * Gets an ID for the passed name.
 * @idGenerator {consistent}
 * @param {string} myLongName The name to get an ID for.
 * @return {string} A short, unique ID that is consistent per input name.
 */
foo.getId = function(myLongName) {
  return myLongName;
};

foo.myElement.id = foo.getId('my-super-duper-element');
foo.myOtherElement.id = foo.getId('my-other-element');
foo.mainId = foo.getId('my-super-duper-element'); 
```

When uncompiled, the bare long name is used as the ID.  When compiled though, the compiler replaces `getId` invocations with a string literal.  For example:

```js
foo.myElement.id = 'x0';
foo.myOtherElement.id = 'x1';
foo.mainId = 'x0';
```

## Object literal id generators

For convenience, the compiler also supports generating unique ids for keys in an entire object literal. For example:

```js
/**
 * Returns an Object literal that has every key transformed to be a unique id.
 * @idGenerator {consistent}
 * @param {!Object<?>} id Must be an object literal
 * @return {!Object<string, ?>}
 */
foo.getId = function(id) {
  return id;
};

foo.ids = foo.getId({myElement: myElementInstance, myOtherElement, mainId: 'This is the main id.'});
```

When compiled, the compiler replaces the string keys in `getId` invocations with string literals.  For example:

```js
foo.ids = foo.getId({'x0': myElementInstance, 'x1': myOtherElement, 'x2': 'This is the main id.'});
```

The compiler will report an error if the object literal contains computed properties, member functions, getters, or setters.

A single `@idGenerator` function may generate unique ids for both string literal and object literal arguments.

## ID Generator Strategies

Five ID generation strategies are supported.

`@idGenerator {unique}` creates short names using a simple counter.  Since compiled IDs are dependent on invocation order, all `@idGenerator {unique}` invocations must be unconditional and either global or module scope.  The generated ID is unique within the compilation unit for the named function but may be repeated by other ID generators (and for that reason should not be treated as a GUID).  While `@idGenerator {unique}` is guaranteed to be an injective function, the exact IDs produced should be considered opaque and are not guaranteed to be stable across separate compilations.

`@idGenerator {consistent}` produce the same short name for the same input string, as shown in the example above. For a consistent id generator `id`, `id('foo')` will produce the same result (unlike `@idGenerator {unique}` which produces a unique result each time).  Invocations of the generator method may also be in any context. As with `@idGenerator {unique}`, the ids may be repeated with those from other @idGenerator functions. The exact IDs produced should be considered opaque and are not guaranteed to be stable across separate compilations.

`@idGenerator {stable}` constructs IDs using a deterministic hashing strategy, at the cost of slightly longer IDs than `@idGenerator {consistent}` and `@idGenerator {unique}`.  Unlike `@idGenerator {consistent}` and `@idGenerator {unique}`, its IDs are guaranteed to be stable across separate compilations.  Invocations may also be made in any context.

`@idGenerator {mapped}` constructs IDs using the name map provided to the compiler. The map is provided via the CompilerOptions#setIdGenerators method. Stability and uniqueness of the naming depends on the map provided.  NOTE: Unlike renaming maps that might be provided for `unique` and `consistent` which taken as a "hint" which might be ignored or where missing names will be provided, it is an error for a requested name to be missing from the map.

`@idGenerator {xid}` constructs IDs using a customizable deterministic hashing strategy. The results are very similar to `@idGenerator {stable}`, in that IDs are stable across separate compilations. By default the compiler uses `Object#hashCode()`. However, it is possible to provide a custom hashing strategy through [`Xid.HashFunction`](src/com/google/javascript/jscomp/Xid.java). Call `CompilerOptions.setXidHashFunction()` to configure the hashing strategy for all `@idGenerator {xid}` functions. This option is only available through the Java API.

Only one of the ID generator annotations may be added to a particular function.

## Compiler Options

ID generator replacement is on by default, but may be toggled using the Java API with the `CompilerOptions#setReplaceIdGenerators` method.

For debugging support, `CompilerOptions#setGeneratePseudoNames` may be set to make the transformed IDs human readable.

To improve stability across builds it is possible to supply name maps for use by the `ReplaceIdGenerators` compiler pass, the map is used as a hint for the renaming and the compiler is free to pick a different name.  If cross build stability is required `@idGenerator {stable}` should be used. Setting the renaming map is done via `CompilerOptions#setIdGeneratorsMap`.

**NOTE:** There are three deprecated variants: `@idGenerator`, `@consistentIdGenerator`, and `@stableIdGenerator`.  These map to `@idGenerator {unique}`, `@idGenerator {consistent}`, and `@idGenerator {stable}`.  Support for these will be removed in a future compiler release.
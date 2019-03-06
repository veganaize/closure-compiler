# Overview of Closure's Type System for JavaScript

# Introduction

Closure Compiler's type system is often compared to Java's.  The Closure type system does have the concept of classes and interfaces, primitive, and Object types, which is similar to Java's. However, the Java type system is missing many of the features of Closure's and Closure's type system has few of the guarantees of Java's and also lacks a few features.

Closure's type system is also similar to that of TypeScript, as they both aim to type a very dynamic language, but has some key differences.

# Details

Closure type system:
- is optional
- has unions
- has the concept of non-nullable Object types (null and undefined are represented as distinct types)
- has record types (structural types)
- has an "unknown" type (annotated with "?")
- has an "all" type for primitives and Objects (annotated with "`*`")
- has function types
- uses type inference
- has template types (similar to generic types in Java, but Closure lacks bounded generics)
- template types are [invariant](https://medium.com/@thejameskyle/type-systems-covariance-contravariance-bivariance-and-invariance-explained-35f43d1110f8) except for `Array` and `Object` (bivariant) and `Promise` (covariant)

Some differences from Java:
- generic types cannot be bounded, and are always unknown inside functions
- type inference sometimes defaults to an "unknown" type
- type annotations and typecasts are not enforced at runtime

Some differences from TypeScript:
- function subtyping is stricter by default; like TypeScript's [`strictFunctionTypes`](https://www.sitepen.com/blog/typescript-2-6-and-strict-functions/) flag but also applies to method overrides
- no 'tuple types' (e.g. typing an array as `[number, string]` is impossible, it must be `!Array<number|string>`)
- `@interface` is not structural by default, although `@record` is. Regular classes are never structural by default. (See TypeScript's ['duck typing'](https://www.typescriptlang.org/docs/handbook/interfaces.html) vs. [Closure](https://github.com/google/closure-compiler/wiki/Structural-Interfaces-in-Closure-Compiler)). Note that using structural interfaces in Closure may impair its ability to disambiguate unrelated property names in optimizations.

Other references: <br>
[Annotating Types](https://github.com/google/closure-compiler/wiki/Annotating-Types)<br>
[JSDoc Annotations](https://github.com/google/closure-compiler/wiki/Using-JSDoc-Annotations-with-Closure-compiler)<br>
[@struct and @dict Annotations](https://github.com/google/closure-compiler/wiki/@struct-and-@dict-Annotations)<br>
[Types in the Closure Type System](https://github.com/google/closure-compiler/wiki/Types-in-the-Closure-Type-System)<br>

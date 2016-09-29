# Bad Type Annotation - Unknown Type

The Closure Compiler will report an error for types referenced that it does not know about. The error that it produces looks like `Bad type annotation. Unknown type ...`. This error can arise for a few common reasons:

## Typo

The first reason that this error could happen is that there is a typo in the referenced type. Check that the [type is correct](https://github.com/google/closure-compiler/wiki/Types-in-the-Closure-Type-System) and it matches a type defined in file that exists in your project.

## Code is not compatible with closure compiler

Some projects use [JSDoc](http://usejsdoc.org/tags-type.html) annotations which are similar to [closure compiler annotations](https://github.com/google/closure-compiler/wiki/Annotating-JavaScript-for-the-Closure-Compiler) but not entirely the same.  Additionally, there are [assumptions used by the compiler](https://github.com/google/closure-compiler/wiki/Compiler-Assumptions) which code that was not designed with closure compiler in mind might not follow.

## File is not included in the compilation unit

It is also possible that the referenced type is a valid type, but that the file that defines the type is not included in the compilation unit (the list of sources passed to the Closure Compiler). These types are called "forward declared types". The best approach in this situation is to just include the type in the compilation unit. The compiler will then see the full type definition and able to do type checking and optimizations against the code.

If using the Closure dependency system, you can just add a `goog.require` for the type directly in the file. Requiring the file will make sure that the file is included in the compilation unit and that the complete type definition is available. The Closure Compiler linter will also not flag this as an unnecessary require since it understands that the require is used for a type in JS Doc.

There are a few scenarios to watch out for when requiring forward declared types:
*  Circular references - file A uses a type from file B, and file B uses a type from File A. It is impossible for both files to require each other.
*  Late loading code - A type is used in code that is loaded early, but the definition for that type is loaded at a later time in order to defer downloading that code until it is actually necessary.
*  Code with side effects - requiring code to satisfy forward declarations that has known side effects may alter the functionality of your program and the compiler's ability to optimize the code, such as dead code elimination or cross module code motion.

In these situations, there are alternatives to requiring the type that are discussed below.

### Extracting An Interface

When it's not possible to require another class due to circular references or late loading the code, the interface of that functionality could be extracted to a separate file, and that interface could be required. This is very useful since the interface contains all the necessary type information for the type, but is 0 additional code and should not have circular references problems.

When it's not possible to extract an interface to the class, it is possible to forward declare a type, but this has some drawbacks discussed below.

### Forward declaring a type

It is possible to tell the compiler that certain types are valid without requiring the type. You can use the `goog.forwardDeclare` statement in files that reference the type to tell the compiler to not warn about this unknown type.

Example:

```javascript
goog.forwardDeclare('circular.referenced.Type');
```

The `goog.forwardDeclare` function tells the compiler that this is in fact a real type, but the type definition can not be loaded yet, so the compiler should not issue any Bad Type Annotation errors for this type.

When using `goog.forwardDeclare`, the type should always eventually be included in the compilation unit still, even if there is no require for that code. This is because forward declared types which don't exist anywhere in the compilation unit are treated as unknown types, which means that code that interacts with the forward declared type can't be typechecked properly if the type is nowhere in the compilation unit, and optimizations may not work as well. Whenever possible, it is better to ensure that the type is included in the compilation unit for any file that references that type.

If the type is eventually included in the compilation unit anyway, adding a `goog.forwardDeclare` in these cases is not strictly necessary. Since it may mask errors down the line if the type is not included in the compilation unit, you may not want to include this but make sure that developers include the type in their compilation unit in other ways, such as including the right sources from those directories or modules that contain the late loaded code.

## Forbidding Unresolved Types

The Closure Compiler has a [conformance rule](https://github.com/google/closure-compiler/wiki/JS-Conformance-Framework) to forbid unresolved types. This check is great to ensure that the right types are always included, and it is recommended for all projects. To use it, add this blob to your project's conformance config:

```
requirement: {
   type: CUSTOM
   java_class: 'com.google.javascript.jscomp.ConformanceRules$BanUnresolvedType'
   error_message: 'Accessing properties from objects that are forward-declared type names is discouraged. Please require this type.'
 }
 ```
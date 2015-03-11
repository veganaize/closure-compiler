# Overview of Closure's Type System for JavaScript

# Introduction

Closure Compiler's type system is often compared to Java's.  Closure type system does have the concept of classes and interfaces, primitive and Object types which is similar to Java's but Java type system is missing many of the features of Closure's and Closure's type system has few of the guarantees of Java's.

# Details

Closure type system:
- is optional
- has unions
- has the concept of non-nullable Object types (null and undefined are represented as distinct types)
- has record types (structural types)
- has an "unknown" type (annotated with "?")
- has an "all" type for primitives and Objects (annotated with "`*`")
- has function types
- uses type inferrence



Other references: <br>
AnnotatingTypes<br>
StructDictAnnotations<br>

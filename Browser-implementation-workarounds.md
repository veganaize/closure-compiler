# Workarounds

Closure library and compiler work around a number of browser ES6+ implementation issues.

This page tracks those issues so that
- users who are already aware of these bugs can check if Closure handles them correctly
- compiler developers can remove these workarounds if the bugs are fixed and we drop support for the buggy version.

## Edge

* [ChakraCore #1496](https://github.com/Microsoft/ChakraCore/issues/1496) is an optimizer bug that causes constructor invocations to return the class instead of the instance.  Closure Library explicitly checks the user agent for Edge and considers it a non-ES6-compliant browser as a result.
* [ChakraCore #3217](https://github.com/Microsoft/ChakraCore/issues/3217) is a bug in `Reflect.construct` that prevents the compiler's super() call transpilation from working correctly.  The compiler checks for this bug and uses a slower polyfill that calls `Reflect.setPrototypeOf` instead.

## Safari

* Safari 10 Object.seal(class C{}) throws an exception stating that the properties are not configurable, but not 10.2 so no bug was reported.  Closure Library avoids calling Object.seal on functions for goog.module exports.
* [Webkit #167328](https://bugs.webkit.org/show_bug.cgi?id=167328) Safari 10 incorrectly scopes functions in eval.  Closure Library, when this is detected, avoid the problem when loading goog.modules by wrapping code the code in an IIFE which has the correct scoping.
* [Webkit #182414](https://bugs.webkit.org/show_bug.cgi?id=182414) Safari incorrectly treats var redeclarations of parameters as rewriting the parameter, instead of ignoring it. One particular compiler optimization back offs to avoid triggering this (https://github.com/google/closure-compiler/issues/3164) but the compiler is still at risk of outputting code triggering the bug.
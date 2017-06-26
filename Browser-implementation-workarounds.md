# Workarounds

Closure works around a number of browser implementation issues.

## Edge

* [ChakraCore #1496](https://github.com/Microsoft/ChakraCore/issues/1496) is an optimizer bug that causes constructor invocations to return the class instead of the instance.  Closure Library explicitly checks the user agent for Edge and considers it a non-ES6-compliant browser as a result.
* [ChakraCore #3217](https://github.com/Microsoft/ChakraCore/issues/3217) is a bug in `Reflect.construct` that prevents the compiler's super() call transpilation from working correctly.  The compiler checks for this bug and uses a slower polyfill that calls `Reflect.setPrototypeOf` instead.

## Safari

* TODO(@concavelenz): eval scope issues, etc
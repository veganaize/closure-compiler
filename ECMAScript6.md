# Status of ES6+ support in Closure Compiler

# Introduction

ECMAScript 2015 - 2020 are now officially supported as both input and output language for the Closure Compiler.

If you find that the Closure Compiler does not support your favorite feature, you can file an issue. 

This is a place to collect changes that will need to be made to the compiler.

## Overview

Summary: https://github.com/lukehoban/es6features

Current draft Spec: https://tc39.github.io/ecma262/
Finished proposals: https://github.com/tc39/proposals/blob/master/finished-proposals.md

# Details

## Transpilation

We support transpilation of the following ES6 features down to ES5/3.

* let/const
* arrow functions
* default parameters and rest ("...") parameters in functions
* The spread ("...") operator in function calls and array literals
* Classes (ES6 classes will be @struct by default. You can add @unrestricted to override that behavior. See the ['JSDoc Tag Reference'](https://github.com/google/closure-compiler/wiki/Annotating-JavaScript-for-the-Closure-Compiler#struct)
* Computed properties and short properties in object literals.
* Method declarations in object literals
* "for of" loops
* generator functions
* template strings
* Better unicode escapes in string literals (e.g. "\u{1F436}" == "🐶")
* destructuring assignment
* modules
* async functions
* async generators
* `for await (const item of asyncIterator) {}`

## Type Checking

Type checking is done on the original code before transpilation.

## Optimizations

Optimization currently occur after the code is transpiled to your chosen output language.


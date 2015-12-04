# Status of ES6 support in Closure Compiler

# Introduction

ECMAScript 6 is now officially supported as an input language for the Closure Compiler. You cannot use it as an output language yet; for now you need to transpile down to ES5 or ES3. If you find that the Closure Compiler does not support your favorite ES6 feature, you can file an issue, but please be aware that supporting the entire ES6 specification is a non-goal for the Closure Compiler. We currently avoid features that cannot be transpiled cleanly to ES5, and features that tend to lead to code which is difficult to analyze and typecheck. 

This is a place to collect changes that will need to be made to the compiler.

## Overview

Summary: https://github.com/lukehoban/es6features

Spec: http://people.mozilla.org/~jorendorff/es6-draft.html

# Details

## Parser

The new parser is in place, and has replaced the old Rhino parser. It understands all ES6 features though there may still be a few bugs here and there.

## Transpilation

We support transpilation of the following ES6 features down to ES5/3.

* let/const
* arrow functions
* default parameters and rest ("...") parameters in functions
* The spread ("...") operator in function calls and array literals
* Classes (ES6 classes will be @struct by default. You can add @unrestricted to override that behavior. See the 'JSDoc Tag Reference' section of the [Google JS style guide](https://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml) for information about the @struct annotation.)
* Computed properties and short properties in object literals.
* Method declarations in object literals
* "for of" loops
* generator functions
* template strings
* Better unicode escapes in string literals (e.g. "\u{1F436}" == "🐶")
* destructuring assignment
* modules

## Type Checking

Type checking is currently done on "transpiled" code.  Direct support will happen after the new type inference is stable.

## Optimizations

Optimization currently occur on "transpiled" code.  Direct support will happen after the "check" phase understands ES6.

## Code Printer

The code printer should at this point support all of ES6
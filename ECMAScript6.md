# Status of ES6 support in Closure Compiler

# Introduction

There are major changes in store for EcmaScript 6 and will require major changes to Closure Compiler to support it.

This is a place to collect changes that will need to be made to the compiler.

## Overview

Summary: https://github.com/lukehoban/es6features

Spec: http://people.mozilla.org/~jorendorff/es6-draft.html

# Details

## Parser

The new parser is in place, and will be enabled by default in the next release (probably in May). It already parses a lot of the new ES6 syntax, and we plan to be able to parse nearly all of ES6 by the end of Q2 2014.

## Type Checking

## Optimizations

## Code Printer

In progress.
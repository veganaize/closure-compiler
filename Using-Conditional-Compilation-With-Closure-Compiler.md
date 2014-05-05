# Using Internet Explorer's "conditional compilation" with Closure Compiler

# Introduction

Closure Compiler does not natively support Internet Explorer's [conditional compilation](http://msdn.microsoft.com/en-us/library/121hztk3(VS.71).aspx), however it is still possible to use them in a limited way.

# Details

IE's conditional compilation can not be used directly, but they can be used in eval conditions.  As an example, one common use is to detect Internet Explorer:

`var is_ie = /*@cc_on!@*/false;`

This can be rewritten to be compatible with Closure Compiler in this way:

`var is_ie = eval("/*@cc_on!@*/false");`

The rules for using this solution are the same for the use of eval in general.
# Introduction

To facilitate better optimization in the common case the Closure Compiler assumes `toString` and `valueOf` implementation are effectively side-effect free.  

# Details

In EcmaScript, `toString` and `valueOf` are implicitly called when type conversions are necessary, for example `'' + x` is equivalent to `'' + x.valueOf()` or `'' + x.toString()` if `valueOf` is not defined. A `valueOf` or `toString` with side-effects can be called in surprising places.  A `toString` implementation such as `Foo.prototype.toString = function() {alert("hi")}` is a useful learning exercise but allowing for such behavior prohibit many basic optimizations, such as:
` 'a' + 'b' + 'c' ==> 'abc' `
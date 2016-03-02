_Note: The lint check about `arguments` has been removed in https://github.com/google/closure-compiler/commit/3734b26b1215693d5c46636d91b7a5bb2715f690, but the following patterns are still discouraged._

#### What's up with the `arguments` object?

`arguments` is a variable that is in scope in every function, which
more or less acts like an array, but unfortunately is not an array.

There is a lint check that warns about many usages of the `arguments` variable.
If you're iterating over it in a for loop, or accessing specific elements of it,
there is no problem. But if you're using `Array.prototype.slice` to convert it
to an Array, or passing it to another function, the linter will give you a
warning that looks like:

    WARNING - This use of the arguments object is discouraged.
      var args = Array.prototype.slice.call(arguments);
                                            ^

These usages are discouraged. If you pass the Arguments object to another function, it can result in a significant slowdown. See discussion on [this bug](https://github.com/google/closure-compiler/issues/1015) for more details.

We make an exception for `Function.prototype.apply` so there is no warning for `f.apply(obj, arguments);`.
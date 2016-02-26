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

These usages are discouraged for the following reasons:

1. If you pass the Arguments object to another function, it can result in a
   significant slowdown. See discussion on [this bug](https://github.com/google/closure-compiler/issues/1015)
   for more details.
1. If you pass the Arguments object to another function and then use ES6
   destructuring, or use the object in a for/of loop, the compiler's ES6-to-ES3
   transpilation will not handle it correctly. For example,

   ```js
   function f(x) {
     return [...x];
   }

   function g() {
     f(arguments);
   }
   ```

   will transpile to

   ```js
   function f(x) {
     return [].concat($jscomp.arrayFromIterable(x));
   }

   function g() {
     f(arguments);
   }
   ```

   and `$jscomp.arrayFromIterable` will not successfully convert the Arguments
   object into an Iterable in pre-ES6 browsers. This is tracked at [#1303](https://github.com/google/closure-compiler/issues/1303).

We make an exception for `Function.prototype.apply` so there is no warning for
`f.apply(obj, arguments);`.

In most other cases, simply iterating over `arguments` with a `for` loop is
more performant and should be preferred.
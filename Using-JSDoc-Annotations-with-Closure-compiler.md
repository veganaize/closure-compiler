The Closure-compiler uses a subset of the [JSDoc annotations][1] (and adds a few of its own). See the [annotation reference for the compiler][2] for the complete set. A JSDoc annotation is similar to a JavaDoc annotation and is a comment block that begins with `/**` (two stars). While each line of the comment often begins with it's own `*`, that is a convention that is not required.

    /**
     * @fileoverview I am a JSDoc comment block.
     */

Syntax
=========
JSDoc annotations must be at the start of the line. Multiple annotations may be listed on the same line, but comment text can only follow the last annotation.

    /** @const @type {string} this var is a string */ var foo;

JSDoc annotation arguments can span multiple lines.

    /**
     * This annotation spans multiple lines for readability
     * @type {{
     *    id:number,
     *    val:string,
     *    data:object
     *  }}
     */
     var foo;

The annotation typically applies to the following statement. Here are some examples:

Variable
-------

    /** @type {string} */ var a;

Type Cast
--------

    var b = /** @type {string} */ (window['foo']);
*note the extra parenthesis*

Named Function
--------

    /**
     * @param {string} bar
     * @return {boolean}
     */
    function foo(bar) { return true; }

Alternatively, you can annotate the parameters and the return type inline.

    function /** number */ foo(/** number */ n) { return n - 1; }

Function Expressions
-------------------

    /** @type {function(string):boolean} */
    var foo = function(bar) { return true; }

    var foo2 =
      /**
       * @param {string} bar
       * @return {boolean}
       */
      function(bar) { return true; }

Typedef
--------

Complex types (including unions, and record types) can be aliased for convenience and maintainability using a typedef. These annotations can be long, but can be split over multiple lines for readability.

    /** 
     * @typedef {{
     *            foo:string,
     *            bar:number,
     *            foobar:number|string
     *          }}
     */
    var Mytype;

    /** @param {MyType} bar */
    function foo(bar) {}

To see a wide range of possible annotations, take a look at the [extern files in the Compiler project][3].

  [1]: http://code.google.com/p/jsdoc-toolkit/
  [2]: https://developers.google.com/closure/compiler/docs/js-for-compiler
  [3]: https://github.com/google/closure-compiler/tree/master/contrib/externs
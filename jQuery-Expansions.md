# How the compiler handles jQuery style code.

## Introduction

jQuery uses conventions that Closure-Compiler doesn’t natively understand. In order to produce functioning code, remove warnings and enable optimizations, Closure-compiler expands known jQuery methods and aliases in a similar way to how macros are expanded in other languages. Specifically certain calls to <b>jQuery.extend</b> and any call to <b>jQuery.expandedEach</b> may look like a function call, but won’t have the same semantics (ex: aliased versions of these functions will not be expanded).

## The --process_jquery_primitives Flag

Using the "--process_jquery_primitives" flag causes the compiler to use the jQuery Coding Convention and enables the ExpandJqueryAliases pass. jQuery will not successfully compile without this flag. This flag has no effect unless the compilation level is also set at ADVANCED_OPTIMIZATIONS.

## The jQuery.fn Alias

<b>jQuery.fn</b> is an alias for the <b>jQuery.prototype</b> property. All <b>jQuery.fn</b> references are replaced with <b>jQuery.prototype</b> references so that the compiler can properly recognize these methods.

## jQuery.extend

<b>jQuery.extend</b> is used to add properties and methods to objects including adding prototypes and static methods to the jQuery object and namespace. So that the compiler can recognize these assignments and be able to optimize them (or remove them in the case of dead code) <b>jQuery.extend</b> calls are expanded into direct assignments when one of the following conditions is met:

<ol><li>jQuery.extend is called with a single argument that is an object literal. The jQuery namespace is extended.

Example:
<code>jQuery.extend({meth1:function(){}, meth2:function() {}})</code>
becomes
<code>jQuery.meth1 = function(){}; jQuery.meth2 = function(){};</code>
</li>
<li>jQuery.extend is called with two arguments and the first argument is a name or property (not a literal) and the second argument is an object literal.

Example:
<code>jQuery.extend(myobj, {meth1:ref1, meth2:ref2})</code>
becomes
<code>myobj.meth1 = ref1; myobj.meth2 = ref2;</code>
</li>
</ol>
If the jQuery.extend return value is used, the expansion is wrapped in an immediately executed anonymous function which returns the object being extended.

## jQuery.expandedEach

<b>jQuery.each</b> causes problems with Closure-Compiler when it is used to assign properties to objects in an outer scope. For instance given the following code:
<code>jQuery.each([function(i, val) {
    jQuery.prototype\[val\](“trim”],) = String.prototype.trim;
});</code>

Closure-compiler can not natively recognize that <b>jQuery.prototype.trim</b> now exists. However many uses of <b>jQuery.each</b> do not cause any issues and should be left alone. In addition, the use of the [syntax instead of a dotted syntax blocks Closure-Compiler renaming and dead-code elimination.

To address these issues, the <b>jQuery.each</b> function should be aliased as <b>jQuery.expandedEach</b>. Closure compiler recognizes <b>jQuery.expandedEach</b> calls and unravels the loop into a series of immediately executed functions. It also converts assignments using [](]) syntax to “.” syntax so that renaming and dead-code elimination will occur.

<b>jQuery.expandedEach</b> calls must meet the following criteria:
<ul><li>The first argument MUST be an object or array of strings literal. The expansion happens before inlining optimizations so variable and property references cannot be used.</li>
<li>The object properties or array strings MUST be valid property names without requiring quotes.</li>
<li>The callback function arguments MUST be used in an assignment to an object in an outside scope.</li>
</ul>

Example:
<code>jQuery.expandedEach(
    {width: function() {}, height: function() {}},
    function(key, val) {
        jQuery.fn[key] = val;
    }
);</code>
becomes
<code>(function() {
    jQuery.fn.width = function() {};
})();
(function() {
    jQuery.fn.height = function() {};
})();
</code>
In most cases the extra function wrappers will be removed by later Closure-Compiler passes.

Developers who wish to use <b>jQuery.expandedEach</b> should recreate the alias within their code so that it works with both a compiled and uncompiled build of jQuery.

Example:

<code>jQuery.expandedEach = jQuery.each;</code>
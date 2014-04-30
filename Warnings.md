# How to tell closure compiler which warnings you want

## Introduction

Closure Compiler has a `--warning_level` flag that gives you a few high-level options to control what warnings you see. The `--warning_level` flag gives you an easy way to choose the most common options. But even `--warning_level=VERBOSE` doesn't give you all the warnings that Closure Compiler can emit, and sometimes you want more fine-grained control.

## Warnings Level API

Closure Compiler has an API for configuring the errors and warnings that you would like to see, and what level they're emitted at. 

<table>
  <tr><td>**Flag API**</td><td>**Java API**</td><td>**Effect**</td></tr>
  <tr><td>`--jscomp_error=<type>`</td><td>`options.setWarningLevel(<type>, CheckLevel.ERROR);`</td><td>Makes all warnings of the given type to build-breaking error.</td></tr>
  <tr><td>`--jscomp_warning=<type>`</td><td>`options.setWarningLevel(<type>, CheckLevel.WARNING);`</td><td>Makes all warnings of the given type a non-breaking warning.</td></tr>
  <tr><td>`--jscomp_off=<type>`</td><td>`options.setWarningLevel(<type>, CheckLevel.OFF);`</td><td>Silences all warnings of the given type.</td></tr>
</table>

## Warnings Categories

In the examples above, `<type>` is a pre-defined category of warnings.

<table>
  <tr><td>**Flag API**</td><td>**Java API**</td><td>**Effect**</td><td>**Default Value**</td></tr>
  <tr><td>accessControls</td><td>`DiagnosticGroups.ACCESS_CONTROLS`</td><td>Warnings when @deprecated, @private, or @protected are violated.</td><td>OFF</td></tr>
  <tr><td>ambiguousFunctionDecl</td><td>`DiagnosticGroups.AMBIGUOUS_FUNCTION_DECL`</td><td>Warnings about ambiguous definitions of functions</td><td>WARNING</td></tr>
  <tr><td>checkDebuggerStatement</td><td>`DiagnosticGroups.DEBUGGER_STATEMENT_PRESENT`</td><td>Check for the use of the `debugger` statement</td><td>OFF</td></tr>
  <tr><td>checkRegExp</td><td>`DiagnosticGroups.CHECK_REGEXP`</td><td>Check for regular expression literal problems</td><td>OFF</td></tr>
  <tr><td>checkTypes</td><td>`DiagnosticGroups.CHECK_TYPES`</td><td>Type-checking</td><td>OFF by default, WARNING on `--warning_level=VERBOSE`</td></tr>
  <tr><td>checkVars</td><td>`DiagnosticGroups.CHECK_VARS`</td><td>Warnings when vars are not declared.</td><td>OFF by default, ERROR on `--warning_level=VERBOSE`</td></tr>
  <tr><td>const</td><td>`DiagnosticGroups.CONST`</td><td>Warnings when @const annotated properties or variables are reassigned or deleted.</td><td>OFF</td></tr>
  <tr><td>constantProperty</td><td>`DiagnosticGroups.CONSTANT_PROPERTY`</td><td>Warnings when @const annotated properties are reassigned or deleted.</td><td>OFF</td></tr>
  <tr><td>deprecated</td><td>`DiagnosticGroups.DEPRECATED`</td><td>Warnings when non-deprecated code accesses code that's marked @deprecated</td><td>OFF by default, WARNING on `--warning_level=VERBOSE`</td></tr>
  <tr><td>duplicate</td><td>`DiagnosticGroups.DUPLICATE`</td><td>Warnings when a variable is declared twice in the global scope</td><td>OFF by default, ERROR on `--warning_level=VERBOSE`</td></tr>
  <tr><td>es5Strict</td><td>`DiagnosticGroups.ES5_STRICT`</td><td>Warnings about code that is invalid in EcmaScript 5 Strict mode</td><td>WARNING</td></tr>
  <tr><td>externsValidation</td><td>`DiagnosticGroups.EXTERNS_VALIDATION`</td><td>Warnings about malformed externs files</td><td>WARNING</td></tr>
  <tr><td>fileoverviewTags</td><td>`DiagnosticGroups.FILEOVERVIEW_JSDOC`</td><td>Warnings about duplicate @fileoverview tags</td><td>WARNING</td></tr>
  <tr><td>globalThis</td><td>`DiagnosticGroups.GLOBAL_THIS`</td><td>Warnings about [improper use of the global this](http://closuretools.blogspot.com/2010/10/this-this-is-your-this.html).</td><td>OFF by default, WARNING on `--warning_level=VERBOSE`</td></tr>
  <tr><td>internetExplorerChecks</td><td>`DiagnosticGroups.INTERNET_EXPLORER_CHECKS`</td><td>Warnings about syntax errors on Internet Explorer, like trailing commas.</td><td>ERROR</td></tr>
  <tr><td>invalidCasts</td><td>`DiagnosticGroups.INVALID_CASTS`</td><td>Warnings about invalid type casts.</td><td>OFF by default, WARNING when type checking is on</td></tr>
  <tr><td>missingProperties</td><td>`DiagnosticGroups.MISSING_PROPERTIES`</td><td>Warnings about whether a property will ever be defined on an object. Part of type-checking.</td><td>OFF by default, WARNING on `--warning_level=VERBOSE`</td></tr>
  <tr><td>nonStandardJsDocs</td><td>`DiagnosticGroups.NON_STANDARD_JS_DOCS`</td><td>Warnings when JSDoc has annotations that the compiler thinks you misspelled.</td><td>WARNING</td></tr>
  <tr><td>strictModuleDepCheck</td><td>`DiagnosticGroups.STRICT_MODULE_DEP_CHECK `</td><td>Warnings about all references potentially violating module dependencies</td><td>OFF</td></tr>
  <tr><td>suspiciousCode</td><td>`DiagnosticGroups.SUSPICIOUS_CODE`</td><td>Warning about things like missing missing semicolons and comparisons to NaN</td><td>WARNING</td></tr>
  <tr><td>undefinedNames</td><td>`DiagnosticGroups.UNDEFINED_NAMES`</td><td>Warnings when properties of global names are undefined.</td><td>OFF by default, WARNING on `--warning_level=VERBOSE`</td></tr>
  <tr><td>undefinedVars</td><td>`DiagnosticGroups.UNDEFINED_VARS`</td><td>Warnings when variables are undefined.</td><td>OFF by default, ERROR on `--warning_level=VERBOSE`</td></tr>
  <tr><td>unknownDefines</td><td>`DiagnosticGroups.UNKNOWN_DEFINES`</td><td>Warnings when unknown @define values are specified.</td><td>WARNING</td></tr>
  <tr><td>uselessCode</td><td>`DiagnosticGroups.USELESS_CODE`</td><td>Warnings when the compiler sees useless code that it plans to remove.</td><td>WARNING</td></tr>
  <tr><td>visibility</td><td>`DiagnosticGroups.VISIBILITY`</td><td>Warnings when @private and @protected are violated.</td><td>OFF</td></tr>
</table>

## Defining Your Own Categories

If there is a category of warnings that you would like to configure but is not listed in the table above, it is easy to define a new one.

   1. Find the unique identifier of the warnings that you'd like to configure.
   2. Create a new [DiagnosticGroup](http://code.google.com/p/closure-compiler/source/browse/src/com/google/javascript/jscomp/DiagnosticGroup.java) that gives this warning category a human-readable name, and add it to our central repository of warnings categories:
      http://code.google.com/p/closure-compiler/source/browse/src/com/google/javascript/jscomp/DiagnosticGroups.java
   3. Send us a patch!

This is how we "canary" new warnings until we feel they're stable.

## @suppress Tags

You can also silence warnings by adding JSDoc annotations to your code. All of the warnings categories above can be used in a @suppress tag in a @fileoverview JSDoc comment or a function JSDoc comment. For example,

    /**
     * @fileoverview This is a file where deprecation checks are disabled.
     * @suppress {deprecated}
     */
    
    /**
     * This is a function where type checking is disabled.
     * @suppress {checkTypes}
     */
    function f() { /** ... */ }

There is also one warning category that you can suppress with @suppress tags, but is not available from the command-line.

    /** @suppress {duplicate} */ foo.prop = ...
Suppresses warnings about a declaration of the same method or property twice in the global scope.
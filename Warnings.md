# How to tell closure compiler which warnings you want

## Introduction

Closure Compiler has a `--warning_level` flag that gives you a few high-level options to control what warnings you see. The `--warning_level` flag gives you an easy way to choose the most common options. But even `--warning_level=VERBOSE` doesn't give you all the warnings that Closure Compiler can emit, and sometimes you want more fine-grained control.

## Warnings Level API

Closure Compiler has an API for configuring the errors and warnings that you would like to see, and what level they're emitted at. 

**Flag API**|**Java API**|**Effect**
------------|------------|----------
`--jscomp_error=<type>`|`options.setWarningLevel(<type>, CheckLevel.ERROR);`|Makes all warnings of the given type to build-breaking error.
`--jscomp_warning=<type>`|`options.setWarningLevel(<type>, CheckLevel.WARNING);`|Makes all warnings of the given type a non-breaking warning.|
`--jscomp_off=<type>`|`options.setWarningLevel(<type>, CheckLevel.OFF);`|Silences all warnings of the given type.

## Warnings Categories

In the examples above, `<type>` is a pre-defined category of warnings.

Run the compiler with the --help flag to get a comprehensive up-to-date list.

The following table describes the available diagnostic groups.  The name used with the --jscomp_off/warning/error flags and DiagnosticGroups member name that can be used to reference the group via the Java API. 

Name|Field|Effect|Default Value
------------|------------|----------|-----------------
accessControls|`ACCESS_CONTROLS`|Warnings when @deprecated, @private, or @protected are violated.|OFF
ambiguousFunctionDecl|`AMBIGUOUS_FUNCTION_DECL`|Warnings about ambiguous definitions of functions|WARNING
checkDebuggerStatement|`DEBUGGER_STATEMENT_PRESENT`|Check for the use of the `debugger` statement|OFF
checkRegExp|`CHECK_REGEXP`|Check for regular expression literal problems|OFF
checkTypes|`CHECK_TYPES`|Type-checking|OFF by default, WARNING on `--warning_level=VERBOSE`
checkVars|`CHECK_VARS`|Warnings when vars are not declared.|OFF by default, ERROR on `--warning_level=VERBOSE`
const|`CONST`|Warnings when @const annotated properties or variables are reassigned or deleted.|OFF
constantProperty|`CONSTANT_PROPERTY`|Warnings when @const annotated properties are reassigned or deleted.|OFF
deprecated|`DEPRECATED`|Warnings when non-deprecated code accesses code that's marked @deprecated|OFF by default, WARNING on `--warning_level=VERBOSE`
duplicate|`DUPLICATE`|Warnings when a variable is declared twice in the global scope|OFF by default, ERROR on `--warning_level=VERBOSE`
es5Strict|`ES5_STRICT`|Warnings about code that is invalid in EcmaScript 5 Strict mode|WARNING
externsValidation|`EXTERNS_VALIDATION`|Warnings about malformed externs files|WARNING
fileoverviewTags|`FILEOVERVIEW_JSDOC`|Warnings about duplicate @fileoverview tags|WARNING
globalThis|`GLOBAL_THIS`|Warnings about [improper use of the global this](http://closuretools.blogspot.com/2010/10/this-this-is-your-this.html).|OFF by default, WARNING on `--warning_level=VERBOSE`
internetExplorerChecks|`INTERNET_EXPLORER_CHECKS`|Warnings about syntax errors on Internet Explorer, like trailing commas.|ERROR
invalidCasts|`INVALID_CASTS`|Warnings about invalid type casts.|OFF by default, WARNING when type checking is on
missingProperties|`MISSING_PROPERTIES`|Warnings about whether a property will ever be defined on an object. Part of type-checking.|OFF by default, WARNING on `--warning_level=VERBOSE`
nonStandardJsDocs|`NON_STANDARD_JS_DOCS`|Warnings when JSDoc has annotations that the compiler thinks you misspelled.|WARNING
strictModuleDepCheck|`STRICT_MODULE_DEP_CHECK `|Warnings about all references potentially violating module dependencies|OFF
suspiciousCode|`SUSPICIOUS_CODE`|Warning about things like missing missing semicolons and comparisons to NaN|WARNING
undefinedNames|`UNDEFINED_NAMES`|Warnings when properties of global names are undefined.|OFF by default, WARNING on `--warning_level=VERBOSE`
undefinedVars|`UNDEFINED_VARS`|Warnings when variables are undefined.|OFF by default, ERROR on `--warning_level=VERBOSE`
unknownDefines|`UNKNOWN_DEFINES`|Warnings when unknown @define values are specified.|WARNING
uselessCode|`USELESS_CODE`|Warnings when the compiler sees useless code that it plans to remove.|WARNING
visibility|`VISIBILITY`|Warnings when @private and @protected are violated.|OFF
--------------------------------------------------------------------------------------------------

## Defining Your Own Categories
    
If there is a category of warnings that you would like to configure but is not listed in the table above, it is easy to define a new one.
     
   1. Find the unique identifier of the warnings that you'd like to configure.
   2. Create a new [DiagnosticGroup](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/DiagnosticGroup.java) that gives this warning category a human-readable name, and add it to our central repository of warnings categories:
      https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/DiagnosticGroups.java
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
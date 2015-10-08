# How to tell closure compiler which warnings you want

## Introduction

Closure Compiler has a `--warning_level` flag that gives you a few high-level options to control what warnings you see. The `--warning_level` flag gives you an easy way to choose the most common options. But even `--warning_level=VERBOSE` doesn't give you all the warnings that Closure Compiler can emit, and sometimes you want more fine-grained control.

## Warnings Level API

Closure Compiler has an API for configuring the errors and warnings that you would like to see, and what level they're emitted at. 

**Flag API**|**Java API**|**Effect**
------------|------------|----------
`--jscomp_error=<group>`|`options.setWarningLevel("<group>", CheckLevel.ERROR);`|Makes all warnings of the given group to build-breaking error.
`--jscomp_warning=<group>`|`options.setWarningLevel("<group>", CheckLevel.WARNING);`|Makes all warnings of the given group a non-breaking warning.|
`--jscomp_off=<group>`|`options.setWarningLevel("<group>", CheckLevel.OFF);`|Silences all warnings of the given group.

## Warnings Categories

In the examples above, `<group>` is a pre-defined category of warnings.

Run the compiler with the --help flag to get a comprehensive up-to-date list. This may be out of date. Or consult [DiagnosticGroups.java](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/DiagnosticGroups.java#L87). 

The following table describes the available diagnostic groups.

Name|Effect|Default Value
----|------|-----------------
accessControls|Warnings when @deprecated, @private, @protected, or @package are violated.|OFF
ambiguousFunctionDecl|Warnings about ambiguous definitions of functions. On Chrome, <br/> if (false) { function f() {} } <br/> declared 'f' in the global scope anyway. Future versions of javascript forbid this, because the <br/> actual semantics differ between browsers.|ERROR
checkDebuggerStatement|Warnings when the 'debugger' keyword is used.|OFF
checkEventfulObjectDisposal|Warnings about undisposed eventful objects.|OFF
checkRegExp|Warnings about weird regular expression literals|OFF
checkTypes|Type-checking|WARNING
checkVars|Warnings when vars are not declared or declared multiple times in the global scope.|ERROR
closureDepMethodUsageChecks|Warnings about misused goog.provide/goog.require calls|ERROR
conformanceViolations|Warnings about conformance violations and possible conformance violations.|WARNING
constantProperty|Warnings when a member property marked @const is reassigned.|OFF
const|Warnings when a variable or member property marked @const is reassigned.|OFF
deprecatedAnnotations|Warnings when using annotations that are deprecated|OFF
deprecated|Warnings when non-deprecated code accesses code that's marked @deprecated|OFF
duplicateMessage|Warnings when two i18n messages have the same id|ERROR
duplicate|Warnings when a variable is declared twice in the global scope|ERROR
es3|Warnings about EcmaScript3|ERROR
es5Strict|Warnings about EcmaScript5 strict mode|ERROR
externsValidation|Warnings about malformed externs files|WARNING
extraRequire|Warnings about unnecessary goog.require calls|OFF
fileoverviewTags|Warnings about duplicate @fileoverview tags|WARNING
globalThis|Warnings about [improper use of the global this](http://closuretools.blogspot.com/2010/10/this-this-is-your-this.html).|WARNING
inferredConstCheck|Warnings if a @const declaration contains no declared type and the compiler cannot infer one|OFF
internetExplorerChecks|Warnings about syntax error on Internet Explorer.|ERROR
invalidCasts|Warnings about invalid type casts.|WARNING
misplacedTypeAnnotation|Warnings about jsdoc type annotations that are misplaced|WARNING
missingGetCssName|Warnings about strings that should only be used inside calls to goog.getCssName|OFF
missingProperties|Warnings about whether a property will ever be defined on an object. Part of type-checking.|OFF
missingProvide|Warnings if missing a goog.provide('Foo') when defining a Foo class|OFF
missingRequire|Warnings if missing a goog.require('Foo') when a new Foo() is encountered|OFF
missingReturn|Warnings if missing return in a function which a non-void return type|OFF
newCheckTypes|Warnings from new type checker|OFF
nonStandardJsDocs|Warnings when JSDoc has annotations that the compiler thinks you misspelled.|WARNING
reportUnknownTypes|Warnings for any place in the code where type is inferred to ?. NOT RECOMMENDED!|OFF
strictModuleDepCheck|Warnings about all references potentially violating module dependencies|OFF
suspiciousCode|Warning about things like missing semicolons and comparisons to NaN|WARNING
tweakValidation|Warnings about goog.tweak primitives|OFF
typeInvalidation|Warn about properties that cannot be disambiguated when using type based optimizations|OFF
undefinedNames|Warnings when a property of a global name is not defined.|OFF
undefinedVars|Warnings when a variable is never defined.|ERROR
unknownDefines|Warnings when unknown @define values are specified.|WARNING
useOfGoogBase|Warnings aout usages of goog.base, which is not compatible with strict mode|OFF
uselessCode|Warnings when the compiler sees useless code that it plans to remove.|WARNING
violatedModuleDep|Warnings when there are references in earlier modules to variables defined in later modules|ERROR
visibility|Warnings when @private and @protected are violated.|OFF
--------------------------------------------------------------------------------------------------

## Defining Your Own Categories
    
If there is a category of warnings that you would like to configure but is not listed in the table above, it is easy to define a new one.
     
   1. Find the unique identifier of the warnings that you'd like to configure.
   2. Create a new [DiagnosticGroup](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/DiagnosticGroup.java) that gives this warning category a human-readable name, and add it to our central repository of warnings categories:
      https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/DiagnosticGroups.java
   3. Send us a patch!
    
This is how we "canary" new warnings until we feel they're stable.

## @suppress Tags
    
You can also silence warnings by adding JSDoc annotations to your code. All of the warnings categories above should be available for use in a @suppress tag in a @fileoverview JSDoc comment or a function JSDoc comment. However check [ParserConfig.properties](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/parsing/ParserConfig.properties#L131) for a comprehensive list of suppressible warnings.

For example,

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

If you want to suppress many warnings at once, you can write

    /** @suppress {duplicate, accessControls} */ function f() { ... }
or

    /** @suppress {duplicate|accessControls} */ function f() { ... }
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

The following table describes the available diagnostic groups, but may be out of date.

Additional information:
 * Run the compiler with the --help flag to get a comprehensive up-to-date list.
 * Read the [@suppress annotation](https://github.com/google/closure-compiler/wiki/@suppress-annotations) documentation to suppress a specific error in source code.
 * Consult [DiagnosticGroups.java](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/DiagnosticGroups.java#L87). 

Name|Effect|Default Value
----|------|-----------------
accessControls|Warnings when @deprecated, @private, @protected, or @package are violated.|OFF
ambiguousFunctionDecl|[DELETED in v20200101] Warnings about ambiguous definitions of functions. On Chrome, <br/> if (false) { function f() {} } <br/> declared 'f' in the global scope anyway. Future versions of javascript forbid this, because the <br/> actual semantics differ between browsers.|ERROR
checkDebuggerStatement|Warnings when the 'debugger' keyword is used.|OFF
checkEventfulObjectDisposal|[DELETED in v20191111] Warnings about undisposed eventful objects.|OFF
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
es3|[DELETED in v20200101] Warnings about EcmaScript3|ERROR
es5Strict|Warnings about EcmaScript5 strict mode. See [here](https://github.com/google/closure-compiler/issues/3014) for how to turn these off.|ERROR
externsValidation|Warnings about malformed externs files|WARNING
extraRequire|Warnings about unnecessary goog.require calls|OFF
fileoverviewTags|[DELETED in v20200101] Warnings about duplicate @fileoverview tags|WARNING
globalThis|Warnings about [improper use of the global this](http://closuretools.blogspot.com/2010/10/this-this-is-your-this.html).|WARNING
internetExplorerChecks|[DELETED in v20200101] Warnings about syntax error on Internet Explorer.|ERROR
invalidCasts|Warnings about invalid type casts.|WARNING
misplacedSuppress|Warnings about misplaced @suppress annotations|WARNING
misplacedTypeAnnotation|Warnings about jsdoc type annotations that are misplaced|WARNING
missingGetCssName|Warnings about strings that should only be used inside calls to goog.getCssName|OFF
missingProperties|Warnings about whether a property will ever be defined on an object. Part of type-checking.|OFF
missingProvide|Warnings if missing a goog.provide('Foo') when defining a Foo class|OFF
missingRequire|Warnings if missing a goog.require('Foo') when a new Foo() is encountered|OFF
missingReturn|Warnings if missing return in a function which a non-void return type|OFF
newCheckTypes|[DELETED in v20200101] Warnings from new type checker|OFF
nonStandardJsDocs|Warnings when JSDoc has annotations that the compiler thinks you misspelled.|WARNING
reportUnknownTypes|Warnings for any place in the code where type is inferred to ?. NOT RECOMMENDED!|OFF
strictCheckTypes|Combines `strictMissingProperties` and `strictPrimitiveOperators`|OFF
strictMissingProperties|Warnings for missing properties that forbid accessing subclass props off superclasses|OFF
strictModuleDepCheck|Warnings about all references potentially violating module dependencies|OFF
strictPrimitiveOperators|Warnings about legal operand types for primitive operators like `+` and `-`|OFF
suspiciousCode|Warning about things like missing semicolons and comparisons to NaN|WARNING
tweakValidation|Warnings about goog.tweak primitives|OFF
typeInvalidation|Warn about properties that cannot be disambiguated when using type based optimizations|OFF
undefinedNames|Warnings when a property of a global name is not defined.|OFF
undefinedVars|Warnings when a variable is never defined.|ERROR
unknownDefines|Warnings when unknown @define values are specified.|WARNING
unusedLocalVariables|Warnings about unused variables in local scopes|OFF
unusedPrivateMembers|Warnings about @private class properties that are unused|OFF
untranspilableFeatures|Warnings about use of features that cannot be transpiled down to the specified `--language_out`|ERROR
useOfGoogBase|[DEPRECATED]Warnings about usages of goog.base, which is deleted from Closure library|OFF
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
    
You can also silence some warnings by adding JSDoc annotations to your code. See https://github.com/google/closure-compiler/wiki/@suppress-annotations for more details.
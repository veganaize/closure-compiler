# Introduction

The JS Conformance framework is tool that is part of the [Closure Compiler](https://github.com/google/closure-compiler/) that allows developers to specify configuration for automatically checking
compiled code for conformance to certain requirements, such as forbidding
access to a certain property or calls to a certain function.

Conformance rules are provided to the Closure Compiler as arguments, and they
are enforced during compilation and are reported as warnings (which can be
promoted to errors).  The conformance rules are enforced strictly, which in
this case means that insufficient type information, can result in a “<em>possible violation</em>” warning rather than not being reported at all (this has the interesting
side-effect of revealing lacking type information in unexpected places).

# Running the Closure Compiler with Conformance Enabled

To invoke the Closure Compiler with a conformance configuration, pass the
"--conformance\_configs" flag to the compiler with a list of files that contain
the conformance config.

Ex: <code>java -jar build/compiler.jar --conformance\_configs
my\_conformance\_config.textproto</code>

If invoking the compiler programmatically, the <code>setConformanceConfig</code> or <code>setConformanceConfigs</code> function can be called in the <code>CompilerOptions</code> class.

# Creating a Conformance Requirement

Each requirement has three required fields: “type”, “value”, and
“error\_message”

<strong>type</strong> is used to trigger the specific kind or require as described below.

<strong>value</strong> is a type specific value as described below.  The type and meaning of this
value depends on the <strong>type</strong> parameter. As a convenience “value” may be repeated to create a set of related
rules that shared the same type, error\_message and whitelist.

<strong>error\_message</strong> is the message reported when the requirement is violated.  The error\_message
should include short link a document that describes the reason for the
requirement and a means for discussing the requirement.

Additionally, every requirement may have zero or more <strong>whitelist</strong> and/or <strong>whitelist\_regexp</strong> entries.  The each <strong>whitelist</strong> entry contains a path prefix of file that are allowed to violate the
requirement.  A complete path can be provide to whitelist a single file. The <strong>whitelist\_regexp </strong>entry is an Java RegExp pattern and files that match the pattern are allowed to
violate the requirement.

An example conformance configuration looks like:


```protobuf
requirement: {
  type: BANNED_PROPERTY_WRITE
  value: 'Location.prototype.href'
  error_message: 'Assignment to Location.prototype.href '
                 'is not allowed. Use '
                 'goog.dom.safe.setLocationHref instead.'
  whitelist: 'javascript/closure/dom/safe.js'
}
```
## Requirement Types

The system defines several different built-in requirement types it can enforce,
and allows you to write custom requirements implemented in an external Java
class. 

### BANNED\_CODE\_PATTERN

A forbidden code pattern. Provides a way of banning code patterns.

The <strong>value</strong> for is a [RefasterJS](https://github.com/google/closure-compiler/wiki/RefasterJS) style template function.  The function template is a literal AST match, where
the function parameters are used to perform type matches.  Due to the literal
nature of the pattern comparison, patterns should be as simple as possible to
avoid missing variations.

This <strong>value </strong>would ban throwing the text “error”:


```js
function template() { throw ‘error’ }
```
This <strong>value </strong>would ban throwing a string value or a string object:


```js
/** @param {string|String} str */
function template(str) { throw str }
```
See the [RefasterJS documentation](https://github.com/google/closure-compiler/wiki/RefasterJS) for details.

### BANNED\_NAME

A forbidden fully distinguished name. The <strong>value</strong> for this rule is the fully distinguished name. For example:

  * A global name like "eval" or "goog"
  * A namespaced value:  “namespace.banned”
  * A 'static' property:  "namespace.Foo.banned"

NOTE: This can not be used to ban a JS keyword like “with” or “debugger” or
“this”.

### BANNED\_PROPERTY

A banned instance property.  The <strong>value </strong>for this rule is the fully distinguished type name, the separating to token
“.prototype.” followed by the banned property. For example:

  * An 'instance' property:   "namespace.Foo.prototype.banned"
  * All properties of a given name "Object.prototype.banned"

NOTE: “prototype” is used here simply to separate the type name from the
property. It is not limited to properties declared on the prototype.
Specifically, properties assigned to an instance of a type would still be
checked (such as in the constructor).  Additionally, the property being banned
does not need a known member of the type.

### BANNED\_PROPERTY\_READ

An banned instance property, like BANNED\_PROPERTY but where writes of the
property are allowed. The <strong>value</strong> for this rule follows the same format as BANNED\_PROPERTY. For example:

  * An 'instance' property: "namespace.Foo.prototype.banned"
  * All properties of a given name "Object.prototype.banned"

NOTE: This allows definitions (“Foo.prototype.banned = … “) but not use (“new
Foo().banned()).

### BANNED\_PROPERTY\_WRITE

An banned instance property, like BANNED\_PROPERTY but where reads of the
property are allowed. The <strong>value</strong> for this rule follows the same format as BANNED\_PROPERTY. For example:

  * An 'instance' property: "namespace.Foo.prototype.banned = 1"
  * All properties of a given name "Object.prototype.banned++"

### BANNED\_PROPERTY\_NON\_CONSTANT\_WRITE

A banned write of a non-constant value to an instance property.
Unlike BANNED\_PROPERTY\_WRITE, this only bans assignments of a
non-constant value.

### BANNED\_PROPERTY\_CALL

A banned function call. For example:

  * An 'instance' property: "namespace.Foo.prototype.banned"
  * All properties of a given name: "Object.prototype.banned"

Unlike BANNED\_PROPERTY, this only bans calls to the property, i.e. using the
property as a value is allowed.

### RESTRICTED\_NAME\_CALL

A restricted call to a fully distinguished name. The <strong>value </strong>for this rule provides an alternate type signature for a function to which all
call sites must conform. The format for this <strong>value </strong>is the fully distinguished name, a separating “:”, followed by function type
signature. For example:

  * the "parseInt" call must be called with a radix:
parseInt:function(string, int)

NOTE: This rule does not override a type signature.  The call site will be
validated by both the original function signature during normal type checking
and by the signature provided by the rule.

### RESTRICTED\_METHOD\_CALL

A restricted call to an instance property. As with RESTRICTED\_NAME\_CALL, the <strong>value </strong>for this rule provides an alternative type signature.  The property is declared
as with BANNED\_PROPERTY.

### CUSTOM

A requirement enforced with code an external java class. Before implementing a
custom rule please email js-compiler@ to confirm such a rule is necessary and
is not appropriate as standard rule.

An example of a custom conformance rule:

requirement: {
  type: CUSTOM
  java_class: 'com.google.javascript.jscomp.ConformanceRules$BanExpose'
  error_message: '@expose is not allowed.'
}

#### Predefined Custom Rules

Some rules are difficult to express with the standard rules and difficult to
generalize into a standard rule (or haven’t been yet) but are common enough to
include in the standard framework.

##### BanExpose

Ban’s @expose which can lead to unexpected problems with property collapsing
problems as all properties of the same name are exposed regardless of type.

class: com.google.javascript.jscomp.ConformanceRules$BanExpose

reason for custom rule: There is not yet a standard action to inspect jsdoc
annotations directly

##### BanThrowOfNonErrorTypes

Only Error (or subtype) objects will consistently have stack traces attached.
Ban’s other types (strings are the most common)

class: com.google.javascript.jscomp.ConformanceRules$BanThrowOfNonErrorTypes

reason for custom rule: It is common to catch and rethrow unknown types, the
custom rule excludes rethrows in catch blocks.

##### BanUnresolvedType

Ban’s types that are referenced and known via forward declarations but not
included in the compilation jobs dependencies. For legacy and code size
reasons, this is normally allowed but it  can result in confusing type errors
and missing type information as the type details of the type and its
relationship to other types is unknown.

class: com.google.javascript.jscomp.ConformanceRules$BanUnresolvedType

reason for custom rule: There is no type name or relationship that can be used
to look for these unresolved types.

##### BanGlobalVars

Global variables are code smell: a source of subtle bugs and incompatibilities
and are often enough introduced accidentally. This rule requires that the only
global declarations are goog.provide’d namespaces or equivalent. Note: other
uses can we whitelisted in the standard ways.

class: com.google.javascript.jscomp.ConformanceRules$BanGlobalVars

reason for custom rule: There is no concept of “scope” in the standard rules.

##### BanNullDeref

Dereferencing null or undefined is a prevalent problem and something the
compiler can check for but loose declarations in common code and generated
files can make this impractical for many projects. However, as conformance rule
can be enforced selectively, this rule allow projects to get null safety where
it is practical.

class: com.google.javascript.jscomp.ConformanceRules$BanNullDeref

reason for custom rule: Selective enforcement.

##### BanUnknownThis

“this” values that are typed as “unknown” are a common way to unintentionally
escape type checking.  Banning these allows for a sanity check on type checking
in a relatively low noise and fairly straightforward and unobtrusive way. This
is similar to the warnings seen in critique. Common reason for unknown this
values and possible solutions are discussed here:

[unknownthis](https://github.com/google/closure-compiler/wiki/Conformance:-Unknown-type-of-%E2%80%9Cthis%E2%80%9D)

class: com.google.javascript.jscomp.ConformanceRules$BanUnknownThis

reason for custom rule: There is not a standard way to check for the type of a
particular kind of AST node, and particular allowed uses (casts and asserts),
make this better as a standard rule.

##### BanUnknownTypedClassPropsReferences

Property references that are typed as “unknown” are a common way to
unintentionally escape type checking.  Banning these allows for a sanity check
on type checking.  Common reason for unknown property types and possible
solutions are discussed here:

[unknownprops](https://github.com/google/closure-compiler/wiki/Conformance:-Sources-of-%22unknown%22)

<strong>class</strong>:
com.google.javascript.jscomp.ConformanceRules$BanUnknownTypedClassPropsReferences

<strong>value</strong>: this rules accepts “value” entries that can be used to whitelist types that
are expected to produce references to unknown properties.

<strong>reason for custom rule</strong>: There is not a standard way to check for the type of a particular kind of AST
node, and particular allowed uses (casts and asserts), make this better as a
standard rule.
# Testing Conformance Checks

The [http://closure-compiler-debugger.appspot.com/](http://closure-compiler-debugger.appspot.com/) service contains a spot to write a conformance configuration and test it on
sample code. In addition to verifying that it correctly catches the code that
it should, the page also has fields for Compiler Warnings and source AST to
help debug any problems.

# Example Conformance Checks

Example conformance checks can be found at [https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/example\_conformance\_proto.textproto](https://github.com/google/closure-compiler/blob/master/src/com/google/javascript/jscomp/example_conformance_proto.textproto) - these conformance configurations can be copied into your own project's
configuration.

[Example conformance config](http://closure-compiler-debugger.appspot.com/#input0%3Dgoog.forwardDeclare('bad.namespace')%253B%250A%250A%252F**%2520%2540param%2520%257Bbad.namespace%257D%2520x%2520*%252F%250Afunction%2520f(x)%2520%257B%250A%2520%2520var%2520z%2520%253D%2520x.toString()%253B%250A%257D%250A%250A(function()%2520%257B%2520return%2520this%255B'some'%2520%252B%2520'prop'%255D%253B%2520%257D)()%253B%250A%250A%252F**%2520%2540constructor%2520*%252F%250Afunction%2520Foo()%2520%257B%2520this.myprop%253B%2520%257D%250A%250A%252F**%2520%2540param%2520%257BFoo%257D%2520x%2520*%252F%250Afunction%2520g(x)%2520%257B%2520return%2520x.myprop%2520%252B%25201%253B%2520%257D%26input1%26conformanceConfig%3D%250Arequirement%253A%2520%257B%250A%2520%2520type%253A%2520CUSTOM%250A%2520%2520java_class%253A%2520'com.google.javascript.jscomp.ConformanceRules%2524BanUnknownThis'%250A%2520%2520error_message%253A%2520'References%2520to%2520%2522this%2522%2520that%2520are%2520typed%2520as%2520%2522unknown%2522%2520are%2520not%2520allowed.%2520See%2520http%253A%252F%252Fgo%252Fclosure-js-conformance%2523unknownThis'%250A%257D%250A%250Arequirement%253A%2520%257B%250A%2520%2520type%253A%2520CUSTOM%250A%2520%2520java_class%253A%2520'com.google.javascript.jscomp.ConformanceRules%2524BanUnresolvedType'%250A%2520%2520error_message%253A%2520'Forward-declared%2520types%2520must%2520be%2520included%2520in%2520the%2520compilation.'%250A%257D%250A%250Arequirement%253A%2520%257B%250A%2520%2520type%253A%2520CUSTOM%250A%2520%2520java_class%253A%2520'com.google.javascript.jscomp.ConformanceRules%2524BanUnknownTypedClassPropsReferences'%250A%2520%2520error_message%253A%2520'References%2520to%2520object%2520props%2520that%2520are%2520typed%2520as%2520%2522unknown%2522%2520are%2520discouraged.'%250A%257D%250A%26externs%3Dvar%2520goog%253B%250Agoog.forwardDeclare%253B%26refasterjs-template%26includeDefaultExterns%3D1%26CHECK_SYMBOLS%3D1%26CHECK_TYPES%3D1%26CLOSURE_PASS%3D1%26LANG_IN_IS_ES6%3D1%26MISSING_PROPERTIES%3D1%26PRESERVE_TYPE_ANNOTATIONS%3D1%26PRETTY_PRINT%3D1%26TRANSPILE%3D1) that enables checks related to unknown types.
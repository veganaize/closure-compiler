# Improved property renaming using type information

_As of the 20150315 release, these passes are enabled by default and therefore no longer considered experimental._

# Introduction

The two compiler passes DisambiguateProperties and AmbiguateProperties improve property renaming, dead code removal and other optimization using type information.  

AmbiguateProperties and DisambiguateProperties may be enabled individually through the Java API or together by using the `use_types_for_optimization` flag on the command line or in the web service.

# Details

These passes use type information to allow improved property renaming.

DisambiguateProperties generally renames a property `prop` on class `Foo` to something like `Foo$prop`.  This prevents other compiler passes (that do not understand types) from being confused by the fact that properties on two unrelated classes happen to have the same name.  This can improve the quality of code produced by such passes as inlining.  However, some properties must be renamed the same way for the compiled output to behave as expected.  In particular, subclasses with the same property (e.g., overridden methods) must have the same name.  Also, if a property reference `x.prop` is found, where `x` has the union type `Foo|Bar`, then the `prop` property of `Foo` and `Bar` must have the same name.  DisambiguateProperties deals correctly with both of these cases.

AmbiguateProperties does the opposite of DisambiguateProperties:  it tries to rename unrelated properties to have the same name.  Ideally, every class would end up with properties named a, b, c, ....  However, as above, there are cases where properties must be renamed differently.  In particular, properties on subclasses must use names distinct from those of their superclass so that both values can be stored in the object at run-time, so in those cases, AmbiguateProperties will use different names. 

These two passes are intended to be used together.  However, they should also be usable separately.

Both of these passes depend on correct type information in order to work properly.  For example, if the compiler does not know that one class extends from another, it may rename their properties to the same names, so one value overrides the other.  (In the particular case where @extends does not match the goog.inherits call, the compiler will issue a warning.)

That does not mean that these passes cannot be used unless the type information is 100% accurate.  Both passes are designed to degrade gracefully in the presence of missing or incorrect type information.  Specifically, if a property reference `x.prop` is found anywhere in the code where the type of `x` is unknown, these passes will leave the name `prop` unchanged throughout the code.  Also, if a type warning involving class `Foo` is encountered, these passes may choose to leave all properties of those classes unchanged.

That said, it is impossible for these passes to guarantee correct behavior in the presence of arbitrary type errors.  So if you wish to use these passes on a program with type errors, you will need to test the resulting program carefully.  (Note that missing type information, by itself, should never produce incorrect behavior.)

In addition to incorrect type information, another common source of difficulties is reflection.  For example, some code uses `goog.mixin` to copy properties between two objects (reflecting by looping over the property names).  However, if the type information does not reflect that the two classes involved are related, then the source and target class may receive conflicting names.

In general, reflection can be supported, but the compiler needs to understand what is going on.  Special knowledge of these constructs can be added to the compiler as necessary (`goog.mixin` is not currently supported, as the compiler does not model multiple inheritance).

One final mechanism related to renaming is the method `goog.reflect.object(type, {..})`.  This method (which does nothing at run-time) informs the compiler that the given object literal should have its keys renamed in the same manner as the properties of the given type.  This can be used, for example, to construct a map from unobfuscated property names to obfuscated property names by transposing this object literal.
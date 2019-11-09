# Mixins

[Mixins](https://en.wikipedia.org/wiki/Mixin) have a very low level support in Closure Compiler's type system. It's possible to annotate code so that mixins can be typechecked, but requires some extra boilerplate. Closure Compiler does not understand mixins at all in unannotated code.

While there are a wide variety of different mixin patterns in JavaScript, this page will discuss how to implement mixins using inheritance.

## Inheritance-based mixin example

Say you have a mixin that adds a `hello` method. This may be implemented (omitting any Closure types for now) as.

``` js
const helloMixin = superClass => class extends superClass {
  hello(name) {
    console.log(`Hello, ${name}.`);
  }
};
```

Let's also define a sample usage of this mixin, and intentionally insert some bugs we'd like Closure Compiler to report.

``` js

class GoodbyeSayer {
  goodbye(name) {
    console.log(`Good bye, ${name}.`);
  }
}
const SocialButterfly = helloMixin(GoodbyeSayer);
const butterfly = new SocialButterfly();
// Oops - we forgot that both hello and goodbye expect a name.
butterfly.hello();  // "Hello, undefined."
butterfly.goodbye(); // "Good bye, undefined."
```

Given this code, Closure Compiler has no idea what the type of `SocialButterfly` is. This code won't cause any type errors or missing property warnings, but also won't emit the useful error saying that `hello` and `goodbye` are called with too few arguments. (If you're enabling https://github.com/google/closure-compiler/wiki/JS-Conformance-Framework, this will trigger certain unknown property type errors).

[Debugger link](https://closure-compiler-debugger.appspot.com/#input0%3Dconst%2520helloMixin%2520%253D%2520superClass%2520%253D%253E%2520%250A%2520%2520%2520class%2520extends%2520superClass%2520%257B%250A%2520%2520%2520%2520hello(name)%2520%257B%250A%2520%2520%2520%2520%2520%2520console.log(%2560Hello%252C%2520%2524%257Bname%257D.%2560)%253B%250A%2520%2520%2520%2520%257D%250A%2520%2520%257D%253B%250Aclass%2520GoodbyeSayer%2520%257B%250A%2520%2520goodbye(name)%2520%257B%250A%2520%2520%2520%2520console.log(%2560Good%2520bye%252C%2520%2524%257Bname%257D.%2560)%253B%250A%2520%2520%257D%250A%257D%250Aconst%2520SocialButterfly%2520%253D%2520helloMixin(GoodbyeSayer)%253B%250Aconst%2520butterfly%2520%253D%2520new%2520SocialButterfly()%253B%250Abutterfly.hello('John')%253B%250Abutterfly.goodbye('Jane')%253B%26input1%26conformanceConfig%26externs%3Dvar%2520console%253B%250Aconsole.log%253B%26refasterjs-template%26CHECK_TYPES%3Dtrue%26STRICT_CHECK_TYPES%3Dtrue%26MISSING_PROPERTIES%3Dtrue%26CLOSURE_PASS%3Dtrue%26PRESERVE_TYPE_ANNOTATIONS%3Dtrue%26PRETTY_PRINT%3Dtrue)

## Typing the mixin

### Defining an interface

Getting Closure Compiler to type your mixin requires creating a new interface containing all the properties in the mixin.

This is necessary because Closure Compiler currently does not support multiple inheritance or inferring intersection types. The closest feature in the type system is `@implements`.

In order to get Closure to know what types you are returning from `helloMixin`, you must
 1. define an @interface `HelloMixin` that contains all the properties and methods added by your mixin
 2. annotate the return value of `helloMixin` with `@implements {HelloMixin}`
 3. annotate the return value of `helloMixin(GoodbyeSayer)` with `@implements {Mixin}`
 4. annotate the return value of `helloMixin(GoodbyeSayer)` with `@constructor @extends {GoodbyeSayer}`

Step 2 just ensures that your interface matches your actual mixin implementation. It doesn't actually affect type inference on any calls to the mixin.

Step 3 tells the compiler that the return value contains the properties defined in your mixin.

Step 4 tells the compiler that the return value is a subclass of GoodbyeSayer.

Here's what the end result should look like:

``` js
/** @interface */
class HelloMixin {
  hello(name) {}
}

const helloMixin = superClass => 
   /** @implements {HelloMixin} */
   class extends superClass {
    hello(name) {
      console.log(`Hello, ${name}.`);
    }
  };
class GoodbyeSayer {
  goodbye(name) {
    console.log(`Good bye, ${name}.`);
  }
}
/** 
 * @constructor
 * @implements {HelloMixin}
 * @extends {GoodbyeSayer}
 */
const SocialButterfly = helloMixin(GoodbyeSayer);
const butterfly = new SocialButterfly();
butterfly.hello();
butterfly.goodbye();
```

Now Closure Compiler is able to recognize the types of both `hello` and `goodbye` and emits the following warnings:

```
input0:29: WARNING - [JSC_WRONG_ARGUMENT_COUNT] Function SocialButterfly.prototype.hello: called with 0 argument(s). Function requires at least 1 argument(s) and no more than 1 argument(s).
butterfly.hello();
^^^^^^^^^^^^^^^^^

input0:30: WARNING - [JSC_WRONG_ARGUMENT_COUNT] Function GoodbyeSayer.prototype.goodbye: called with 0 argument(s). Function requires at least 1 argument(s) and no more than 1 argument(s).
butterfly.goodbye();
^^^^^^^^^^^^^^^^^^^
```

[Debugger link](https://closure-compiler-debugger.appspot.com/#input0%3D%252F**%2520%2540interface%2520*%252F%250Aclass%2520HelloMixin%2520%257B%250A%2520%2520hello(name)%2520%257B%257D%250A%257D%250A%250A%252F**%250A%2520*%2520%2540param%2520%257Bfunction(new%253A%2520Object)%253A%2520%253F%257D%2520superClass%250A%2520*%2520%2540return%2520%257Bfunction(new%253A%2520Object)%253A%2520%253F%257D%250A%2520*%252F%250Aconst%2520helloMixin%2520%253D%2520superClass%2520%253D%253E%2520%250A%2520%2520%2520%252F**%2520%2540implements%2520%257BHelloMixin%257D%2520*%252F%250A%2520%2520%2520class%2520extends%2520superClass%2520%257B%250A%2520%2520%2520%2520hello(name)%2520%257B%250A%2520%2520%2520%2520%2520%2520console.log(%2560Hello%252C%2520%2524%257Bname%257D.%2560)%253B%250A%2520%2520%2520%2520%257D%250A%2520%2520%257D%253B%250Aclass%2520GoodbyeSayer%2520%257B%250A%2520%2520goodbye(name)%2520%257B%250A%2520%2520%2520%2520console.log(%2560Good%2520bye%252C%2520%2524%257Bname%257D.%2560)%253B%250A%2520%2520%257D%250A%257D%250A%252F**%2520%250A%2520*%2520%2540constructor%250A%2520*%2520%2540implements%2520%257BHelloMixin%257D%250A%2520*%2520%2540extends%2520%257BGoodbyeSayer%257D%250A%2520*%252F%250Aconst%2520SocialButterfly%2520%253D%2520helloMixin(GoodbyeSayer)%253B%250Aconst%2520butterfly%2520%253D%2520new%2520SocialButterfly()%253B%250Abutterfly.hello()%253B%250Abutterfly.goodbye()%253B%26input1%26conformanceConfig%26externs%3Dvar%2520console%253B%250Aconsole.log%253B%26refasterjs-template%26CHECK_TYPES%3Dtrue%26STRICT_CHECK_TYPES%3Dtrue%26MISSING_PROPERTIES%3Dtrue%26CLOSURE_PASS%3Dtrue%26PRESERVE_TYPE_ANNOTATIONS%3Dtrue%26PRETTY_PRINT%3Dtrue)

### Annotate the mixin parameter and return type

The previous step skipped adding `@param` and `@return` to `helloMixin`. This is because there is no way to sufficiently express the result of `helloMixin` with these annotations alone. However, it's a good idea to annotate the actual mixin to catch a few more bugs. Let's express that:
 - the mixin must be called with a var-args function. (We'd like to restrict this further to constructors, but that's not currently possible in JSDoc.)
 - the mixin is known to return a subclass of the constructor passed as a parameter

For now, let's type both the input and output constructors as taking any number of arguments of any type.

``` js
/**
 * @param {function(new: T, ...?): ?} superClass
 * @return {function(new: T, ...?): ?}
 * @template T
 */
const helloMixin = superClass => 
   /** @implements {HelloMixin} */
   class extends superClass {
    helloThere(name) {
      console.log(`Hello, ${name}.`);
    }
  };
```

Unfortunately there's no way to get Closure Compiler to infer the constructor parameters based on the base class. The only options are
 - allow var args parameters with `...?`, as in the above example
 - hard-code the expected parameters of the constructor (e.g. replace `...?` with `string, number = undefined`)

### Known limitations of Closure Compiler support

- requires an @interface definition separate from the actual mixin
- cannot infer types of constructor parameters automatically
# Explaining the @struct and @dict annotations

@struct is used to create objects that have a fixed number of properties, ie, objects that behave similarly to Java objects.

@dict is used for objects that are dictionaries.

When you annotate a constructor C with @struct, the compiler checks that you only access the properties of C objects using dot notation. It also checks that you do not add new properties to C objects after they're created.

    /** 
     * @constructor
     * @struct
     */
    function Foo(x) {
      this.x = x;
    }
    
    var obj = new Foo(123);
    var n1 = obj['x']; // warning
    var n2 = obj.x; // OK
    obj.y = "asdf"; // warning

When you annotate a constructor C with @dict, the compiler checks that you only access the properties of C objects using brackets.

    /** 
     * @constructor
     * @dict
     */
    function Foo(x) {
      this['x'] = x;
    }
    
    var obj = new Foo(123);
    var n1 = obj.x; // warning
    var n2 = obj['x']; // OK

By using @struct, you know that the compiler will rename all properties safely, because you can't use bracket access. By using @dict, you know that the properties will have the same name after compilation.

You can also use @struct and @dict on object literals directly.

    var s = /** @struct */ { x: 1 }, d = /** @dict */ { y: 321 };
    var n1 = s[‘x’]; // warning
    var n2 = d.y; // warning

This is how the annotations work in a nutshell. Read on to learn about the subtler issues of these annotations.

# More details

## Additional type checking

@struct also tightens missing property checks.  Normally, any property defined on a subclass of a type is allowed off of a known super type.  With an @struct annotated type, this is not allowed.  To access a property defined on a subtype without a warning, the value must first be down cast to the subtype.


## Inheritance maintains the annotations

Consider the following example.

    /** @constructor */
    function Foo(x) {
      this.x = x;
    }
    
    /** 
     * @constructor
     * @struct
     * @extends {Foo}
     */
    function Bar(x, y) {
      this.x = x;
      this.y = y;
    }
    
    /** @param{Foo} obj */
    function getx(obj) { return obj[‘x’]; }
    var z = getx(new Bar(123, 456)); // no warning

In this example, Bar inherits from Foo and is also annotated with @struct. But then, a function that takes Foos can use bracket access for Bar objects, because Bar is a subtype of Foo. For this reason, the compiler does not allow a struct or dict class to extend an unannotated class. Also, when Bar extends Foo and Foo is annotated with @struct or @dict, Bar automatically inherits the annotation.

## Annotating the prototype

Let's say you annotate a constructor function with @struct or @dict, and you also want to modify the prototype property of that function. Since annotated constructors can’t inherit from unannotated ones, when you annotate a constructor with @struct, you must also annotate the prototype. (This is not necessary if you don't change the default prototype property.)

One way to do this is to create a constructor for the prototype and annotate that constructor.

    /** 
     * @constructor
     * @struct
     */
    function FooProto() {
      this.identity = function(x) { return x; };
      this.add1 = function(x) { return x+1; };
    }
    
    /** 
     * @constructor
     * @struct
     */
    function Foo(x) {
      this.x = x;
    }
    
    Foo.prototype = new FooProto();

This is verbose because you create FooProto and only use it once. Alternatively, you can use an annotated object literal.

    /** 
     * @constructor
     * @struct
     */
    function Foo(x) {
      this.x = x;
    }
    
    Foo.prototype = /** @struct */ {
      id: function(x) { return x; },
      add1: function(x) { return x+1; }
    };

## Subtyping and unsoundness

There are a few cases where the compiler cannot enforce the properties of structs and dicts. Fixing those cases would require breaking changes to the type system, so we decided to live with some missed checks. 
First, structs and dicts are subtypes of Object. The compiler cannot enforce the struct/dict properties for the Object type.

    /** 
     * @constructor
     * @dict
     */
    function Foo(x) {
      this[‘x’] = x;
    }
    
    /** @param{Object} obj */
    function fun1(obj) { return obj.toString(); }
    
    fun1(new Foo(123)); // dict accessed with dot but no warning

Second, if a struct or dict implements an interface, it’s a subtype of that interface. The compiler cannot enforce the struct/dict properties for the interface type.

    /** @interface */
    function Foo() {}
    
    /**
     * @constructor
     * @struct
     * @implements {Foo}
     */
    function Bar() { this.x = 123; }
    
    var n = /** @type{Foo} */(new Bar())[‘x’];

By casting to Foo, the compiler doesn't warn about the illegal bracket access.

Last, structs and dicts can be used in places that expect a record type, like any other class. The compiler cannot enforce the struct/dict properties for the record type.

    /** 
     * @constructor
     * @struct
     */
    function Foo() { this.x = 123; }
    
    /** @param{{x: number}} rec */
    function fun1(rec) { return rec['x']; }
    
    fun1(new Foo()); // struct accessed with brackets but no warning
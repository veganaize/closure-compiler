# Type Expressions

You can specify the data type of any variable, property, expression or function parameter with a type expression. A type expression consists of curly braces ("{ }") containing some combination of the type operators described below.

Use a type expression with the `@param` tag to declare the type of a function parameter. Use a type expression with the `@type` tag to declare the type of a variable, property, or expression.

The more types you specify in your code, the more optimizations the compiler can make and the more mistakes it can catch.

The compiler uses these annotations to type-check your program. Note that the Closure Compiler does not make any promises that it will be able to figure out the type of every expression in your program. It makes a best effort by looking at how variables are used, and at the type annotations attached to their declarations. Then, it uses a number of type inference algorithms to figure out the type of as many expressions as possible. Some of these algorithms are straightforward ("if x is a number, and we see y = x;, then y is a number"). Some are more indirect ("if f's first parameter is documented as a callback that must take a number, and we see `f(function(x) { /** ... */ });`, then x must be a number")

### Type Name	

Specifies the name of a type.

```
{boolean}
{Window}
{goog.ui.Menu}	
```

### Type Application

Parameterizes a type with a set of type arguments. Similar to Java generics.

```
// An array of strings.
@type {Array<string>}

// An object in which the keys are strings and the values are numbers.
{Object<string, number>} 
```

> Note: `Array.<string>` was the original syntax. The `.` was considered unnecessary and the requirement was removed. `Array<string>` is now the preferred form. 

### Type Union

Indicates that a value might have type A OR type B. Note the parentheses, which are required.

```
// A number or a boolean. 
{(number|boolean)}
```

### Record Type	

Indicates that the value has the specified members with values of the specified types.

Braces are part of the type syntax. For example, to denote an Array of objects that have a length property, you might write: `Array<{length}>`. In the example below, the outer braces indicate that this is a type expression and the inner braces indicate that this is a record type.

```
// An anonymous type with both a property named myNum 
// that has a value of type number 
// and a property named myObject that has a value of any type.
{{myNum: number, myObject}} 
```

### Nullable type (a union type)

Indicates that a value is type A or null.

> All object types are nullable by default whether or not they are declared with the Nullable operator. An object type is defined as anything except a function, string, number, or boolean. To make an object type non-nullable, use the Non-nullable operator.

```
// A number or null.	
{?number}
```

### Non-nullable type

Indicates that a value is type A and not null.

> Functions and all value types (boolean, number, and string) are non-nullable by default whether or not they are declared with the Non-nullable operator. To make a value or function type nullable, use the Nullable operator.

```
// An Object, but never the null value.	
{!Object}
```

### Function Type	

Specifies a function and the types of the function's parameters and/or return value.

```
// A function that takes two parameters (a string and a boolean), and has an unknown return value.
{function(string, boolean)}

// A function that takes no parameters and returns a number.
{function(): number}

// A function that takes one parameter (a string), 
// and executes in the context of a goog.ui.Menu.	
// Specifies the type of the value of this within the function.
{function(this:goog.ui.Menu, string)}

// A function that takes one parameter (a string), and creates //
// a new instance of goog.ui.Menu when called with the 'new' keyword.	
// Specifies the constructed type of a constructor.
{function(new:goog.ui.Menu, string)}

// A function that takes one parameter (a string), and then 
// a variable number of parameters that must be numbers.
{function(string, ...number): number}
	
// Indicates that a function type takes a variable number of parameters, 
// and specifies a type for the variable parameters.
@param {...number} var_args
```


### Optional parameter in a @param annotation (a union type)

Indicates that the argument described by a @param annotation is optional. A function call can omit an optional argument. An optional parameter cannot precede a non-optional parameter in the parameter list.

```
// An optional parameter of type number.
@param {number=} opt_argument

// A function that takes one optional, nullable string and one optional number as arguments.
{function(?string=, number=)}
```	

If a method call omits an optional parameter, that argument will have a value of undefined. Therefore if the method stores the parameter's value in a class property, the type declaration of that property must include a possible value of undefined, as in the following example:

```
/**
 * Some class, initialized with an optional value.
 * @param {Object=} opt_value Some value (optional).
 * @constructor
 */
function MyClass(opt_value) {
  /**
   * Some value.
   * @type {Object|undefined}
   */
  this.myValue = opt_value;
}
```

### The ALL type
```
// Indicates that the variable can take on any type.
{*}
```

### The UNKNOWN type	

```
// Indicates that the variable can take on any type, and the compiler should not type-check any uses of it.
{?}	
```
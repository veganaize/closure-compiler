# Special Interfaces

The Closure type system include a few special types.

## IObject

```javascript
/**
 * @interface
 * @template KEY, VALUE
 */
function IObject() {}
```


IObject describes the "[]" operator (aka computed property accessor). An object that declares that it implements IObject can restrict the "key" used to lookup a value, and the "value" that is expected to be returned.  This can be used to describe "map-like" objects.  As with all Objects, the key should be a `Symbol`, `number`, `string` or a value that reasonably coerces to `string`.

```javascript
/** @constructor @implements IObject<string, Foo> */
function MapStringToFoo() {}

(new MapStringToFoo())[0]  // type warning here.
(new MapStringToFoo())['bar']  // OK, the expression is of type Foo
```

## IArrayLike

```javascript
/**
 * @interface
 * @extends {IObject<number, VALUE>}
 * @template VALUE
 */
function IArrayLike() {}

/** @type {number} */
IArrayLike.prototype.length;
```

IArrayLike extends IObject, such that the "key" is number and the "value" is the content type of the Array-like structure. IArrayLike is an improvement on the `{length:number}` type that is traditional used to signal array like in that the content type of the object can be supplied.

```javascript
/** @constructor @implements IArrayLike<Foo> */
function ArrayLikeOfFoo() { this.length = 0; }

(new ArrayLikeOfFoo())['bar']  // type warning here.
(new ArrayLikeOfFoo())[0]  // OK, the expression is of type Foo
```

## IThenable<VALUE>

IThenable is used to describe Promise like objects and improves upon the traditional "thenable" type (`{then:!Function}`) by allowing the result type to be known and appropriate handle "Promise" unwrapping.

```javascript
/** @constructor @implements IThenable<Foo> */
function MyPromise() {}

/** @override */
MyPromise.prototype.then = function(opt_onFulfilled, opt_onRejected) {};
```



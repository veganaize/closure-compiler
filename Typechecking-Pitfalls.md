# Common Typechecking Pitfalls

## Invariant Generics

There is a function `f` that expects a `Container<Foo|Bar>` and I'm calling it with a `Container<Foo>` -- why isn't this working?

  * Generics are invariant, so `Container<Foo>` is not a subtype of
    `Container<Foo|Bar>`. You can cast the argument to `Container<Foo|Bar>`
    to get rid of the warning.

### Exceptions to invariant generics

Both `Iterable<T>` (plus anything that implements `Iterable`) and `Object<K, V>` are bivariant. This means that you can assign an `Array<Foo>` to an `Array<Foo|Bar>`, and vice versa. This is not type-safe but was accepted as a compromise when trying to land invariant generics.

On the other hand, `Promise` and all other subtypes of `IThenable` are covariant. This means you can assign a `Promise<Foo>` to a `Promise<Foo|Bar>`, but not vice versa. This /is/ type-safe. A user of a `Promise<Foo|Bar>` can always handle the Promise resolving to `Foo`, but they cannot surprise anyone by somehow making a `Promise<Foo>` resolve to a `Bar`.

## Contravariant Parameter Types

A function `f` expects a callback of type `function(?Foo)` (i.e. a function
that takes a `Foo` or `null`) and I'm passing a `function(!Foo)`. `!Foo` is a subtype
of `Foo`, so why doesn't this work?

   * If `f` expects a callback of type `function(?Foo)` then it may in fact pass
     a null to that callback. The function you're providing as a callback does
     not know how to handle nulls. See [this blog post](http://closuretools.blogspot.com/2012/06/subtyping-functions-without-poking-your.html)
     for more details on this problem. You may want to update your callback so
     that it can handle null, or you may want to change the type of `f`'s
     parameter to `function(!Foo)`.


## Return type of `this`

What is the right way to annotate the return type of a method that ends with
`return this;`? If I just write

```javascript
class Foo {
  /** @return {!Foo} */
  bar() {
    ...
    return this;
  }
}
```
then subclasses of Foo will have a return type of Foo instead of the correct
subclass.

*  Do this instead:

     ```javascript
     class Foo {
       /**
        * @template T
        * @this {T}
        * @return {T}
        */
       bar() {
         ...
         return this;
       }
     }
     ```
# Common Typechecking Pitfalls

## Invariant Generics

There is a function `f` that expects a `Promise<Foo|Bar>` and I'm calling it
by doing `f(Promise.resolve(new Foo()));` -- why isn't this working?

  * Generics are invariant, so `Promise<Foo>` is not a subtype of
    `Promise<Foo|Bar>`. You can cast the argument to `Promise<Foo|Bar>`
    to get rid of the warning.


## Contravariant Parameter Types

A function `f` expects a callback of type `function(Foo)` (i.e. a function
that takes a `Foo` or null) and I'm passing a `function(!Foo)`. `!Foo` is a subtype
of `Foo`, so why doesn't this work?

   * If `f` expects a callback of type `function(Foo)` then it may in fact pass
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
A couple of useful tricks using @template.

## Return the calling subtype from a super class method

    /** @constructor */
    function Foo() {}
    
    /** 
     * @this {THIS}
     * @return {THIS}
     * @template THIS
     */
    Foo.prototype.method = function() {
      return this;
    }
    
    /** @constructor @extends {Foo} */
    function Bar() {}
    Bar.prototype = new Foo();
    
    var x = new Bar().method(); // x is of type "Bar"




---
layout: post
title: "OO javascript: Creating objects"
date: 2014-07-16 07:35
comments: true
categories:
---
For a few posts we’ll talk a bit about programming javascript in such a way as to take advantage of OO principles of reuse, including encapsulation,
inheritance, and polymorphism. Javascript of course has no concept of such things as classes; therefore, we use common javascript principles in order to
take advantage of such things–even for how we define a javascript “object”.
This post in particular will focus on javascript object creation.

<!-- more -->
There really is no way to define a class in javascript. ECMA-262 defines an object as an “unordered collection of properties each of which contains a
primitive value, object, or function.” As such, a javascript object is really nothing more than an array of values (which could be any combination of
primitives, other objects, or functions) that are named. This means that an object in javascript is more closely aligned to what I as a traditional
OO-language developer think of as a hash table or a map–a grouping of name-value pairs.

### Creating objects
Here’s how you’d typically create an object in javascript:

```javascript
var guitar = new Object();
guitar.brand = "Fender";
guitar.model = "Stratocaster";
guitar.year = 1963;
guitar.type = "Electric";

guitar.getBrand = function() {
    alert(this.brand);
};
```
Looks good. The problem with this traditional approach is that if you were to create multiple guitar objects, you’d be duplicating a lot of code. Dang! This is where programming patterns can help.

### The factory pattern
Let's set up a bit of code that acts like a factory--in other
words, it is responsible for creating a guitar whenever we call it.
We'll call it `createGuitar`:

```javascript
function createGuitar(brand, model, year, type) {
    var o = new Object();
    o.brand = brand;
    o.model = model;
    o.year = year;
    o.type = type;

    o.getBrand = function() {
        alert(this.brand);
    };
    return o;
}

```

...And then this factory function could be used to build multiple objects with little code duplication:

```javascript
var guitar1 = createGuitar("Fender", "Stratocaster", 1963, "electric");
var guitar2 = createGuitar("Gibson", "Hummingbird", 1957, "acoustic");
```

The problem with the factory pattern is that we can’t identify what type of an object guitar1 or guitar2 is. Change the names of the variables, and it’s even more potentially confusing. That's
where the constructor pattern helps.

### The constructor pattern
Let's create a function that--like the factory pattern--is responsible
for creating our objects. However, let's take advantage of some built-in
features of javascript itself (specifically the `new` operator and the
`this` keyword) to be able to identify the object type:

```javascript
function Guitar(brand, model, year, type) {
    this.brand = brand;
    this.model = model;
    this.year = year;
    this.type = type;
    this.getBrand = function() {
        alert(this.brand);
    };
}

var guitar1 = new Guitar("Fender", "Stratocaster", 1963, "electric");
var guitar2 = new Guitar("Gibson", "Hummingbird", 1957, "acoustic");
```

Note that this constructor is just a function named Guitar (with an uppercase “G”, as javascript convention dictates). No object being created, the properties and methods are assigned to the “this” object, and no return statement. Calling a constructor with the “new” operator causes four steps:
1. Create a new object.
2. Assign the scope of the constructor to the new object so `this` points to the new object.
3. Execute the code inside the constructor which adds properties to the new object.
4. Return the new object.

Furthermore, each of these objects has a “constructor” property that points back to the “Guitar” constructor/function. This property was intended for use in identifying the object type:

```javascript
guitar1.constructor == Guitar // true
guitar2.constructor == Guitar // true
```

This is an advantage over the Factory pattern. The “instanceof” operator is considered to be a safer way of determining type.

The only difference between javascript “constructors” and other javascript functions is the way they are called (with the `new` operator).
Any function that is called with the `new` operator acts as a constructor. So, what if I call it without the `new` operator? The properties
get added to the `window` object. `this` then points to the “Global” object when a function is called in the global scope. Using `new`
assigns the scope of the function to the new object represented by the variable (`var`).

So, any possible problems here?

Try this:

```javascript
console.log(guitar1.getBrand == guitar2.getBrand);
```

What do you think the result is?

If you said "false", give yourself a brownie.
For every object created with a constructor pattern, a new method/function instance is created for each object instance–they are not the same instance of “Function”. The constructor is really doing the following:

```javascript
function Guitar(brand, model, year, type) {
    this.brand = brand;
    this.model = model;
    this.year = year;
    this.type = type;
    this.getBrand = new Function(alert(this.brand));
    };
}
```

For the function/method `getBrand`, a new Function object
is created every time a new Guitar is created.

We really don’t need two instances of Function that do the same thing. One workaround is to move the function definition outside of the constructor:

```javascript
function Guitar(brand, model, year, type) {
    this.brand = brand;
    this.model = model;
    this.year = year;
    this.type = type;
    this.getBrand = getBrand; // set the getBrand variable to the global
getBrand() function
}

// getBrand defined here is a global function
function getBrand() {
    alert(this.brand);
};
```

This works, but clutters up the global scope. Think if `Guitar` had more methods, like `getModel`, `getYear`, `play`, etc.!

### The prototype pattern

The prototype pattern solves this problem. Every function is created with a `prototype` property, which is an object containing properties and methods that
should be available to instances of a particular reference type.

Let’s rewrite the previous example to use the prototype pattern:

```javascript
function Guitar() {
}

Guitar.prototype.brand = "Fender";
Guitar.prototype.model = "Stratocaster";
Guitar.prototype.year = 2006;
Guitar.prototype.type = "electric";
Guitar.prototype.getBrand = function() {
    alert this.brand;
}

var guitar1 = new Guitar();
guitar1.getBrand();     // "Fender"

var guitar2 = new Guitar();
guitar2.getBrand();     // "Fender"

alert(guitar1.getBrand == guitar2.getBrand);    // true

```

Note that we still call the (now empty) `Guitar` function as a constructor
to create new guitars and we have the properties and methods present, which
here are shared among instances.

`guitar1` has a property that points to the prototype object. So does `guitar2`.
(This is known internally as `__proto__`, which can be examined via the `isPrototypeOf()` method).
The `Guitar` constructor function has a member called `prototype`, which also points to the
prototype object. So,

* `guitar1.__proto__` points to `Guitar`’s prototype object
* `guitar2.__proto__` points to `Guitar`’s prototype object
* `Guitar.prototype` points to `Guitar`’s prototype object

This protoype object has a `constructor` property. It also has other
properties and methods as defined by the user. This constructor property points back to the `constructor` function (`Guitar`).


Here's alternate prototype syntax:

```javascript
function Guitar() {
}

Guitar.prototype = {
    constructor : Guitar,
    brand : "Fender",
    model : "Stratocaster",
    year : 2006,
    type : "electric",
    getBrand = function() {
        alert this.brand;
        }
};

var guitar1 = new Guitar();
guitar1.getBrand();     // "Fender"

var guitar2 = new Guitar();
guitar2.getBrand();     // "Fender"

alert(guitar1.getBrand == guitar2.getBrand);    // true
```

It is important to note that it is not possible to overwrite prototype’s values. In other words, if you add a property to an instance that is the same name as a property on the object’s prototype, you create the property on the instance, which shadows the property on the prototype. For example:

```javascript
guitar2.brand = "PRS";
guitar2.getBrand();  // PRS
```

To find a property, the javascript compiler performs a two-Step Search order: First it checks the instance for the property/method. If it’s not found there, it then checks the prototype.

There are some problems with this approach. You can’t pass initialization arguments into the constructor–all instances of the object get the same property values by default.

This kinda blows because typically we don’t want all of our object’s properties to have the same values–we want a guitar that’s a Fender brand, we want one that’s a PRS, etc. And yet we still want to take advantage of having methods shared among the various objects. What we need is a combination of approaches.

### The Constructor/Prototype combination pattern
In the Constructor/Prototype Combination pattern, the constructor defines instance properties, and the prototype defines methods and shared properties. Best of both worlds, right? To do it,
we take a clue from the alternate prototype syntax described above, and explicitly set the constructor
property on the prototype object to point to our constructor function. However, the only other property we define is the method we want shared. Every other property
goes in the constructor function:

```javascript
function Guitar(brand, model, year, type) {
  this.brand = brand;
  this.model = model;
  this.year = year;
  this.type = type;
}

Guitar.prototype = {
    constructor : Guitar,
    getBrand : function() {
        alert(this.brand);
  }
};

var guitar1 = new Guitar("Fender", "Stratocaster", 1963, "electric");
var guitar2 = new Guitar("Gibson", "Hummingbird", 1957, "acoustic");

guitar1.getBrand();
guitar2.getBrand();
alert(guitar1.getBrand === guitar2.getBrand);
```

Now we have our cake and eat it, too: We can create guitars passing
in different initialization parameters for each one, and yet all guitars
share the same `getBrand` method. Great, huh?

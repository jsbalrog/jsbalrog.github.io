---
layout: post
title: "Decorator pattern in JavaScript"
date: 2014-08-15 12:29
comments: true
categories: [programming, javascript]
---
Last time I mentioned I was going to focus on some OO-related javascript concerns.
For this post, I want to talk about a programming pattern that I was quite often:
the Decorator pattern.

<!-- more -->
The Decorator pattern is defined as "a design pattern that allows behavior to be
added to an individual object, either statically or dynammically, without affecting
the behavior of other objects from the same class." [(source)](https://http://en.wikipedia.org/wiki/Decorator_pattern)

What this means in plain english is that we can extend an object's functionality--
even at runtime--and even with multiple decorators. This is pretty common in the
'classical' OO world of, say, java. Here's how we'd do it prototypically in
javascript.

Let's say that we've got a Guitar store shopping cart written in javascript. At
checkout time the user can select various options or upgrades to add to their
guitar. We will implement these options as decorators.

First, the guitar model. It will manage a list of decorators. Here's the constructor:

```javascript
function Guitar(price) {
	this.price = price || 699;
	this.decoratorsList = [];
}
```

Next, the decorators themselves. Decorators will be implemented as properties of
`Guitar.decorators`.

```javascript
Guitar.decorators = {};
```

Now, we add the various decorator properties. These are the options/upgrades the
user can select.

```javascript
// Ebony fretboard option: $100
Guitar.decorators.ebonyFretboard = {
  getPrice: function(price) {
    return price + 100;
  }
};

// Mother of pearl inlay option: $75
Guitar.decorators.motherOfPearlInlays = {
  getPrice: function(price) {
    return price + 75;
  }
};

// Gold hardware option: $50
Guitar.decorators.goldHardware = {
  getPrice: function(price) {
    return price + 50;
  }
};
```

Here's the Guitar's decorate method. This is the method called at runtime, and
it simply pushes each decorator onto the guitar's `decoratorList`.

```javascript
Guitar.prototype.decorate = function(decorator) {
  this.decoratorsList.push(decorator);
  return this;
};
```

Notice that we return the guitar instance. This is so we can chain decorator calls.

Finally, our `getPrice()` method for our shopping cart checkout. It simply goes
through a list of decorators and calls each decorator's `getPrice()` method, passing
the results from the previous call.

```javascript
Guitar.prototype.getPrice = function() {
  var price = this.price,
    i,
    max = this.decoratorsList.length,
    name;

  for (i = 0; i < max; i++) {
    name = this.decoratorsList[i];
    price = Guitar.decorators[name].getPrice(price);
  }
  return price;
};
```

That's all there is to it. Here's the client code in action. We'll create a couple
of guitars and print out the price.

```javascript
var fender = new Guitar(499);
fender.decorate('goldHardware');
console.log(fender.getPrice()); // 549

var takamine = new Guitar();
takamine.decorate('motherOfPearlInlays').decorate('ebonyFretboard');
console.log(takamine.getPrice()); // 874
```

This ought to whet your appetite for finding other uses for the decorator pattern
in your code.

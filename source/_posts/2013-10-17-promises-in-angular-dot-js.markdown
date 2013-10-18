---
layout: post
title: "Promises in Angular.js"
date: 2013-10-17 21:43
comments: true
categories: 
---
One of the hardest things for me to grok in JavaScript was promise-based style
handling of the async nature of the language.
I understood the async JavaScript just fine, and could wield callbacks
like a banshee. However, promise-based structures provided a more elegant solution
than callbacks--particularly when you were several calls deep. I will describe
what I learned in the context of Angular, as it has a very simple yet elegant
implementation of promises in its $q library.

<!-- more -->

First, imagine we have a function for a person. We will define two callbacks for
this person: one for what success for them would be, and one for what their
life's error condition would look like. We'll assume they are either happy or
sad:

```javascript
var Person = function(name) {
  this.success = function(reason) {
    console.log(name + " is happy because " + reason);    
  };

  this.error = function(reason) {
    console.log(name + " is sad because " + reason);
  };
};
```

Next, we'll create this person:

```javascript
var fred = new Person("Fred");
```

We now need to define a task that needs to be accomplished in the future. We'll
say it's a delivery of shiny new roller stakes from Amazon. Receiving them
would be a success, and thus make Fred happy. Having them not delivered would
be an error, and thus make Fred really sad.

To do so, we'll create a task using $q's defer() method. This creates a
deferred object, which holds the future task execution:

```javascript
var task = $q.defer();
```

This future task object has a promise object as a property; it is the placeholder
for the results of the task's execution:

```javascript
var results = task.promise;
```
We then set up the order of events, using "promise, then" syntax. This takes the
place of passing callback(s) to the task method, and having the task method have to call
them when complete.

```javascript
results.then(fred.success, fred.error);
```
We can then have the task programmatically succeed or fail depending on whatever
arbitrary logic we choose. $q's parlance is "resolve" or "reject".

If we succeed:

```javascript
task.resolve("Roller skates from Amazon have arrived");
```
...the appropriate "happy" success method is called. Conversely, if we fail:

```javascript
task.reject("Roller skates from Amazon are backordered");
```
...the appropriate "sad" error method is called.

This is a pretty simple example, one that maybe unneccessarily takes on a lot of
overhead for promises. But you can see how if we now introduce other objects--say
an Amazon online store that has takeOrder, deliverOrder, and problemWithOrder
methods--using promises could avoid multiple levels of callback heck. The takeOrder
method could take an argument of ordered items, and set up a current order that
had an items property as well as a task deferred object. Instead of returning the
order, it simply returns the promise property of the task deferred object, which
is used to call then() on:

```javascript
var amazon = new MailOrder($q);
var itemDelivered = amazon.takeOrder('Nexus 7');
itemDelivered.then(fred.success, fred.error);
amazon.problemWithOrder('Nexus 7 is no longer in stock!');
```

---
layout: post
title: "Talkin' 'bout angular directives"
date: 2014-10-03 15:37
comments: true
categories: [programming, angularjs]
---
In this post, let’s take a look at the power of directives, and in particular,
the power of isolate scope-based directives.

<!-- more -->

### In the beginning
Let’s start with this use case: I’ve got a list of friends and I want to list out
their name and their hobby. The html looks like this:

``` html
<div ng-controller=“MyCtrl as main”>
  <ul ng-repeat=“friend in main.friends”>
    <li>My name is {{ friend.name }}, I play the {{ friend.hobby }}.</li>
  </ul>
</div>
```

...And here's the controller:

``` js
angular.module(‘myApp’, [])
  .controller(‘MyCtrl’, function() {
     this.friends = [
          { ‘name’: ‘Fred’, ‘hobby’: ‘kazoo’ },
          { ‘name’: ‘Eliot’, ‘hobby’: ‘harp’ }
     ];
});
```

Pretty straightforward. Well, let's add to the use case: I want to be able to
“cool-i-fy” any friend in the list by clicking them—no matter who I click,
I want their name to be upgraded to “B.B. King” and their hobby upgraded to
“guitar”. Instant coolness!

So, I do something like this:

``` html
...
<li ng-click=“main.coolifyMe(friend)”>My name is {{ friend.name }}, I play the {{ friend.hobby }}.</li>
...
```

...and the corresponding method in the controller:

``` js
this.coolifyMe = function(friend) {
     friend.name = “B.B. King”;
     friend.hobby = “guitar”;
};
```

Still straightforward, and works well.

### Enter: A custom directive
But wait, I want this to be available throughout my app,
anywhere I’ve got a list of friends. And I show a list of friends 24 different places.
Sounds like a job for a reusable custom directive!

So, I’m going to create a directive, coolnessDir, that can be reused anywhere.
Here’s what the html looks like:

``` html
<div ng-controller=“MyCtrl as main”>
  <ul ng-repeat=“friend in main.friends”>
    <coolness-dir></coolness-dir>
  </ul>
</div>
```

Nice, neat, compact html, which is what angular directives are awesome for. Here’s
our directive:

``` js
.directive(‘coolnessDir’, function() {
  return {
    restrict: ‘E’,
    replace: true,
    template: ‘<li ng-click=“coolifyMe(friend)”>My name is {{ friend.name }}, I play the {{ friend.hobby }}.</li>’,
    link: function(scope, elem, attrs) {
      scope.coolifyMe = function(friend) {
        friend.name = “B.B. King”;
        friend.hobby = “guitar”;
      }
    }

  }
});
```

I rip the `coolifyMe` function out of the controller altogether. Notice that the `coolifyMe`
method in the directive’s `link` function (I have a `link` function because we’re manipulating
  the DOM) looks nearly identical to our old method.

Okay, everything’s good. Works like a champ.

### A problem arises
But wait. There is a list where I want to use this directive, but it looks like this:

``` html
...
<ul ng-repeat=“foo in main.friends”>
...
</ul>
...
```

What’s this `foo` business? My directive code template refers to `friend.name` and
`friend.hobby`—-this will break!

### Isolate scope to the rescue!
Isolate scope comes to the rescue here—it allows us to seal off the directive from
 the enclosing scope(s), and “poke holes” selectively to allow certain values through,
 while also allowing us to redefine those values. We’re going to make sure that whatever
 value gets through to our directive’s template, it’s called `friend`!

First, here’s what our html looks like:

``` html
...
<ul ng-repeat=“foo in main.friends”>
     <coolness-dir friend=“foo”></coolness-dir>
</ul>
...
```

Notice that we added an attribute to our directive: `friend`. This is the hole
through to the directive. Speaking of which, here’s the code now:

``` js
.directive(‘coolnessDir’, function() {
  return {
    restrict: ‘E’,
    replace: true,
    scope: {
      friend: ‘=‘
    },
    template: ‘<li ng-click=“coolifyMe(friend)”>My name is {{ friend.name }}, I play the {{ friend.hobby }}.</li>’,
    link: function(scope, elem, attrs) {
      scope.coolifyMe = function(friend) {
        friend.name = “B.B. King”;
        friend.hobby = “guitar”;
      }
    }
  }
});
```

Notice how we have a scope property now, and we define a key `friend`, which matches
our attribute name that we added. The equals sign says “we want to two-way data
bind this, and furthermore, do it as the same name: `friend`".

Now, we can easily use this directive wherever—-all we need to do is change the
attribute `friend=` to be whatever `ng-repeat` sets the value to!
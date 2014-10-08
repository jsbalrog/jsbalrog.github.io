---
layout: post
title: "Angularjs providers"
date: 2014-10-07 17:04
comments: true
categories: [programming, angularjs]
---
When it comes to objects--and especially singleton objects or factory instances--
angular offerings are multiple-choice. I want to spell out the differences between
angular-specific providers, services, and factories, and give some guidelines to
when you would use each.

<!-- more -->

### Providers
At root, all three types of objects basically are providers. `angular.service` and 
`angular.factory` are syntactic sugar on top of `angular.provider`. That said, you
should really prefer using a `service` or a `factory` (more on these below); use
a `provider` only when you want to expose an API that you need to access in your
angular module's `config` block, before the app starts. For example, you may
have a reuseable service for which you want to modify the behavior between
applications.

Let's look at a concrete example. Let's say I have an app that has the requirement
of specifying a different language depending on the app instance that is
running--the app running on the servers in Japan, for example, must specify
japanese as the app's language. The default language, if no config is
specified, is english. I know, it's pretty contrived--but hey, I don't write
the requirements, I just implement them.

Here's what our provider looks like:

``` js
angular.module('myApp', []).provider('LanguageProvider', function() {
	this.lang = 'english';

  this.$get = function() {
    var lang = this.lang;
    return {
      getLang: function() {
        return lang;
      }
    };
  };

  this.setLang = function(lang) {
    this.lang = lang;
  };
})
```

The key here is the $get function--this returns an object that exposes the
API for the app to use at runtime--in this case `getLang` is available to
call within an arbitrary controller, etc.

Moreover--and here's the important thing--the `setLang` method is callable 
in the app's config block.

### Services

### Factories

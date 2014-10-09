---
layout: post
title: "Angularjs providers, factories, and services"
date: 2014-10-07 17:04
comments: true
categories: [programming, angularjs]
---
When it comes to objects--and especially singleton objects or factory instances--angular 
offerings are multiple-choice. I want to spell out the differences between
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
angular.module('myApp', []).provider('languageSetter', function() {
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

So, our app's config code can look like this:

``` js
angular.module('myApp').config(function(languageSetterProvider){
	// Configuring a provider
	languageSetterProvider.setLang('japanese');
});
```

Notice that we have to append "Provider" onto the end of the name of our provider
to be injected into our `config` function. According to the angular docs, "This 
injection is done by a provider injector which is different from the regular instance 
injector, in that it instantiates and wires (injects) all provider instances only."

During bootstrap of our application, Angular configures and instantiates all providers 
during the configuration phase before it moves on to create our services (in the run phase). 
Therefore, services aren't yet available. This means that any pre-run configuration
(like our language configuration here) is a prime candidate to be thrown into a provider.

Here's a controller, for sake of completion:

``` js
angular.module('myApp').controller('MyCtrl', function($scope, languageSetter) {
    
  $scope.languages = [
    languageSetter.getLang()
  ];
});
```

...and then we can show off our language in an admittedly contrived manner:

{% raw %}
``` html
...
<div ng-app="myApp">
  <div ng-controller="MyCtrl">
    {{ languages }}
  </div>
</div>
...
```
{% endraw %}

Even though our default language is "english", during the app config we set the
language to "japanese" using our provider.

### Factories
An Angular factory constructs a new service...Wait--what? Yes, even though Angular has
a separate Service recipe, the Angular factory recipe is for providing a singleton
service. The nice thing about a factory is that it's very easy to use (think JavaScript
revealing module pattern), it can have other services dependency injected, and can
be lazily initialized.

To continue our example, let's say that a marketing scheme is to be computed based on
the language that is now stored in the browser's local storage for our app. Here's what
the factory service looks like:

``` js
angular.module('myApp').factory('languageFactory', function() {
	var schemify = function(lang) {
		return lang + " marketing scheme--activate!";
	};
	
	var getScheme = function() {
		var lang = window.localStorage.getItem('language');
		return schemify(lang);
	};
	
  return {
    getScheme: getScheme
  };
});
```

...and then we can inject our factory (service) in our controller, and it will be
lazily instantiated as a singleton:

``` js
angular.module('myApp').controller('MyCtrl', function($scope, languageFactory) {    
  $scope.marketingScheme = languageFactory.getScheme();
});
```

Notice that we do not have access to our Factory in the config block of our app; it is
not yet instantiated at that point.

### Services
So, we know so far that a Provider is a service, and a Factory is a service. Not to
mess with you, but a Service is a service as well. Here's what the Angular documentation
says on the topic: "Yes, we have called one of our service recipes 'Service'. We regret 
this and know that we'll be somehow punished for our misdeed. It's like we named one of 
our offspring 'Child'. Boy, that would mess with the teachers."

So now you know. Hopefully we can avoid some confusion. The Service recipe provides a
service just like Factory does, but the difference is that it invokes a constructor with
the JavaScript `new` operator. The arguments passed to the constructor represent 
dependencies needed by instance of this type.

To extend our example once more, let's see what our factory service would look like re-
written as a service, ah, service:

``` js
angular.module('myApp', []).service('languageService', function() {
  this.schemify = function(lang) {
    return lang + " marketing scheme--activate!";
  };
  
  this.getScheme = function() {
    var lang = 'portuguese';
		return this.schemify(lang);
  };
});
```

Looks pretty familiar, doesn't it? We're not doing the public/private thing with methods
here, just defining them on the `this` scope, like we would in a plain ol' JavaScript
constructor. Our controller really doesn't change much.

So, why the difference? Well, in practice, let me say that I use the Factory recipe far
more than the Service recipe. I like being able to specify what methods are exposed as
callable in the return block. Must be some remnant of my old Java public/private days.
I really don't find much use for the Service recipe--can't see what it buys me. I have
used the Provider recipe a couple of times when I do need to do some configuration in 
my app, before it hits the run phase. But again, Factory is far and away what I use most.

Hope this helps clear up some of the confusion between these three services. Of course,
there are also `Value` and `Constant`, but those are topics for another day...
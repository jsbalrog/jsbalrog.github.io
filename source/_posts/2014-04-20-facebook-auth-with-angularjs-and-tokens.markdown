---
layout: post
title: "Facebook auth with Angularjs and tokens"
date: 2014-04-20 19:28
comments: true
categories: 
---
(For full example code, see my <a href="https://github.com/jsbalrog/angular-fullstack-tokens">angular-fullstack-tokens repo</a>.)
<p>
I have an angular-fullstack based app (angular, node, express, passport),
and I'm using token-based authentication following <a href="https://auth0.com/blog/2014/01/07/angularjs-authentication-with-cookies-vs-token/">this great blogpost</a>, 
rather than cookie-based, using express-jwt and jsonwebtoken. 
<p>
I started with daftmonk's <a href="https://github.com/DaftMonk/generator-angular-fullstack">angular-fullstack</a>,
replaced its authentication system with tokens (easy enough), then set
out to include social-based authentication for facebook, using passport-
facebook. I kept running into CSRF-prevention issues, with the error 
`No 'Access-Control-Allow-Origin' header is present on the requested 
resource.` 
<p>
Without going too far off topic, this was due to the fact that
I was attempting to make an `$http` call from the client to a node route
that I had set up for facebook authentication, rather than simply using
an anchor tag href call on the client. Ajax calls are particular tricky
across domain origins. I had to set it up this way because I wanted my
server to return user info of my making back to the client.
<p>
I tried solutions outlined <a href="http://matthewtyler.io/handling-oauth2-with-node-js-and-angular-js-passport-to-the-rescue/">here</a> 
and <a href="http://scotch.io/tutorials/javascript/easy-node-authentication-facebook">here</a>, 
each to no avail. Granted, both were under the "cookie-based auth" umbrella, 
but that shouldn't matter, right?
<p>
After searching for nearly two days, I came up with the idea of making
auth request client-side, rather than server-side. Using the <a href="https://github.com/ninjatronic/ngFacebook">ngFacebook</a>
angular module along with <a href="https://github.com/werk85/grunt-ng-constant">grunt-ng-constant</a> gave me the answer I needed.
Here's how to do it.

<!-- more -->
<h3>Client-side auth with ngFacebook</h3>
First, install ngFacebook via Bower:

```
bower install ngFacebook -save
```

Then, in your angular app.js (or wherever you do your module's config),
declare 'facebook' as a dependency, and initialize using your Facebook's
app id, thus:

```javascript
angular
    .module('my-angularjs-app', ['facebook'])
    .config(['$facebookProvider', function($facebookProvider) {
        $facebookProvider.init({
            appId: 'myFbApplicationId'
        });
    }]);
```

(But what about multiple environments? Those FB apps are picky about site URLs.
More on this in the next section...)
<p>
Now, wherever you plan on having the user do the facebook auth, add the proper
button:

```html
<button type="button" class="btn btn-primary btn-large" data-ng-click="loginFacebook()">Login with Facebook</button>
```
Next, in your controller, set up calls to do your facebook logins much the
same way you would typically make calls using Facebook's javascript SDK:

```javascript
$scope.loginFacebook = function() {
  $facebook.login({scope:'email'}).then(function(response) {
    $scope.me();
  },
  function(response) {
    console.log("Error!", response);
  });
};
$scope.me = function() {
  $facebook.api('/me', {fields: 'id, name, email, username'}).then(function(response) {
    Auth.loginFacebook(response).then(function() {
      $location.path('/');
    });
  });
};
```

Notice that it's all promised-based goodness. For my purposes, in the result
of my call to '/me' I'm in turn making an `$http` call to one of my server routes
that I have set up for checking for user info/creating a new user in my own
data store. On a successful return there, I direct the user to the homepage.
<p>
For sake of completeness, here's my Auth service method:

```javascript
loginFacebook: function(user, callback) {
      var deferred = $q.defer(),
          cb = callback || angular.noop;
      console.log("here", user);
      $http.post('/login/facebook', user)
        .success(function(data) {
          $window.sessionStorage.token = data.token;
          // Do something with the user here
          deferred.resolve(profile);
        })
        .error(function(data) {
          delete $window.sessionStorage.token;
          // Do something else here
          deferred.reject($rootScope.currentUser);
        });
      return deferred.promise;
    }
```

So, back to that question of different facebook app id's to account for
different environments. grunt-ng-constant to the rescue...

<h3>Using grunt-ng-constant</h3>
First, let's install grunt-ng-constant:

```
npm install grunt-ng-constant --save-dev
```
Next, in your `Gruntfile.js`, enter the following:

```javascript
// constants
ngconstant: {
  // Options for all targets
  options: {
    name: 'config',
  },
  // Environment targets
  development: {
    options: {
      dest: '<%= yeoman.app %>/scripts/config.js'
    },
    constants: {
      ENV: {
        name: 'facebook',
        appId: 'YOUR_APP_ID'
      }
    }
  },
  production: {
    options: {
      dest: '<%= yeoman.dist %>/scripts/config.js'
    },
    constants: {
      ENV: {
        name: 'facebook',
        appId: 'YOUR_PROD_APP_ID'
      }
    }
  }
},
```

This tells grunt to automatically create a file called config.js under
the `scripts` folder.
<p>
How do we tell grunt when to create this file? I put mine in the section
that tells it to run a local server for development. I added it to the run
task of the `grunt serve` task:

```javascript
grunt.registerTask('serve', function (target) {
  if (target === 'dist') {
    return grunt.task.run(['build', 'connect:dist:keepalive']);
  }

  grunt.task.run([
    'clean:server',
    'ngconstant:development',
    'bower-install',
    'concurrent:server',
    'autoprefixer',
    'connect:livereload',
    'watch'
  ]);
});
```
And then, for grunt production, I added mine to the `grunt build` task:

```javascript
grunt.registerTask('build', [
  'clean:dist',
  'ngconstant:production',
  'bower-install'
]);
```

When grunt runs, a config.js file is created:

```javascript
angular.module('config', [])

.constant('ENV', {
  'name': 'facebook',
  'appId': 'your-development-appId'
});
```
We need to add this new file to our index.html:

```html
<script src="/scripts/config.js"></script>
```

Now we can inject it into our angular app:

```javascript
angular.module('myApp', ['ngCookies', 'ngResource', 'ngSanitize', 'ngRoute', 'facebook', 'config']).config(function ($routeProvider, $locationProvider, $httpProvider, $facebookProvider, ENV) {
```

Finally, in the config of our app (the same place where we init our facebook module),
we use the ENV constant:

```javascript
$facebookProvider.init({appId: ENV.appId});
```
And that's it! Now, when you click the "Login with Facebook" button, the facebook login
popup will show that is the right facebook app for your environment.
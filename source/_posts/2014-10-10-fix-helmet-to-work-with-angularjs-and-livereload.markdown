---
layout: post
title: "Fix Helmet to work with Angularjs and LiveReload"
date: 2014-10-10 14:53
comments: true
categories: [programming, angularjs]
---
Helmet is a nifty node module--it provides some protection from several types of application attacks. Here’s a great
[post](http://scottksmith.com/blog/2014/09/21/protect-your-node-apps-noggin-with-helmet/) that explains why you should
be using helmet in your node apps.

However, I was getting a couple of errors in my angular-based node app, and on top of that, my livereload stopped working,
which I really depend on in my dev workflow. One of the errors had to do with angular itself, the other had to do with the
injection of the livereload script into my index.html. Here are the two things I did to fix these errors.

<!-- more -->

### The Errors

First, the errors that were showing up in my browser console whenever I loaded the app:
`Refused to evaluate a string as JavaScript because 'unsafe-eval' is not an allowed source of script in the following
Content Security Policy directive: "script-src 'self' 'unsafe-inline’ *.blah.com”`

and

`Refused to load the script 'http://localhost:35729/livereload.js?snipver=1' because it violates the following Content
Security Policy directive: "script-src 'self' 'unsafe-inline' *.blah.com”`

The first error was angular-related, the second livereload-related. I specify `‘self’` and a domain for `defaultSrc` and
`scriptSrc` in my helmet middleware. See the blog post referenced above for more on that.

### The Solutions

First, to solve the angular-related issue, I add the `ng-csp` to my `ng-app` tag in my index.html so it looks like this:
``` html
<html ng-app="app" ng-csp>
```

Second, to solve the liveReload issue, I move my `app.use(helmet.csp{...})` middleware call into my `if(production === env)`
block, which is where it should be. I shouldn’t have to worry too much about CSP during dev on my box, which is the only
environment in which connect-livereload inserts the script tag into my index.html anyways.

Performing these two things got rid of the annoying console errors, and bam--livereload began to work again! Hopefully
this helps someone else.
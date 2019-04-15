---
layout: post
title:  "Brocolli, ES6, Browserify and NPM"
date:   2015-03-23
categories: javascript
---
This post is more of a brain dump than a tutorial. I had some trouble getting this one working, and I couldn't find much help out there using this particular set of tools together in this way.

## What are we doing?
I want to be able to use ES6 syntax, and ES6 modules on the browser, I want to be able to load scripts directly from my node_modules folder, and I want the build to be as fast as possible.

In short, I want ember-cli even when I'm not writing Ember apps!

To do this we use the tools titled above (along with Babel as our ES6 transpiler)

## Setting up
You need the following tools, each can be installed through NPM.

{% highlight javascript %}
> npm install -g broccoli-cli 
> npm install --save-dev broccoli
> npm install --save-dev broccoli-merge-trees
> npm install --save-dev broccoli-static-compiler
> npm install --save-dev broccoli-babel-transpiler
> npm install --save-dev broccoli-browserify
> npm install --save-dev browserify
{% endhighlight %}

Create a Brocfile.js at the root of your project, we'll put all of our JavaScript code directly into an App folder, for simplicity.

The Brocfile.js looks like this

{% highlight javascript %}
var browserify = require('broccoli-browserify');
var copy = require('broccoli-static-compiler');
var merge = require('broccoli-merge-trees');
var babel = require('broccoli-babel-transpiler');

var app = 'app';

var html = copy(app, {
  srcDir: '/',
  files: ['index.html'],
  destDir: '/'
});

var js = babel(app, {});

js = browserify(js, {
  entries: ['./main.js'],
  outputFile: 'application.js'
});

module.exports = merge([html, js]);
{% endhighlight %}

To talk through this very quickly (maybe I'll write a full post on broccoli at some point)

We're copying the index.html file back to the root folder, this means if we specify a destination during a build, the index.html file will be copied over too.

We're grabbing all of our javascript code from the app folder, and using Babel to transpile it to ES5. By default all of the modules will be converted to CommonJS, which works with Browserify.

We're then using browserify to package up all of our code into a single app.js file. This means it will go through and get all of the code that we are importing too, meaning we can grab vendor scripts from our node_modules folder no problem.

Then we're merging our branches together and exporting them.

If we create a couple of JS files in the App folder like this:

{% highlight javascript %}
//app/main.js
import foo from './foo';

console.log(foo());

//app/foo.js
export default function () {
  console.log('spiffy!');
}
{% endhighlight %}

Then run broccoli serve from the command line and navigate to http://localhost:4200 in our browser you should be everything working as intended. At this point broccoli will already be watching for file changes and rebuilding our project (very quickly). It also supports live reloading out of the box, which is nice.

When it comes to build time you can type broccoli build dist into the command line and everything will be built into a dist folder for us. We can copy that folder to the server. It's also extremely fast thanks to broccoli.

Hopefully this post will help out anyone that is stuck, or someone smarter than me can come along and correct any mistakes I may have made!

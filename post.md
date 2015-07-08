## 1 Why use ES6 Harmony
ES6 is short for ECMAScript6, which is the blue print for upcoming version of JavaScript. As a language, JavaScript is a double edged sword. It can be freakingly ugly, but at the same time extremely flexible and powerful. JS code is often criticized for having un-necessary redundancy. Thus abstractions like `CoffeeScript` and `TypeScript` became very popular. I personally love writing in CoffeeScript, but CoffeeScript threatens to fragment the JavaScript community, its code bases and package eco-system. So the majority of JavaScript influencers frown upon abstractions and instead opt to patiently help make JavaScript get better. 

ES6 (code named Harmony) introduces some new JavaScipt language features similar to CoffeeScript. Cleaner iterators (loop syntax), arrow functions, de-structuring assignment and even classes to name a few.

## 2 ES6 features that excite me

I was immediately jazzed to regain the following similar CoffeeScript features within the first hour of having Harmony running:

### 2.1 Cleaner iterator syntax

Thank god, no more index and length based looping in my code.

<!--?prettify lang=javascript linenums=false?-->

    // es6 set style iterator syntax
    for (var post of posts) {
    	console.log(post.url);
    }

### 2.2 Destructuring assignment

Destructuring assigment allows you to pull out attributes of an object and assign them to scope level variables in one line. It's especially handy for pulling out separate functions & values from an imported object.

<!--?prettify lang=javascript linenums=false?-->    
    
    var post = {
    	url: '/first-post',
    	title: 'My First Post'
    }

    // es6 de-structuring assignment
    // Similar to CoffeeScript, but important to use the var
    var {url,title} = post;
    
    console.log('url', url, 'post', post);

### 2.3 Arrow functions
Arrow function in es6 are not as Cool CoffeeScript, but it's better than nothing! There is only a fat arrow function definition (`=>`) and it does not do magic with the function context (`this` variable) like in CoffeeScript. CoffeeScript `(arg) ->` == es6 `(arg) =>`

<!--?prettify lang=javascript linenums=false?-->

    // single line arrow functions do not require braces
    app.get('posts', (req, res) => res.render('/posts.html') );

    // multi line arrow functions require braces
    app.get('posts', (req, res) => {
        console.log('in post route');
        res.render('/posts.html');
      }
    );

Braces are required in multi-line functions because unlike CoffeeScript, JavaScript does not care about indentation.

### 3 ES6 Module Pattern
As folks started building large-scale JavaScript apps, it became necessary to keep code organized and stop variables from conflicting with each other. This need became amplified when JavaScript matured to the point where apps usually integrated other people's code. `RequireJS` and `CommonJS` module patterns rose up to power most web apps and node respectively. ES6 is the first version of JavaScript that comes with its own native module loading pattern and syntax. It's definitely one of the biggest features of ES6.

<!--?prettify lang=javascript linenums=false?-->

    // node (CommonJS) style module export pattern
    module.exports = {
    	initialize: function() { ... },
    	renderPost: function(postName) { ... }
    }

<!--?prettify lang=javascript linenums=false?-->

    // node style module import pattern
    var blog = require('./blog');
    blog.initialize();
    blog.renderPost('hello-world');
---
<!--?prettify lang=javascript linenums=false?-->

    // es6 style module export pattern
    export function initialize() { ... }
    export function renderPost(postName) { ... }

<!--?prettify lang=javascript linenums=false?-->

    // es6 style function import pattern
    import {initialize,renderPost} from './blog'
    initialize();
    renderPost('hello-world');

    // es6 style namespaced import pattern
    import * as blog from './blog';
    blog.initialize();
    blog.renderPost('hello-world');

## 4 Setting up ES6 with NodeJS
Setting up es6 with node is not as clean as I would have liked. We might be a little bit early adopting Harmony, but our engineering culture is ok with being on the bleeding edge with mainstream technologies. There is a flag you can pass node like `node index.js --harmony` but it only works with the current dev branch of nodejs and which explains why I never got it to work.

### 4.1 es6-module-loader package
To get es6 working in nodejs, we can install a package `npm i es6-module-loader -S` and wrap our index.js file in a 'bootstrap' file that imports our app using the es6 module pattern. From there all code inside the import supports es6 features.

<!--?prettify lang=javascript linenums=false?-->
    
    // bootstrap.js file

    var System = require('es6-module-loader').System;

    System.import('./index').then(function(index) {
	    index.run(__dirname);
    }).catch(function(err){
	    console.log('err', err);
    });

Note you should tack on the `catch()` promise function so you can see stack errors. 

Once we setup our `boostrap.js` file, we no longer start our app with `node index.js` but instead use:

    node boostrap.js

### 4.2 appdir vs __dirname
The ES6 module system mixes up the normal `__dirname` node variable, so you can't gracefully mix and match ES6 styles imports with require syntax and use consistent relative paths. To get around this Uri showed me a solution to pass in the __dirname value from the bootstrap code (above) and add it (in index.js) onto our express app object for later reference like so:

<!--?prettify lang=javascript linenums=false?-->

    export function run(appdir)
    {
	    var express = require('express');
	    var app = express();
	    app.dir = appdir;
	    ...
	    app.use(express.static(app.dir + '/app'));
	    ...
    }


### 5 Summary

I was really keen to embrace ECMAScript6. I'd had fooled around trying to set it up myself for 30 minutes and got nowhere. Uri Shaked helped me understand the bootstrapping process in 10 minutes. I know it would have taken me more than an hour. With these cutting edge scenarios, it feels great to leverage someone else's fiddling time to save my own. It was also really fun to feel my expert enjoying what we were setting up just as much as I did.

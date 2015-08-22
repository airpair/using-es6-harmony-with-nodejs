Synopsis
> ES6 is the latest version of JavaScript. Coming from a love for CoffeScript
its new features which are similar but a bit less radical than many Coffe features have been enough for me to get back into normal JavaScript. This
post covers how to setup EST for anyone that wants to user ECMAScript6 with NodeJS as well as exploring some of the features with  a perspective
of coming back from Coffee.

** \* This post was originally published Oct 2014, has been slightly updated
in Aug 2015 and due around Nov for a major opinionated re-write exploring different ES6 patterns developed over the last year. **


## Why use ES6 Harmony?
ES6 is short for ECMAScript6, which is the blue print for upcoming version of JavaScript. As a language, JavaScript is a double edged sword. It can be freakingly ugly, but at the same time extremely flexible and powerful. JS code is often criticized for having un-necessary redundancy. Thus abstractions like `CoffeeScript` and `TypeScript` became very popular. I personally love writing in CoffeeScript, but CoffeeScript threatens to fragment the JavaScript community, its code bases and package eco-system. So the majority of JavaScript influencers frown upon abstractions and instead opt to patiently help make JavaScript get better. 

ES6 (code named Harmony) introduces some new JavaScipt language features similar to CoffeeScript. Cleaner iterators (loop syntax), arrow functions, de-structuring assignment and even classes to name a few.

## ES6 features that excite me

I was immediately jazzed to regain the following similarish CoffeeScript features within the first hour of having Harmony running:

### Cleaner iterator syntax

Thank god, no more index and length based looping in my code!

````javascript
// es6 set style iterator syntax
for (var post of posts) {
	console.log(post.url);
}
````

Careful of the `of` keyword. I frequently after a year still type `in`
and wonder what is going wrong with my code.

### Destructuring assignment

Destructuring assigment allows you to pull out attributes of an object and assign them to scope level variables in one line.

````javascript
var post = {
  url: '/first-post',
  title: 'My First Post'
}

// es6 de-structuring assignment
// Similar to CoffeeScript, but important to use the var
var {url,title} = post;

console.log('url', url, 'post', post);
// => 'url', '/first-post', 'post', { url: '/first-post',title: 'My First Post' }
````

ES6 Destructuring assigment is especially handy for pulling out separate functions & values from an imported object, but be careful the ES6 version
is a bit less forgiving than CoffeeScript which hadles undefined attributes
silently, where ES6 will throw an error.

### Arrow functions
Arrow function in es6 are quite differnt to CoffeeScript. When you first leave Coffee they feel like a huge setback, but they have their own interesting usages when you get the hang of thenm! 

There is only a fat arrow function definition (`=>`) which has very differnt behavior for from the CoffeScript magic with the function context (`this` variable). 

Also there are some nuances around syntax sugar for fat arrow functions.
<!--CoffeeScript `(arg) ->` == es6 `(arg) =>`-->

````javascript
// single line arrow functions do not require braces or the "return" keyword
app.get('posts', (req, res) => res.render('/posts.html') );

// multi line arrow functions require braces
app.get('posts', (req, res) => {
    console.log('in post route');
    res.render('/posts.html');
  }
);
````

Braces are required in multi-line functions because unlike CoffeeScript, JavaScript does not care about indentation. Be really careful when you
change a one line function to a multi-line to reinclude the the `return`
keyword.

### ES6 Module Pattern
As folks started building large-scale JavaScript apps, it became necessary to keep code organized and stop variables from conflicting with each other. This need became amplified when JavaScript matured to the point where apps usually integrated other people's code. `RequireJS` and `CommonJS` module patterns rose up to power most web apps and node respectively. ES6 is the first version of JavaScript that comes with its own native module loading pattern and syntax. It's definitely one of the biggest features of ES6.

<!--?prettify lang=javascript linenums=false?-->

    // node (CommonJS) style module export pattern
    module.exports = {
    	initialize: function() { ... },
    	renderPost: function(postName) { ... }
    }

````javascript
// node style module import pattern
var blog = require('./blog');
blog.initialize();
blog.display('hello-world');
````
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

  
- - - 
> **Update: Aug, 2015**  
I really didn't like ES6 Modules and ended up converting the whole of
airpair.com back tgo CommonJS style. I'm looking discussing why. The main
issues were:
1. We use browserify on the front end without including es6 source mappings.
CommonJS meant we could reuse code more easily on the front-end and backend.
2. Forcing all imports to happen on app load (since import must be used up
the top) meant slow app load times. AirPair uses LOTS of lazy loading of
JS files to shave about 8 seconds of app start time.
- - -


## Setting up ES6 w NodeJS Option 1
Setting up es6 with node is not as clean as I would have liked. We might be a little bit early adopting Harmony, but our engineering culture is ok with being on the bleeding edge with mainstream technologies. There is a flag you can pass node like `node index.js --harmony` but it only works with the current dev branch of nodejs and which explains why I never got it to work.

### es6-module-loader package
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

### appdir vs __dirname
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

## Setting up ES6 w NodeJS Option 2
- - -
Update Aug 2015
> ### ES6 `traceur` compiler = a better alternative
`es6-module-loader` was kind of annoying with mixing module styles, 
the appdir vs __dirname annoyance and was not compatible with running 
CoffeeScript specs from Mocha.
Luckily a better solution came my way using the `traceur` and `traceur-source-maps` packages to "bootstrap" any modules called after
setting up es6 source maps to interpret es6 syntax.
### es6 on Node with traceur
The AirPair project stucture looks partially like:
````
/server
- /util
-- setup.js
/test
- /server
-- bootstrap.coffee
- bootstrap.js
bootstrap.js
app.js
````
You'll notice, 3 differnt "bootstrap" files. Each of these files runs the
app with a different configuration / mode. 
1. `mocha /test/server/bootstrap.coffee` runs the app in test mode outputting 
mocha reporter results to the terminal. When mocha is finished the process
dies.
2. `node /test/bootstrap.js` runs the app in test mode, but as a node process
you can hit via your browser to use the site with test config (good for things
like oauth UX debugging)
3. `node bootstrap.js` runs the app in dev mode if local or prod if on heroku.
All 3 contexts use the same `/server/util/setup.js` file which looks like this:
```` javascript
var colors  = require('colors')
var traceur = require('traceur')
require('traceur-source-maps').install(traceur)
traceur.require.makeDefault(function (filePath) {
  return !~filePath.indexOf('node_modules')
})
````
Basically setup.js can't have es6 syntax, but once called, everything
required from `bootstrap` can.  
````javascript  
// root bootstrap.js file
var setup = require('./server/util/setup')
require('./app').run()  // fire up express and everything else
````
- - -


## ECMAScript6 is a great start!

I was super happy to be on the path to a nicer JavaScript. There were lots
of things I learned over the last 12 months that I'm looking forward to
sharing in a revision of this post soon. I still like writing my tests
in CoffeeScript thoug :). There are some good middle grounds that work
across both languages that can help context switching quite manageable.
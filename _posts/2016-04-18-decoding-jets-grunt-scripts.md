---
layout: post

title: "Decoding JET's Grunt Scripts"

excerpt: "Tracing the Grunt tasks that come from the JET Yeoman generator"

tags: [Oracle JET, JavaScript, Dev]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

If you've scaffolded out a JET QuickStart project using the Yeoman generator, you may have noticed some [Grunt]() tooling in a `scripts` folder in the root of your project. If you haven't, give it a try at the command prompt (assuming you [already have Yeoman installed](http://yeoman.io/learning/index.html)):

{% highlight bash %}
# this gets the JET generator installed once for all time
npm install -g generator-oraclejet

# this generates a JET application with QuickStart_Basic template into a yoJET directory
yo oraclejet yoJET --template=basic

{% endhighlight %}

After this runs, a message even tempts you regarding your new Grunt powers with the hint that you can change to the yoJET directory and run `grunt build` and `grunt serve`. 

So we know there are at least two Grunt tasks delivered in the Oracle JET Yeoman generator. But did you know they run differently in development mode vs. release mode? What other coolness lurks in the JET Grunt implementation?

##A leaned-down Gruntfile.js
In the root of your generated yoJET application is a `Gruntfile.js` file. This is Grunt's most important file, and in most projects it can get very long and cluttered with many lines of configuration for all of the Grunt tasks you want to set up.

But in the generated version, you'll see that things are lean and tidy. In fact, there's none of the normal `grunt.loadNpmTasks()` calls nor endless configuration settings. Instead, the JET Grunt setups lean on [grunt-load-config](https://github.com/firstandthird/load-grunt-config) and [jit-grunt](https://github.com/shootaroo/jit-grunt) to implement the configs and load everything on the fly.

####Gruntfile.js
{% highlight js %}
/**
 * Copyright (c) 2014, 2016, Oracle and/or its affiliates.
 * The Universal Permissive License (UPL), Version 1.0
 */
"use strict";

var path = require("path");

/*
 * Currently the tooling uses load-grunt-config to manage it's tasks.
 * In future grunt plugin will be created for better management
 */

module.exports = function (grunt) 
{
  require("jit-grunt")(grunt, {});

  require("load-grunt-config")(grunt, 
  {
    configPath: path.join(process.cwd(), "scripts/grunt/config"),
    
    jitGrunt: 
    {
      customTasksDir: "scripts/grunt/tasks"
    },

    data: 
    {
      oraclejet: 
      {
        ports: 
        {
          server: 8000,
          livereload: 35729
        }
      }
    }
  });
};

{% endhighlight %}

Notice that the `configPath` is being redirected to the `scripts/grunt/config` folder in our project. There's a whole folder structure with Grunt settings created by the Yeoman generator:

<div class="full zoomable"><img src="/images/20160418/grunt-tree.png"></div>

##What's up with the Scripts folder
This structure is how `load-grunt-config` likes to think. By separating each Grunt task's configs into their own files it keeps things orderly and easy to edit. Meanwhile, `jit-grunt` takes care of loading the node_modules plugins based upon the task names you use instead of loading them all up at the beginning of your Gruntfile. 

The `scripts/grunt/config` files fall into the proper naming pattern and contain the configurations that would normally be in the Gruntfile's `grunt.initConfig()`. A module's configs are stuffed into a `module.exports` object. However, note that the module's config object notation changes ever so slightly: you don't need the eponymous outer object named after the modules and can instead just export the meat of the Grunt module's config. Compare, for example, the generated `scripts/grunt/config/clean.js` export with a [sample Gruntfile config for grunt-contrib-clean](https://github.com/gruntjs/grunt-contrib-clean#-all-tasks) --- there's no outer `clean: {...}` object in the JET-generated case.

The `scripts/grunt/common` folder contains importable variables used by the tasks and configs, such as `build.js` and `bowercopy.js.` These imports are more modular than the global values set in the Gruntfile's `data: {}` object.

Finally, under the `scripts/grunt/tasks` folder you'll find our two seeded Grunt tasks: `build.js` and `serve.js`. These tasks can call the other modules with various targets or even set options based upon which target you call the task with.

##Out-of-the-box Grunt commands

So the fresh-from-the-generator setup for a JET project includes at least the following Grunt tasks:

* `grunt serve` (same as `grunt serve:dev`) -- spins up a webserver pointing at your project code, watches for changes to your files, and uses liveReload to refresh the browser upon changes
* `grunt build` (same as `grunt build:dev`) -- doesn't do anything. Even though the `:dev` target is the default, there isn't anything to "build" since the dev files are served directly out of the project root
* `grunt build:release` -- runs a boatload of tasks which create a `/release` folder in the root of the project and then copy the code and assets needed to deploy your JET project. Notice that the tasks that run include uglifying your JavaScript and moving copies of the Bower-sourced libraries into place
* `grunt serve:release` -- spins up a webserver that points at that `/release` folder so you can verify that everything built correctly before deploying to a production webserver
* `grunt bowercopy` -- an interesting task that copies specified files from the bower_components directory into the js/libs directory. The Yeoman generator accomplishes this when it scaffolds out your project, as you can see (for example inside of `main.js`) that everything points to the supporting files in the `/js/libs/` folders. But if you later use `bower install` to add an additional library of your own to your project, you'll want to update the `scripts/grunt/config/bowercopy.js` file and then rerun `grunt bowercopy` so that your new dependencies are "inside" your project instead of referencing them straight out of the `bower_components` folders


Understanding the way JET leverages `load-grunt-config` and how the files in the `scripts/grunt` folders work together opens up the possibility of tweaking the Grunt tasks to match our build process, and also gives us the knowledge needed to add additional Grunt modules to our projects.

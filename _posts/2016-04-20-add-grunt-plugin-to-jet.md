---
layout: post

title: "Adding a Grunt Plugin to JET"

excerpt: "Our first Grunt plugin.  So very proud."

tags: [Oracle JET, JavaScript, Dev]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

So let's say we're [cool with the whole Grunt-for-a-JET-project thing](http://likeahouseafire.com/2016/04/18/decoding-jets-grunt-scripts/) and now want to extend the scripts by adding a new plugin.

We'll add something simple first. I like [time-grunt](https://www.npmjs.com/package/time-grunt) because I'm always curious about the metrics. Adding it to our project will display the elapsed execution time of all of the tasks whenever we run Grunt.

First we need to download and install the plugin. Then we need to configure our Grunt scripts to fire it up. 

Grunt plugins are installed via npm and registered in the `package.json` file in the root of your JET project. Execute this command to download and install time-grunt:

{% highlight text %}

npm install time-grunt --save-dev
{% endhighlight %}

The `npm install` part pulls down the code needed for time-grunt and puts it in your node_modules folder. The `--save-dev` part adds an entry to your package.json file.

Now we need to add it to the Gruntfile so that it will run on every invocation of `grunt`.  Because it's watching over everything Grunt does and measuring the time it takes, time-grunt loads differently than normal task modules and we don't use the load-grunt-config pattern of adding a `scripts/grunt/config` file like a task. Instead, we put it directly in the Gruntfile, despite the fact that the generated JET code leans on the load-grunt-config approach for all of its other plugins.

####Gruntfile.js
{% highlight js %}
...
module.exports = function (grunt) 
{
  require("jit-grunt")(grunt, {});

  require('time-grunt')(grunt);

  require("load-grunt-config")(grunt, 
  { ...
{% endhighlight %}

Now let's run something with Grunt that takes a while and see the timing results printed out at the end:

{% highlight bash %}

$ grunt build:release
Running "build:release" (build) task

Running ...

...

...

Done, without errors.


Execution Time (2016-04-22 17:54:02 UTC)
loading tasks   118ms  ▇ 1%
clean:release   195ms  ▇ 2%
uglify:release  352ms  ▇▇ 3%
copy:release     2.9s  ▇▇▇▇▇▇▇▇▇▇▇▇▇ 27%
requirejs:main   6.8s  ▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇▇ 65%
Total 10.5s

{% endhighlight %}


time-grunt is unique in how it runs globally instead of being called as a task. [Next we'll configure a plugin that only runs for certain targets](http://likeahouseafire.com/2016/04/22/using-grunt-to-create-war/) instead of every time Grunt is called.



---
layout: post

title: "Using Grunt to Build a WAR&nbsp;File"

excerpt: "Leveraging Oracle JET's Grunt setups to create a WAR file from the release build process output in order to deploy a JET project to WebLogic"

tags: [Oracle JET, JavaScript, Dev]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

Now that we've [boned up on the Grunt tooling that comes with a Yeoman-generated JET project](http://likeahouseafire.com/2016/04/18/decoding-jets-grunt-scripts/) and have even [installed a Grunt plugin](http://likeahouseafire.com/2016/04/20/add-grunt-plugin-to-jet/), let's take it to the next level and have Grunt actually make something for us.

When it comes to JET projects, they're just files of static assets: HTML, CSS and JavaScript. They can be deployed to any webserver that can deliver the files to a browser. We've even served JET up with Node.js using Express.

But here at work we also have a lot of Java EE servers sitting around. A app server like WebLogic can take a WAR file and deliver the static content even if there's no Java classes or JSPs. With a little tweaking, you [can create a valid WAR file made up of nothing but static HTML, CSS and JS content](https://blogs.oracle.com/middleware/entry/publish_static_content_to_weblogic).

But creating a WAR and copying in the JET files by hand is boring, repetitive work. The kind of work that Grunt is great at!

##Installing grunt-war 
We'll use the [grunt-war plugin](https://www.npmjs.com/package/grunt-war) to generate the actual WAR file. The step to build the WAR can come at the end of the existing `grunt build:release` task. Before we can do that, we'll need to add the proper configs to a file in `scripts/grunt/config` and format them the way JET's load-grunt-config setups are expecting.

First download and register grunt-war by executing this at the command prompt:

{% highlight bash %}

npm install grunt-war --save-dev
{% endhighlight %}

JET's use of [jit-grunt](https://www.npmjs.com/package/jit-grunt) means we don't need to add a `loadNpmTasks` call for grunt-war to our Gruntfile; it will get loaded automatically when we run a task that needs it. However we do need to set up grunt-war's config options to load properly.

##Creating scripts/grunt/config/war.js
The [configuration settings for grunt-war](https://www.npmjs.com/package/grunt-war#the-war-task) would normally be set in `grunt.initConfig()` in the Gruntfile. But since JET is using [load-grunt-config we instead make configuration settings modular](http://www.html5rocks.com/en/tutorials/tooling/supercharging-your-gruntfile/) for all of our Grunt plugins, including grunt-war.

We need to create a file named `scripts/grunt/config/war.js` so that load-grunt-config finds the right settings for our grunt-war plugin. Since the plugin is named grunt-**war**, the config file should be named **war**.js.

The config file uses `module.exports = {}` to return an object with the settings we want to use for our plugin. I've commented up the ones that worked for me, including some important additions that make the generated WAR file WebLogic- and JCS-SX-friendly:

####war.js
{% highlight js %}

module.exports =  {
 
  /*
   * Build a WAR (web archive) without Maven or the JVM installed.
   */
      
        target: {
          options: {
            war_dist_folder: 'dist',      /* Folder to generate the WAR into */
            war_name: 'yoJET',            /* The name fo the WAR file (.war will be the extension) */
            webxml_webapp_version: '2.5', /* I needed this older version for JCS-SX */  
            war_extras: [ {filename: 'grunt-war-credits.txt', data: 'This line will appear in the file!\n see http://likeahouseafire.com/2016/04/22/using-grunt-to-create-war/ '}, 
                          {filename: 'WEB-INF/weblogic.xml', data: '<?xml version="1.0" encoding="UTF-8"?>\n<weblogic-web-app xmlns="http://www.bea.com/ns/weblogic/90" xmlns:j2ee="http://java.sun.com/xml/ns/j2ee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.bea.com/ns/weblogic/90 http://www.bea.com/ns/weblogic/90/weblogic-web-app.xsd">\n  <jsp-descriptor>\n    <keepgenerated>true</keepgenerated>\n    <debug>true</debug>\n  </jsp-descriptor>\n  <context-root>/yojet</context-root>\n</weblogic-web-app>'}],
                                          /* the war_extras are extra files to be generated, needed since grunt-war doesn't create a weblogic.xml */  
            webxml_welcome: 'index.html', /* to point web.xml to the default page */
            webxml_webapp_extras: [ '<login-config />\n', '<session-config>\n    <session-timeout>\n    30\n    </session-timeout>\n</session-config>\n' ]  
                                          /* some extra settings for web.xml to work with JCS-SX */

          },
          files: [
            {
              expand: true,
              cwd: 'release',             /* find the source files for the WAR in the /release folder */
              src: ['**'],
              dest: ''
            }
          ]
        }
     
  
  
};
{% endhighlight %}

Note especially the `<context-root>` settings in the `war-extras` property and all the other tweaks made to override the defaults and create a WAR file worthy of deploying to JCS-SX.

With this config file, we could now kick out to the command prompt to execute `grunt war` and it would try to build a WAR file out of whatever it found in the `release` folder.

## Modifying scripts/grunt/tasks/build.js
But since the `release` folder gets a clean, generated copy of our JET project whenever we run `grunt build:release`, let's put the WAR-building task in as the last step in the build process.

Open the existing `scripts/grunt/tasks/build.js` and notice the section under `if (target === "release")`:

{% highlight js %}
...
   if (target === "release")
    {
      grunt.task.run(
      [
        "clean:release",
        "injector:mainReleasePaths",
        "uglify:release",
        "copy:release"
        "requirejs",
        "clean:mainTemp",
        "war"
      ]);
    }
 
 ...
{% endhighlight %}

I added `"war"` to the array of `grunt.task.run()` steps as the final task to run when we target `grunt build:release`.

With all this in place you'll get a deployable WAR file that includes everything from your JET project that was minified and copied into the release folder as part of the build process. 

---
layout: post

title: "Reprise: Using Grunt to Create a WAR file"

excerpt: "Updated for JET 3.x: Leveraging Oracle JET's Grunt setups to create a WAR file from the release:build process to deploy a JET project to Java Cloud"

tags: [Oracle JET, Grunt, JCS, Dev]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

A long time ago, I drafted [notes on how to tweak Oracle JET's Grunt configurations to automatically build WAR files](http://likeahouseafire.com/2016/04/22/using-grunt-to-create-war/). This was in the days of JET 2.x, so those notes don't work quite right anymore.  By the time JET 3.0.0 shipped, [major changes](http://www.oracle.com/webfolder/technetwork/jet-300/globalSupport-releaseNotes.html#tooling_changes) to the Grunt tooling changed the way the `grunt serve` and `grunt build` commands worked internally.

In the "old" days, the Oracle JET tooling was a series of Grunt tasks that were run in sequence, such as `bowercopy` and `uglify`.  These would run in order when you executed a command like `grunt build:release`.  But in the current version, these individual commands were replaced with a single npm module named `oraclejet-tooling` that runs all of the build or serve steps as a single Grunt task.  Although this task can be configured to act differently, we have to change the the way we extend our build process with additional steps if we want to do something like build an asset for deployment.

Instead of modifying the tooling's `oraclejet-build` steps, we'll tack our WAR-creation step on the end of the build task.

##Installing grunt-war

We'll use a clean JET 3.2.0 NavBar project as the basis.  If you don't already have a JET project, scaffold one out at the command line:

{% highlight bash %}
yo oraclejet 320navbar --template=navbar
{% endhighlight %}

Change to the project folder and then install the [grunt-war plugin](https://www.npmjs.com/package/grunt-war) same as before:

{% highlight bash %}
npm install grunt-war --save-dev
{% endhighlight %}


##Creating scripts/grunt/config/war.js

If you look around in your 3.x project, you'll see that the `/scripts/grunt/` directory is pretty clean as compared to the JET 2.0.0 days.  In fact, there are only two files in the `config` directory: `oraclejet-build.js` and `oraclejet-serve.js`, and both of them are mostly commented out.  These files are the way you configure the behavior of the oraclejet-tooling, the details of which is now hidden away in the `node_modules` directory.

But this directory is still the place where we put our configuration for the newly-added `grunt-war` module.  As before, since the plugin is named grunt-**war**, the config file should be named **war**.js.  Create a file named `scripts/grunt/config/war.js`:

####war.js
{% highlight js %}

module.exports =  {
 
  /*
   * Build a WAR (web archive) without Maven or the JVM installed.
   *
   * Template strings in the <%= %> tags are set in the data section of Gruntfile.js,
   *  or you can hardcode the strings here instead
   */
      
        target: {
          options: {
            war_dist_folder: '<%= distdir %>',      /* Folder to generate the WAR into, set in data section of Gruntfile.js */
            war_name: '<%= appname %>',            /* The name for the WAR file (.war will be the extension) */
            webxml_webapp_version: '2.5', /* I needed this older version for JCS-SX */  
            war_extras: [ {filename: 'grunt-war-credits.txt', data: 'This line will appear in the file!\n see http://likeahouseafire.com/2017/08/09/updated-using-grunt-to-create-war-jet3x/ '}, 
                          {filename: 'WEB-INF/weblogic.xml', data: '<?xml version="1.0" encoding="UTF-8"?>\n<weblogic-web-app xmlns="http://www.bea.com/ns/weblogic/90" xmlns:j2ee="http://java.sun.com/xml/ns/j2ee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.bea.com/ns/weblogic/90 http://www.bea.com/ns/weblogic/90/weblogic-web-app.xsd">\n  <jsp-descriptor>\n    <keepgenerated>true</keepgenerated>\n    <debug>true</debug>\n  </jsp-descriptor>\n  <context-root>/<%= appname %></context-root>\n</weblogic-web-app>'}],
                                          /* the war_extras are extra files to be generated, needed since grunt-war doesn't create a weblogic.xml */                                          /* also notice that we're using the <%= appname %> variable in there */  
            webxml_welcome: 'index.html', /* to point web.xml to the default page */
            webxml_webapp_extras: [ '<login-config />\n', '<session-config>\n    <session-timeout>\n    30\n    </session-timeout>\n</session-config>\n' ]  
                                          /* some extra settings for web.xml to work with JCS-SX */

          },
          files: [
            {
              expand: true,
              cwd: '<%= appdir %>',             /* find the source files for the WAR in the /web folder, set in Gruntfile.js */
              src: ['**'],
              dest: ''
            }
          ]
        }
     
  
  
};
{% endhighlight %}

Note that this code has a bunch of fancy template string variables in `<%=  %>` tags.  This is so the WAR filename and context root can be set the same as the project directory name, as well as setting the build directory locations.  We'll set these variables in the Gruntfile, below.

## Modifying Gruntfile.js

Before, we added the war task to the build step by modifying a `build.js` file, but this doesn't exist in our project source files anymore because the `oraclejet-build` step is encapsulated inside the `node_modules` directory.

But we are still OK because we don't really want to interject our WAR step in the middle of the `oraclejet-build` steps; we want it to come at the very end of the flow.  Since this is the case, we can add our war step to the project's `Gruntfile.js`:

####Gruntfile.js
{% highlight js %}
'use strict';

var path = require('path');

module.exports = function(grunt) {

  require('load-grunt-config')(grunt, {
    configPath: path.join(process.cwd(), 'scripts/grunt/config'),
    data: {
      appname: path.basename(process.cwd()),  // same as project directory name, accessible with '<%= appname %>'
      appdir: 'web',  // accessible with '<%= appdir %>'
      distdir: 'dist'  // accessible with '<%= distdir %>'
    }
  });

  grunt.loadNpmTasks("grunt-oraclejet");

  //grunt.loadNpmTasks("grunt-war");

  grunt.registerTask("build", "Public task. Calls oraclejet-build to build the oraclejet application. Can be customized with additional build tasks.", function (buildType) {
    grunt.task.run([`oraclejet-build:${buildType}`, 'war']);
  });

  grunt.registerTask("serve", "Public task. Calls oraclejet-serve to serve the oraclejet application. Can be customized with additional serve tasks.", function (buildType) {
    grunt.task.run([`oraclejet-serve:${buildType}`]);
  }); 
};
{% endhighlight %}


Note that we simply add our `war` task to the end of the array in the "build" `registerTask` function:

{% highlight js %}
    grunt.task.run([`oraclejet-build:${buildType}`, 'war']);
{% endhighlight %}

Since we want it to happen after the `oraclejet-build` task is done, this will work perfect.

Also note that the template string variables like `<%= appname %>` and `<%= distdir %>` are set here in the `data` section of this file, and that we can use Node.js functions like `path.basename()` to dynamically get the name of the directory.  These template strings could be used in other custom Grunt tasks as well.

This is all we need to update our WAR file scripts.  Run `grunt build:release` and you'll have a WAR file sitting in the `/dist` directory, suitable for deploying to JCS.  We could use this same idea to deploy to ACCS or other platforms that need a zipped deployment asset with manifest files by tweaking the settings in the `war.js` config file.




## But what if you did want to change the flow of oraclejet-build?

With the details of the `build` and `serve` tasks hidden away in the `node_modules` directory, you might think you can't change their behavior.  But JET makes provision for customizing some of the steps that these two tasks follow.

See the [Customize a Web Application’s Grunt Build Behavior](http://docs.oracle.com/middleware/jet320/jet/developer/GUID-ACB7BD4E-BAAC-4A9E-B52A-6B2933CD222C.htm#GUID-35A28E14-A97E-48A6-8C0D-64E9E5DF77AB) section of the documentation for details on how JET dynamically merges together changes you make to the files `scripts/grunt/config/oraclejet-build.js` and `oraclejet-serve.js` with its own settings when you run Grunt.  

Also, you can have a looky-see at the source code in `node_modules/oraclejet-tooling`, especially the `node_modules/oraclejet-tooling/lib/defaultconfig.js` file, for more insight on how the tooling works.  Just remember not to tweak the files under `node_modules` because that defeats the whole idea of a package manager. 






---
layout: post

title: "Nav Drawer is the New QuickStart"

excerpt: "Dissecting the new Yeoman templates in OracleJET 2.1.0 "

tags: [Oracle JET, Yeoman, Grunt, Dev]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

With the release of Oracle JET 2.1.0, there are new "starters"--templates that make for quick setup on a new JET project and show some best practices for code structure. You use the new templates via the same [Yeoman scaffolding process](http://docs.oracle.com/middleware/jet202/jet/developer/GUID-ACB7BD4E-BAAC-4A9E-B52A-6B2933CD222C.htm#GUID-F9B1A1E1-2814-49A0-A59A-0ADAAEFC5E93) for a starting a new project.

{% highlight bash %}
yo oraclejet jet210navdrawer --template=navdrawer
{% endhighlight %}

[Edit:]  If the `navdrawer` option doesn't work for you, you may need to update your Yeoman generators to get the latest `generator-oraclejet`. Either do the update through Yeoman's menu system by executing `yo` at the command line, or use this command:

{% highlight bash %}
npm update generator-oraclejet -g
{% endhighlight %}

[/end Edit]
																																																																																																																																																																																																																							
In the past there were only two templates: `blank` for a bare-bones minimal JET scaffolded project with no router, navigation, or sample content views/viewmodels, and `basic` for a QuickStart project. In JET v2.1.0, `blank` and `basic` are still there but now produce different assets--both rather basic--and they are joined by two newcomers: `navbar` and `navdrawer`. Descriptions and links to demos are under the 'Starters' section of the [JET examples landing page](http://www.oracle.com/webfolder/technetwork/jet/globalExamples.html).

The `blank` template is now truly blank: it produces a JET project with no content at all and leaves it to you to implement routing and content components, but it does have the Alta styles, framework references, and Grunt tasks in place. The `basic` template is no longer QuickStart and is pretty blank itself, although it does at least have a header and footer in the index.html.

So where did QuickStart go? It's been replaced by two new templates: `navbar` and `navdrawer`. [Nav Bar](http://www.oracle.com/webfolder/technetwork/jet/globalExamples-Starter-NavBar.html) is the newest and transforms the navigation menu into a bar of icons when the smaller responsive breakpoints kick in.  [Nav Drawer](http://www.oracle.com/webfolder/technetwork/jet/globalExamples-Starter-Drawer.html) will be more familiar to QuickStarters, as it uses the normally-hidden sidebar drawer to shrink down the menu at phone and tablet breakpoints.

These two templates are otherwise identical and scaffold out a router and a few content pages. The three-column main content area of QuickStart is now a single full-width flexbox that gets its content from where the router tells it to. 

There are a few other changes under the hood. Application-wide configuration settings, such as for navigation and the router, have been moved into an `appController.js` module that gets pulled in by `main.js`. This is more modular and provides a pattern for your own global settings that could then be pulled into viewModels in your app as needed.

The individual views and viewModels such as for the Dashboard and Incidents tabs (no more Home tab to delete) don't do much, but it's interesting that each of the sample viewModels draw attention to the [ojModule's lifecycle handlers](https://docs.oracle.com/middleware/jet202/jet/developer/GUID-ABB82BD1-9B65-44D2-AE43-81A0A3A44453.htm#JETDG-GUID-ABB82BD1-9B65-44D2-AE43-81A0A3A44453) by implementing stub methods such as `handleActivated` and `handleAttached`. These are empty methods on a newly-instantiated ojModule so having them blank here doesn't do anything different, but seeing them in your face draws attention to the fact that you can hook into them whenever you use an ojModule (which the router does to swap in your main content).

There's a new `themes` folder under the site root that gets leveraged by the `grunt serve` tasks. [Themes will let you define sets of styles](http://www.oracle.com/webfolder/technetwork/jet/jetCookbook.html?component=theming&demo=themename) that can be swapped in and out at build time. Look for more to come around the [Theme Style Lab](http://www.oracle.com/webfolder/technetwork/jet/globalExamples-ThemeViewer.html).

Another change is to the `/scripts/grunt` folder. This has been massively simplified [compared to earlier versions](http://likeahouseafire.com/2016/04/18/decoding-jets-grunt-scripts/). The two tasks are still `grunt serve` and `grunt build`, but the Oracle JET-specific tasks are [now imported from a grunt-oraclejet NPM module](https://www.npmjs.com/package/grunt-oraclejet).

There are probably more cool things in these updated templates, but these stood out after a quick review.  I really liked using the QuickStart as a seed for my JET projects, and am looking forward to upgrading our team's seed repo with the 2.1.0 changes in the Nav Drawer template. 


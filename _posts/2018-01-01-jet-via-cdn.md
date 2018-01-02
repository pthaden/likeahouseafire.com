---
layout: post

title: "Deliver JET via CDN"

excerpt: "Configure Oracle JET to use a CDN and reduce the deployment footprint"

tags: [Oracle JET, IoT, Dev]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

While playing with some IoT devices such as the JavaScript-centric [JohnnyFive Tessel 2 board](https://www.sparkfun.com/j5ik), I wanted to deploy some JET code onto the microcontroller to be delivered by the board's Express Node.js server.  

In the Tessel tutorial there is [sample code that uses the JustGage library](https://learn.sparkfun.com/tutorials/experiment-guide-for-the-johnny-five-inventors-kit/experiment-10-using-the-bme280) to show sensor data streaming via Socket.io. The deployment steps in that tutorial describe how to ship the supporting node_modules over to the Tessel 2 board, but the more you add to your project the heavier the deployment becomes and the longer it takes to copy over.

<figure class="full zoomable"><img src="/images/20180101/justgage-gauges.png"><figcaption>Gauges built with JustGage, which is deployed and served by the Node.js Express server</figcaption></figure>

Shipping all of the JET supporting modules and all of their dependencies is a whole lot heavier than only one library like JustGage.  Even if you take steps to whittle down the included JET module bundles or minimize the code that gets copied to `web/js/libs/`, it's still a boatload of code to deploy to the Tessel -- especially since it will all simply be served right back to a client and isn't really needed to be resident on the board.

##Using a CDN instead

The client machine running the JET code doesn't really care where the Require.js, Knockout, oj, and other supporting code comes from, as long as it gets there fast.  Instead of serving these dependency libraries off of a low-power IoT board like the Tessel, we can rewrite our JET configs and use a content delivery network to serve up all the libraries when our client accesses the app we deploy to our Express server.

Starting with v4.0.0, [Oracle JET is available via a CDN](https://docs.oracle.com/middleware/jet410/jet/developer/GUID-219A636B-0D0B-4A78-975B-0528497A82DD.htm#JETDG-GUID-219A636B-0D0B-4A78-975B-0528497A82DD) and doesn't have to be running from the same server that delivers your html and js files.

The instructions in that documentation link are straightforward:  You change the path mappings for Require.js in `main.js` and `main-release-paths.json` and also point to the hosted Alta stylesheet and Require.js bootstrap in `index.html`.  In that same file I also had to remove the `<!-- injector:theme -->` comment pair because the build scripts would otherwise try to keep reinserting the local reference to the standard theme's stylesheet instead of respecting the CDN-hosted Alta stylesheet.

##Deploying to a microcontroller's node.js

With the CDN-only version of my JET code, I was able to deploy the built `index.html`, `main.js` and `appcontroller.js` files from a `ojet create --template=basic` project to be delivered by the Express server running on my Tessel's node.js.  Per the instructions in the J5IK tutorial, I slipped the assets in an `app` folder and mapped it as a static Express route.  But thanks to the CDN there was no need to add the JET dependencies from the `node_modules` folder into the `.tesselinclude` file.

This way I "upgraded" the gauges to JET and was able to use websockets to stream the Tessel's BME280 environmental sensors' data to the JET visualizations.  The [modified code up on GitHub](https://github.com/pthaden/j5ik-Experiment10) also shows how to hook Socket.io into the Require.js framework for JET.

<figure class="full zoomable"><img src="/images/20180101/jet-gauges.png"><figcaption>Gauges built with JET's <code>&lt;oj-status-meter-gauge></code></figcaption></figure>

##Using the CDN for JSFiddles and CodePens

We had to use a similar idea in the past to deploy sample JET code to services such as JSFiddle and CodePen (which don't have an option to dynamically load JET). [Back then we set up links to raw GitHub files](/2016/09/27/jsfiddle-jet-base-for-2.1.0/) as a poor-man's CDN, but with the official approach available now I've rewritten the [CodePen](https://codepen.io/pthaden/pen/PEjWyo) and [JSFiddle](https://jsfiddle.net/pthaden/h77Logno/) templates and you can fork these to prototype JET ideas or troubleshoot implementation issues.






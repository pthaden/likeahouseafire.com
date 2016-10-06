---
layout: post

title: "Build a Standalone App with Electron and Oracle&nbsp;JET"

excerpt: "Create cross-platform, native OS desktop apps from Oracle JET codebases"

tags: [Oracle JET, JavaScript, Dev]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

Oracle JET has great [scaffolding and support for building native apps for mobile platforms](http://docs.oracle.com/middleware/jet210/jet/developer/GUID-C75CD8DC-5084-4831-BE1A-FFEE4EA8600C.htm#JETDG-GUID-C75CD8DC-5084-4831-BE1A-FFEE4EA8600C) such as iOS, Android or Windows 10, all from a single JET codebase. And of course you can build browser-based single-page apps that run on the desktop. But what about a native standalone desktop app?

[Electron](http://electron.atom.io/) is a framework for building cross-platform native apps using web stack technologies. It's the tool that the [Atom editor](https://atom.io/) is created with, and it [underpins many other hip apps](http://electron.atom.io/apps/) these days. Electron uses Node.js and the Chromium engine to run web framework code inside of  native OS windows, without needing to launch a browser.

Could we use Electron to package up an Oracle JET app and make it a cross-platform desktop executable?

## Learning about Electron

The best place to start is from the horse's mouth, and Zeke Sikelianos of GitHub gave a good overview of Electron that was recorded at their recent Universe 2016 conference:

<iframe width="560" height="315" src="https://www.youtube.com/embed/FNHBfN8c32U" frameborder="0" allowfullscreen></iframe>


There are also a [bunch of tutorials and references out on the web](https://github.com/sindresorhus/awesome-electron) on how to build an electron app. We'll be leaning on the simple [electron-quick-start](https://github.com/electron/electron-quick-start) to wrap a Oracle JET project inside of Electron.

## Putting JET in an Electron blanket

Start with a working JET app. We'll use the generator to scaffold a plain template with just a header and footer so we feel at home:

{% highlight bash %}
yo oraclejet electronJet --template=basic
{% endhighlight %}

I added one of the [cookbook visualizations](http://www.oracle.com/webfolder/technetwork/jet/jetCookbook.html?component=diagram&demo=animations) to the code for my basic template. With a `grunt serve` I've got a simple Oracle JET app:

<div class="full zoomable"><img src="/images/20161004/jet-starter-template-web-basic.png"></div>

You can follow along by [cloning this repository](https://github.com/pthaden/electronJet), checking out the `jetwebapp` tag, and running `npm start` to automatically install all the npm and bower components: 

{% highlight bash %}
git clone https://github.com/pthaden/electronJet.git
cd electronJet
git checkout jetwebapp
npm start
{% endhighlight %}

Now we have a working JET project with its own `package.json`.  Let's tweak things to add Electron into the mix. Back at the command line, install the electron npm package as a dependency:

{% highlight bash %}
npm install electron --save-dev
{% endhighlight %}

Electron is a node.js app and expects an entry in the `package.json` file for the startup script entry point. Edit the file to include a `"main":` entry pointing at a to-be-created main.js file:

#### package.json
{% highlight json %}
{
  "name": "electronJet",
  "version": "1.0.0",
  "description": "A sample Oracle JavaScript Extension Toolkit (JET) web app wrapped in Electron",
  "author": {
    "name": "Paul Thaden"
  },
  "main": "main.js",
  "scripts": { ...
  {% endhighlight %}

Next we need some node.js code to spin up an electron window and stick our JET code inside of it. Create a file in the root of the project called `main.js` (not to be confused with JET's `main.js` in the `src/js` or `web/js` folders).

#### main.js
{% highlight js %}
const {app, BrowserWindow} = require('electron')

// Keep a global reference of the window object, if you don't, the window will
// be closed automatically when the JavaScript object is garbage collected.
let win

app.on('ready', () => {
  // Create the browser window.
  win = new BrowserWindow({
    width: 800, 
    height: 600,

    // turn off node integration in the Electron window to prevent conflicts with require.js
    // see http://electron.atom.io/docs/faq/#i-can-not-use-jqueryrequirejsmeteorangularjs-in-electron
    webPreferences: { nodeIntegration: false }
  })

  // and load the index.html of the built JET app.
  win.loadURL('file://' + __dirname + '/web/index.html')
})
{% endhighlight %}

This is a bare-bones Electron app, but it will get the job done. Note the modification to the `BrowserWindow`'s `webPreferences` setting: I disabled `nodeIntegration` in the Electron renderer process because of conflicts between the `require()` function in Node and the Require.js loader in JET. You'll also see that I'm pointing to the `/web/index.html` in the JET project, which assumes you've executed `grunt build` at least once.

To test it out, kick back to the command line at the root of the project and start Electron:


{% highlight bash %}
electron .
{% endhighlight %}

If all works well, you'll have a native app window with your JET content running inside:


<div class="full zoomable"><img src="/images/20161004/electron-app-with-jet-inside.png"></div>

If you want to make it all build automatically, modify the `npm start` in the `scripts` section of the package.json file to call Electron instead of serving via Grunt:

#### package.json
{% highlight json %}
{
  "name": "electronJet",
  "version": "1.0.0",
  "description": "A sample Oracle JavaScript Extension Toolkit (JET) web app wrapped in Electron",
  "author": {
    "name": "Paul Thaden"
  },
  "main": "main.js",
  "scripts": {
    "postinstall": "bower install && grunt bowercopy",
    "prestart": "npm install",
    "start": "grunt build && electron ."
  },
  "devDependencies": { ...
  {% endhighlight %}


## Does it really make an executable?

I used [electron-packager](https://github.com/electron-userland/electron-packager) to create Mac, Win32 and Linux packaged executable bundles out of the electronJET app. I modified the `main.js` code a bit to better match the [electron-quick-start](https://github.com/electron/electron-quick-start/blob/master/main.js) code and to support the MacOS window closing and quitting conventions. 

Here's the Windows version running native on that OS:

<div class="full zoomable"><img src="/images/20161004/windows-electronjet-app.png"></div>

It's all [up on the GitHub repo](https://github.com/pthaden/electronJet), and you can download zips for each platform:

* [electronJet-darwin-x64.zip](https://github.com/pthaden/electronJet/raw/master/electron-packager%20output/electronJet-darwin-x64.zip) for MacOS
* [electronJet-win32-x64.zip](https://github.com/pthaden/electronJet/raw/master/electron-packager%20output/electronJet-win32-x64.zip) for Windows
* [electronJet-linux-ia32.zip](https://github.com/pthaden/electronJet/raw/master/electron-packager%20output/electronJet-linux-ia32.zip) for Linux

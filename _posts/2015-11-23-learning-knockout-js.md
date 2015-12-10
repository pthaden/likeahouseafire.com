---
layout: post

title: Learning Knockout.js


excerpt: "Oracle JET leans on Knockout.js for data binding.  It's not an absolute requirement because JET is modular, but..."

tags: [Knockout, Oracle JET, JavaScript]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

Oracle JET leans on Knockout.js for data binding.  It's [not an absolute requirement](https://blogs.oracle.com/geertjan/entry/data_binding_with_oracle_jet) because JET is modular, but the JET sample apps and templates do come with Knockout baked in.

This is a decent intro to Knockout.js:
[Learning Knockout.js from Packt Publishing](https://www.udemy.com/learning-knockoutjs/learn/#/).  In the video tutorials Robert Gaut reviews some design patterns and uses his foundation code to discuss bindings, context, and custom functions. They even gave me a fancy certificate!

<div class="full zoomable"><img width="300" src="/images/20151123/certificate.jpg"></div>

The video doesn't introduce AMD and RequireJS until the end and then only brushes on it in a quick lesson, so you'll need to take that in consideration when looking at the Oracle JET sample code.

For that, you could take a look at [Building Large, Maintainable, and Testable Knockout.js Applications](https://code.tutsplus.com/tutorials/building-large-maintainable-and-testable-knockoutjs-applications--net-30996) by Jonathan Creamer.  He starts his article with coverage of require.js and builds up an app from there. He also discusses "the `this` problem" to give some background on why you keep seeing `var self = this` in code samples.

Another couple articles (ht [Jim Marion](http://jjmpsj.blogspot.com/)) are [The Top 10 Mistakes That KnockoutJS Developers Make](https://www.airpair.com/knockout/posts/top-10-mistakes-knockoutjs) by Mike Mellenthin and [10 Things to Know About KnockoutJS on Day One](http://www.knockmeout.net/2011/06/10-things-to-know-about-knockoutjs-on.html). Sometimes reading what people much smarter than me and farther along in their adoption of tools or techniques helps me get a feel for what I need to learn.

Here's a video straight from the horse's mouth:  Steve Sanderson himself discusses building a single page app with Knockout in [Architecting large Single Page Applications with Knockout.js](https://vimeo.com/97519516). Sanderson has a few other recorded videos out there, but I haven't gotten to them all yet.

Also, there's the [Knockout.js website itself](http://learn.knockoutjs.com/), with a series of tutorials that walk through the basics of data binding.



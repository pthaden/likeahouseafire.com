---
layout: post

title: "JSFiddle Base for Oracle JET 2.1.0"

excerpt: "Updated scaffolding to run JET version 2.1.0 code on code pen sites"

tags: [Oracle JET, JavaScript, Dev]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

Today I updated the code for my JSFiddle base to use Oracle JET 2.1.0. This is based on work that [Cliff Sanchez](http://cliffsanchez.com/) and [Jim Marion](http://jsjim.blogspot.com/) put together to host JET examples on public code pen sites.

A while back [Jim documented the one weird trick](http://jsjim.blogspot.com/2016/04/jsfiddle-for-oracle-jet-201.html) that got JET working on JSFiddle via RawGit, which is a pseudo-CDN we can use until JET is published to a proper CDN. Also, take a [look at Cliff's work on CodePen](http://codepen.io/cliffsanchez/) in case you prefer that service.

[My v2.1.0 JSFiddle base](https://jsfiddle.net/pthaden/cbev95yr/) is embedded below. You'll see I've updated the JS tab using the [Migrating a v2.0.x Web Application to v2.1.0](http://docs.oracle.com/middleware/jet210/jet/developer/GUID-F4F792A5-709B-4999-81F8-5F80B1A62664.htm#JETDG-GUID-F4F792A5-709B-4999-81F8-5F80B1A62664) instructions. Also, I found that you can move the `requirejs.config()` block down to the bottom and it will still work--this seems to draw more attention to your example viewModel code and less to the frameworks.

<script async src="//jsfiddle.net/pthaden/cbev95yr/embed/"></script>


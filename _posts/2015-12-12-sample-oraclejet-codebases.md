---
layout: post

title: Sample OracleJET Codebases In the Wild


excerpt: "A collection of downloadable OracleJET samples"

tags: [Oracle JET, Dev]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

If you want to get started with OracleJET, the _purest_ way is to download [only the base distribution package and then create a HTML5/JS app in NetBeans](https://blogs.oracle.com/geertjan/entry/starting_with_the_oracle_jet). This creates a very plain app: an empty canvas ready for you to pour your code into from scratch.

But if you're looking to see (and possibly steal ideas from) some working code, the _easiest_ way is to use NetBeans 8.1's [QuickStart Template](http://www.oracle.com/webfolder/technetwork/jet/public_samples/OracleJET_QuickStartBasic/public_html/index.html) and let it download the Basic Starter Template zip file to build out a project for you. The wizard uses the same [Quickstart basic starter code that you can download by hand from OTN](http://www.oracle.com/technetwork/developer-tools/jet/downloads/index.html) if you don't want to use NetBeans.

As I've been ramping up on JET I've stumbled on a few other examples of working JET code that can be imported as HTML5/JS apps into NetBeans following the same steps in Geertjan's blog above. Here's a list of some JET samples in the wild:

## WorkBetter App
The [WorkBetter demo](http://www.oracle.com/webfolder/technetwork/jet/public_samples/WorkBetter/public_html/index.html) is a great showcase of what you can do with JET and stands in contrast to the [QuickStart template](http://www.oracle.com/webfolder/technetwork/jet/public_samples/OracleJET_QuickStartBasic/public_html/index.html) with fancy styling and masonry layout on the home page. If it looks familiar it's because this is the JET version of [the ADF WorkBetter sample app](http://jdevadf.oracle.com/workbetter/faces/index.jsf). The beautiful thing is that you can look at the code by downloading the zip file:

* Working copy: [JET WorkBetter Demo](http://www.oracle.com/webfolder/technetwork/jet/public_samples/WorkBetter/public_html/index.html)
* Code zip file: [WorkBetter.zip](http://www.oracle.com/technetwork/developer-tools/jet/downloads/index.html)

## moviesJET
Blogger [Kenneth Lange](http://www.kennethlange.com/) wrote an [excellent JET tutorial that mirrors an Angular.js exercise](http://www.kennethlange.com/posts/oracle_jet.html) out on the Internets. He builds a CRUD app up from the base distribution and documents the process.

* Code zip file: [oracle_jet_demo_app.zip](http://www.kennethlange.com/resources/oracle_jet_demo_app.zip)

## OracleJET-CommonModel-CRUD
Buried [deep in the JET documentation](https://docs.oracle.com/middleware/jet112/jet/developer/GUID-0C0D187C-CDCB-4235-ADA8-7AE9D93FFA08.htm#JETDG552) is another CRUD sample app. This one is based on the JET Common Model and also is an example of how to use the REST mock server. That documentation page is a rundown of the code, but it's nice to have a working copy in NetBeans to follow along with and hack on.

* Code zip file: [OraclerJET-CommonModel-CRUD](http://www.oracle.com/webfolder/technetwork/jet/public_samples/JET-CommonModel-CRUD.zip)

## SPA-ojModule-ojRouter
Another code sample can be found in the docs at the bottom of [Designing Single-Page Applications Using Oracle JET](https://docs.oracle.com/middleware/jet112/jet/developer/GUID-307B5D75-4D96-413B-A8FB-2212ED401061.htm#JETDG327). This page walks through creating a SPA with ojModule and ojRouter to create a bookmarkable website and talks about when to use query parameters versus URI path segments.


I hope they keep posting downloadable code samples from the documentation, as it illuminates the docs and proves that the steps work.

* Code zip file: [SPA-ojModule-ojRouter](http://www.oracle.com/webfolder/technetwork/jet/public_samples/SPA-ojModule-ojRouter.zip)

## ComponentsInteractions
There's another code sample on the JET OTN Downloads page named "Components Interactions" and you can download the source code from there, but I strongly recommend you instead follow the [OpenWorld Hands-On-Lab instructions](http://www.oracle.com/webfolder/technetwork/jet/globalExamples-HOL.html) and build this code yourself from scratch. The HOL instructions will walk you through it and you'll end up with the same code as below but will learn a lot more from the experience.

* Working copy: [JET-ComponentInteraction](http://www.oracle.com/webfolder/technetwork/jet/public_samples/JET-ComponentInteraction/public_html/index.html)
* Code zip file: [Components Interactions: Sample Application for Oracle JavaScript Extension Toolkit](http://www.oracle.com/technetwork/developer-tools/jet/downloads/index.html)

-----
I'm on the lookout for more sample code, so please let me know if you have links to any other good codebases.




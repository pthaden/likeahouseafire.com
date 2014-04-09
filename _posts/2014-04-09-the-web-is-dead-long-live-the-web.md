---
layout: post

title: The Web Is Dead, Long Live the Web
cover_image: bill_appletons_mobile_architecture.jpg

excerpt: "I love the Ignite format for presentations, where you have to get your point across in five minutes with 20 slides that advance automatically every 15 seconds. Bill Appleton made a good point with his Fluent 2014 Ignite..."

tags: Dev REST

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

I love the [Ignite](http://igniteshow.com/) format for presentations, where you have to get your point across in five minutes with 20 slides that advance automatically every 15 seconds.

Bill Appleton made a good point with his Fluent 2014 Ignite Presentation ["The Big Shift From Web Pages to RESTful Apps"](http://www.youtube.com/watch?v=K9kUzuNsNIs&list=PL055Epbe6d5YbV14E2hqRFm5FoUvFRmko&feature=em-share_video_in_list_user).  He reminds us that the legacy Internet is basically based on Ethernet: high bandwidth, low latency, and cheap because you pay for the connection, not the bytes.  Then he contrasts that with the new mobile web network: lower in bandwidth, higher in latency, and expensive because you're charged for your data consumption.

<iframe width="560" height="315" src="//www.youtube.com/embed/K9kUzuNsNIs?list=PL055Epbe6d5YbV14E2hqRFm5FoUvFRmko" frameborder="0"> </iframe>

This is part of the reason the old school dev paradigm of server-side MVC--with data, logic and presentation all happening on a server tier--is showing its age so poorly.  The old click-get-a-page model is predisposed to a high bandwidth, low latency connection with a simple browser on the end node doing nothing more than rendering what the server sent in HTML.  

Appleton proposes that the new development stacks emerging are in part a response to this infrastructure change.  The new model of services in the middle producing tiny chunks of JSON instead of an app server that creates heavy HTML are at least in part a response to the network limitations of mobile, but it's these new stacks that have also enabled native mobile apps and the Internet of Things.

This meshes with what John Gruber writes in ["Rethinking What We Mean by ‘Mobile Web’"](http://daringfireball.net/2014/04/rethinking_what_we_mean_by_mobile_web).  The new mobile app landscape isn't a replacement for the old HTML/CSS/JavaScript world, it's a complement to it.  

The new web is the old web.  REST APIs fit in well with the web model: it will still be a web of linked content, but we'll be delivering more JSON than HTML.


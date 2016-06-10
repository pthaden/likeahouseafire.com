---
layout: post

title: "Creating a Popup Message Box for Oracle JET Forms"

excerpt: "Popup confirmation messages using off-canvas overlays"

tags: [Oracle JET, JavaScript, Dev]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

Sometimes the end-user just wants to be reassured that you heard them. Recently I needed a pop-up confirmation message for an Oracle JET demo, something some call "[toast](http://developer.android.com/guide/topics/ui/notifiers/toasts.html)."

<div class="full"><a href="/images/20160505/toast.gif"><img src="/images/20160505/toast.gif"></a></div>

The inner [Offcanvas Dismissal cookbook entry](http://www.oracle.com/webfolder/technetwork/jet/uiComponents-offcanvas-dismissal.html) provided a good example for what I was looking for, albeit revealing from the side edge instead of the top. 

The idea is that you wrap the whole page component in a `oj-offcanvas-outer-wrapper` and then wrap the pop-up toast message and content you want to overlay in an `oj-offcanvas-inner-wrapper`. Finally the "offcanvas" content is wrapped in a div with a unique ID and one of four `oj-offcanvas-xxxx` edges (`id="innerDrawer"` and `oj-offcanvas-top` in our example code).  This content is hidden by default, but you can reveal it with a call such as `oj.OffcanvasUtils.open(self.innerDrawer)` when you're ready to show the message.

How does it know what to show? Because that `self.innerDrawer` is an object we pass in that points at the CSS selector for our inner drawer (there are [other options for `OffcanvasUtils()` to play with](http://www.oracle.com/webfolder/technetwork/jet/jsdocs/oj.OffcanvasUtils.html#open) in the JSFiddle below, too):

{% highlight js %}

//offcanvas options
  this.innerDrawer =
    {
      "displayMode": "overlay",
      "selector": "#innerDrawer"
    };
{% endhighlight %}


The confirmation message in our example pulls data from the observables in the form, but it could instead show data that returned from an API call initiated by the submit button. The normally-hidden fields in the innerDrawer could be bound to any data in your viewModel, since it only reveals when you choose to call `oj.OffcanvasUtils.open()`. 

<iframe width="100%" height="330" src="//jsfiddle.net/pthaden/a5nz7s6f/embedded/result,js,html,css/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>



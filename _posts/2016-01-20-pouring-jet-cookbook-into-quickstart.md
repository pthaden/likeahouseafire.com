---
layout: post

title: "Pouring JET Cookbook Code into QuickStart"


excerpt: "Dealing with 'Uncaught Error: You cannot apply bindings multiple times to the same element' when pasting sample code directly into the QuickStart template"

tags: [Oracle JET, Dev]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

The [Oracle JET Cookbook](http://www.oracle.com/webfolder/technetwork/jet/uiComponents-dataVisualizations.html) is filled with awesome examples of widgets and patterns. It's great to see a working JET sample and then read the HTML view and JavaScript viewModel code that built that sample. It's even cooler that the Cookbook UI lets you modify the code in its HTML and JS Editors: clicking <kbd>Apply Changes</kbd> will instantly update the sample at the top of your browser window.

But there's a problem with using the Cookbook like a scrapbook to paste code directly into your projects. If you're going to copy-and-paste JET Cookbook code, you want to only take the parts you need.

If you're pasting Cookbook code verbatim into the templates and modules of the QuickStart codebase, you might run into issues with your Require.js modules or may throw strange Knockout binding errors.

For example, let's say you wanted to paste the [code for an ojSlider](http://www.oracle.com/webfolder/technetwork/jet/uiComponents-slider-slider.html) into your QuickStart to see how it would look. Let's assume you're replacing the entire contents of the Home tab in the QuickStart with an ojSlider.

So you copy the HTML Editor content into `home.tmpl.html` and the JS Editor code into `home.js`, replacing everything in the two QuickStart files with the Cookbook code. When you save the changes and run the QuickStart in NetBeans, you'll get a bunch of errors in the Output - Browser Log (or in your browser's console):

{% highlight html %}
Uncaught (in promise) ReferenceError: Unable to process binding "ojComponent: function (){return {
		    component:'ojSlider',max:max,min:min,step:step,value:currentValue} }"
Message: max is not defined (16:26:21:996 | error, javascript)
Uncaught Error: You cannot apply bindings multiple times to the same element. (16:26:22:192 | error, javascript)
{% endhighlight %}

##Uggh, Errors
So what went wrong? Two things:

First, the QuickStart code is setup to use Require.js to load the viewModel code as ["defined" modules](http://requirejs.org/docs/api.html#define). But the Cookbook code is doing an inline Require call. Note the code you pasted in is a `require()` function, but what you need is a `define()` function that `return`s the viewModel that the QuickStart is expecting.

Sure, the Cookbook `require()` code does help us to see what dependencies and code we need to include to get the ojSlider to work, we just need to make sure we're setting our QuickStart code up as a `define`d module and also make sure we return the code as a viewModel function so that it works the way the QuickStart is expecting.

Second, there's another issue with our copied code. The `require()` code has a function in the jQuery `$(document).ready` that calls `ko.applyBindings()` explicitly on the `slider-container`-ID in your view's HTML code. This makes sense in the Cookbook pages: Knockout needs to apply the right bindings to the elements in your HTML code. 

The problem is that our QuickStart scaffolding already has a call to `ko.applyBindings()`. It's in the `main.js` file as part of the `oj.Router.sync()` promise, and it applies bindings to the whole page. So when you copy-paste the Cookbook JS verbatim and include the redundant `applyBindings()`, you get that Knockout error message about how you cannot apply bindings multiple times.

##How Do We Fix It?
So we need to be surgical with our copy-and-paste from the Cookbook into our codebase. We can use the Cookbook as a guide, but we need to understand what code the QuickStart expects and where it should go.

Let's start with the `define()` versus `require()` structure. Looking at the Cookbook code, we see the array of dependencies as the first parameter to the `require()` call. A purist would pluck out the jQuery dependency, because we won't be needing it in our viewModel function. So we're left with the dependencies we need to pass into the existing `define()` call as its first parameter to support the ojSlider:

{% highlight JavaScript %}
define(['ojs/ojcore', 'knockout', 'ojs/ojknockout', 'ojs/ojslider'
   ], ...
{% endhighlight %}

If you were combining elements from multiple Cookbook examples, you'd want to list out all of the dependencies in that array parameter and separate them with commas.

Next, we'll grab the meat of the Cookbook's `SliderModel()` function and pour it into our viewModel's function that gets returned as part of the `define()` block. We won't need to instantiate a new viewModel as in the Cookbook (`main.js` takes care of that with the Router code) and we don't need the jQuery code to apply the bindings (again thanks to the QuickStart's `main.js`).

So after surgery, here's what `home.js` should look like in our QuickStart:

{% highlight JavaScript %}
define(['ojs/ojcore', 'knockout', 'ojs/ojknockout', 'ojs/ojslider'
   ], function(oj, ko) {
   /**
    * The view model for the main content view template
    */
            function mainContentViewModel() {
                var self = this;
                self.max = ko.observable(200);
                self.min = ko.observable(0);
                self.currentValue = ko.observable(100);
                self.step = ko.observable(10);

            }

   return mainContentViewModel;
});
{% endhighlight %}


It turns out the view code from the Cookbook HTML Editor can be used as-is, so we'll just leave it pasted it in to `home.tmpl.html`. That might not be true of every codebase, but the Cookbook HTML works unchanged in the QuickStart scaffolding in this case.

Now when you run your QuickStart from NetBeans, you'll get your slider and there won't be any errors thrown in your console:

<div class="full zoomable"><img src="/images/20160120/working-ojSlider.png"></div>



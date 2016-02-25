---
layout: post

title: "Modules are Your Friends"

excerpt: "Modular development with Oracle JET using ojTrain as an example"

tags: [Oracle JET, JavaScript, Dev]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

The JET Cookbook has a [section on ojModule](http://www.oracle.com/webfolder/technetwork/jet/uiComponents-ojModule-simpleNavigation.html) and how to use it to split your code up into components. Here's an example of breaking up code to make it modular and easier to understand in the context of an Oracle JET Train. 

The [ojTrain](http://www.oracle.com/webfolder/technetwork/jet/uiComponents-train-basic.html) is a great way of leading a user through the steps of a process, letting them know where there are, where they've been, and how much is left to do. In the cookbook sample code, you can see it's possible to stick all of the code for all panels of the train into one viewModel. 

<div class="full zoomable"><img src="/images/20160223/ojTrainCookbook.png"></div>

Here's the code that creates that tab, modified from the Cookbook code to work with the JET QuickStart:

####views/ojTrain.html
{% highlight html %}
<h1>ojTrain Content</h1>
<div id="train-container" >
  <div id="train" class="oj-train-stretch" style="max-width:700px;margin-left:auto;margin-right:auto;"
    data-bind="ojComponent:{
    component: 'ojTrain', 
    selected: currentStepValue, 
    steps: stepArray}"></div>
  <br/>
  <h3 id="currentStepText"
    data-bind="text: 'You are on ' + currentStepValueText()" 
    style="text-align: center">
  </h3>
  <br/>
</div>
{% endhighlight %}

####viewModels/ojTrain.js
{% highlight js %}

define(['ojs/ojcore', 'knockout', 'jquery', 'ojs/ojknockout', 'ojs/ojtrain', 'ojs/ojbutton'],
  function (oj, ko, $)
  {
    function TrainViewModel() {
      var self = this;
      this.currentStepValue = ko.observable('stp1');
      this.stepArray =
          ko.observableArray(
             [{label: 'Step One', id: 'stp1'},
              {label: 'Step Two', id: 'stp2'},
              {label: 'Step Three', id: 'stp3'},
              {label: 'Step Four', id: 'stp4'},
              {label: 'Step Five', id: 'stp5'}]);
      this.currentStepValueText = function () {
        return ($("#train").ojTrain("getStep", this.currentStepValue())).label;
      };
    };
    
    return new TrainViewModel();

  });
{% endhighlight %}


But what happens when our panels get too huge? Or what if each panel should do something different? Combining all of the code for those panels into one view and viewModel could get messy quick, and maintaining the code gets harder and harder if it's all jumbled together in one file.

##ojModules to the rescue

JET's ojModule Binding lets us break up our code into smaller components. With some creative directory naming and computed variables we can even gather everything together so that it's easy to see where the code for our train panels is stored.

Let's start by creating a folder for the modules just to keep things organized. Right-click on viewModels and choose <kbd>New >> Folder</kbd> and create a `myTrain-modules` folder. Do the same for the views folder and use the same `myTrain-modules` name.

<div class="full zoomable"><img src="/images/20160223/newmyTrainFolders.png"></div>

In these folders we need some simple placeholder content for now, but we'll wire it up with a proper viewModel for future use. We could set up a view-only ojModule by passing a `{viewName: 'viewfilename'}` object instead of the moduleName below, but this way we'll have a backing viewModel in case we need it later. 

In the `viewModels/myTrain-modules` folder create a simple `stp1.js` file with a `define()` block that returns a viewModel function, just like you would for a new tab in the QuickStart (which after all uses ojModule!). In the `views/myTrain-modules` folder create a corresponding `stp1.html` file with some HTML for our first panel. Here's some suggested code that will get you going:

####views/myTrain-modules/stp1.html
{% highlight html %}
<h2>step1</h2>
  <div class="oj-flex">
    <div class="oj-panel oj-panel-alt2 oj-panel-shadow-md oj-margin" style="width: 100%">
      <span data-bind="text: message"></span>
    </div>
  </div>
{% endhighlight %}

####viewModels/myTrain-modules/stp1.js
{% highlight js %}
/**
 * step1 module
 */
define(['ojs/ojcore', 'knockout'
], function (oj, ko) {
  /**
   * The view model for the main content view template
   */
  function step1ContentViewModel() {
    var self = this;
    self.message = ko.observable("this is the modular step one panel");
  }
    
  return step1ContentViewModel;
});
{% endhighlight %}

We have five steps in our cookbook sample, so go ahead and duplicate the stp1 files to stp2, stp3, stp4 and stp5 and make edits accordingly to the viewModels' `self.message` observable.

<div class="full zoomable"><img src="/images/20160223/fiveStepFiles.png"></div>

##Call your submodules from your module

Now we need to modify the ojTrain code so that it uses our module content. We'll hard-code stp1 first to make sure all is in place, and after that we'll make the module dynamically load depending on what step in the train we're on.

Since we'll be using ojModule in the view's HTML, we need to add `'ojs/ojmodule'` to the define block for our `viewModels/ojTrain.js` file:
{% highlight js %}
define(['ojs/ojcore', 'knockout', 'jquery', 'ojs/ojknockout', 'ojs/ojtrain', 'ojs/ojbutton', 'ojs/ojmodule'],
    function (oj, ko, $)
    { ...
{% endhighlight %}

With that in place we can modify the `views/ojTrain.html` file to pull in the step 1 file as a module. Replace the `<h3 id="currentStepText"...` code with a `<div>` that is data-bound to our module. Again, we're hardcoding with a string here; we'll make it dependent on the train's current step in the next section:
{% highlight html %}
...
<br/>
<div data-bind="ojModule: 'myTrain-modules/stp1'"></div>
<br/>
...
{% endhighlight %}

So now you should see new content under the train, but there's that problem that the content is the same no matter which step of the train we're on. We need to make our ojModule dependent upon the ojTrain's current state, and for that we need a computed observable.

<div class="full zoomable"><img src="/images/20160223/almostThereNoDynamicSwitchingYet.png"></div>

##Make it change based on the current train step
Instead of that hard-coded string in the ojModule `data-bind`, we can have an observable in the viewModel tell us the path to the module. If we make that observable a [pureComputed](http://knockoutjs.com/documentation/computed-pure.html), it will always have the right value for the ojModule call even as we click through the steps of our train. 

The cookbook code for the Train is already storing its `selected:` state in a `currentStepValue` observable, and we conveniently named our module filenames with the same naming convention as the currentStepValue. We just need to prepend the module directory and put it in a function that will return the live value of the ojTrain's current step.


{% highlight js %}
...
this.modulePath = ko.pureComputed(
  function () {
    return ('myTrain-module/' + self.currentStepValue());
  }
);
{% endhighlight %}

Last, update the `views/ojTrain.html` binding for ojModule to point to our ModulePath pureComputed observable instead of the hard-coded string above.

####final views/ojTrain.html
{% highlight html %}
<h1>ojTrain Content</h1>
<div id="train-container" >
  <div id="train" class="oj-train-stretch" style="max-width:700px;margin-left:auto;margin-right:auto;"
    data-bind="ojComponent:{
    component: 'ojTrain', 
    selected: currentStepValue, 
    steps: stepArray}"></div>
  <br/>
  <div data-bind="ojModule: modulePath"></div>
  <br/>
</div>
{% endhighlight %}


####final viewModels/ojTrain.js
{% highlight js %}
define(['ojs/ojcore', 'knockout', 'jquery', 'ojs/ojknockout', 'ojs/ojtrain', 'ojs/ojbutton', 'ojs/ojmodule'],
  function (oj, ko, $)
  {
    function TrainViewModel() {
      var self = this;
      this.currentStepValue = ko.observable('stp1');
      this.stepArray =
          ko.observableArray(
             [{label: 'Step One', id: 'stp1'},
              {label: 'Step Two', id: 'stp2'},
              {label: 'Step Three', id: 'stp3'},
              {label: 'Step Four', id: 'stp4'},
              {label: 'Step Five', id: 'stp5'}]);

      this.modulePath = ko.pureComputed(
        function () {
          return ('myTrain-modules/' + self.currentStepValue());
        }
      );
    };

    return new TrainViewModel();

  });
{% endhighlight %}

##Finished product

When you click the steps in the train it updates the selected currentStepValue, which creates a new function for the modulePath, which is used to load the correct ojModule. Click another step in the train, and the beautiful process repeats.

<div class="full zoomable"><img src="/images/20160223/modularTrain.gif"></div>


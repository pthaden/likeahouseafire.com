---
layout: post

title: "Passing Data into ojModules"

excerpt: "Creating submodules to iterate over an array and using the params: option to reference data from both a parent ojModule and its children. "

tags: [Oracle JET, JavaScript, Dev]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

Geertjan Wielenga recently published [a series of articles on inter-module communication](https://blogs.oracle.com/geertjan/entry/intermodular_communication_in_oracle_jet2) with three great ways to get modules talking to each other, or at least sharing data. Here is a different trick I learned from [Jim Marion](http://jjmpsj.blogspot.com/), originally in the context of passing data to Knockout Components but reapplied here to Oracle JET modules.

ojModules [can be used to tidy up your codebase](http://likeahouseafire.com/2016/01/23/modules-are-your-friends/), but they're also useful for creating reusable blocks of code that you might iterate over to produce many copies of the same component, each with its own backing data. 

Using the [$params option for ojModule](http://www.oracle.com/webfolder/technetwork/jet/jsdocs/ojModule.html#Options) we can pass data objects into the called submodule to be used and tweaked in the context of that module's viewModel. Since JavaScript objects are pass-by-copy-of-reference, this means anything we add or change to the passed in $params object will still be there if we try to access it from the parent module.  This is because they're pointing at the same thing in memory.

So for example lets say we have a array of objects we want to iterate over and create index cards out of, [a la the WorkBetter demo](http://www.oracle.com/webfolder/technetwork/jet/public_samples/WorkBetter/public_html/index.html?root=people). Instead of creating all of the code for the card itself in the page's view, we could instead create a module out of the cards. Then if we ever need to use cards again in a different part of our app, we already have the code set up as a module and can just reuse it.

<div class="full zoomable"><img src="/images/20160225/each-card-a-module.png"></div>

## Setting up the parent ojModule
In our example, the module that the router calls when we click on the Employees tab is very simple. Think of it as the parent view for all of the children cards we'll iterate over.

####employeeCards.html view
{% highlight html %}
<h1>Card Modules Content</h1>
<h2>computed cards selected: <span data-bind="text: computedCount"></span></h2>

<!-- ko foreach: allPeople()  -->
    <div data-bind="ojModule: {name: 'cards-modules/card',
                        params: $data }"></div>
<!-- /ko -->
{% endhighlight %}

Notice that there's hardly any HTML in the parent. Basically just a Knockout `foreach` that iterates over an `allPeople` array. Notice also that we're calling a module in a relative subdirectory named `cards-modules/card` and that each time we loop we're passing in the `$data` binding as `params` for the current iteration of the foreach...more on that later.

Here's the backing viewModel for our parent view. Since this is demo data, the `allPeople` observable array is hard-coded but it could just as well be populated by a data retrieval function. There's also a computed observable that returns a count of all cards in the data that have an `isSelected` member set to `true`.

####employeeCards.js viewModel
{% highlight js %}
define(['ojs/ojcore', 'knockout'
], function (oj, ko) {
  function ViewModel() {
    var self = this;

    self.allPeople = ko.observableArray([]);
    self.allPeople([{
            "empId": 205,
            "firstName": "Shelley",
            "lastName": "Higgins",
            "title": "Accounting Director",
            "hireDate": "2010-01-01T00:00:00Z",
            "rating": 5,
            "potential": 3,
            "deptName": "Accounting"
        }, {
            "empId": 118,
            "firstName": "Guy",
            "lastName": "Himuro",
            "title": "Purchasing Clerk",
            "hireDate": "2009-03-01T00:00:00Z",
            "compRatio": 63,
            "rating": 4,
            "potential": 4,
            "deptName": "Purchasing"
        }, {
            "empId": 103,
            "firstName": "Alexander",
            "lastName": "Hunold",
            "title": "IT Director",
            "hireDate": "2006-07-14T00:00:00Z",
            "rating": 2,
            "potential": 2,
            "deptName": "IT"
        }, {
            "empId": 175,
            "firstName": "Arthur",
            "lastName": "Hutton",
            "title": "Public Relations Reprentative",
            "hireDate": "2008-02-14T00:00:00Z",
            "rating": 2,
            "potential": 4,
            "deptName": "Public Relations"
        }]);
    
    // calculate number of selected cards when allPeople receives valueHasMutated()
    self.computedCount = ko.computed(function(){
        //filter array for only the elements that have isSelected == true, then count the length of that filtered array
        var countNewvalue = self.allPeople().filter(function (c) {
                // must check if the object even has an isSelected member, because it doesn't on the first pass
                if (c.isSelected){
                    return c.isSelected();
                }
            }).length;
        return countNewvalue;
    });
  }

  return ViewModel;
});

{% endhighlight %}

But wait: looking at the hard-coded data for `allPeople`, there is no member for `isSelected`&mdash;at least not yet. That's because we are going to add it inside of the `cards-modules/card` child module as the result of a click handler. But the cool thing is that we'll be able to access that data here in the parent's viewModel because of the way we pass the `$data` object.

## Setting up the child ojModule
The submodule's HTML is unabashedly copied from the WorkBetter sample app, but it is only the index card code. Each time we iterate over the parent's `allPeople` array, we'll get one copy of this card HTML. 

####cards-modules/card.html view
{% highlight html %}
<div class="oj-col oj-sm-12 oj-md-6 oj-lg-4 oj-xl-3">
  <!-- click handler toggleSelected() receives $parents from binding context  -->
  <div class="oj-panel oj-panel-alt1" style="height: 226px; margin: 5px 0px 10px 0px;" data-bind="click: function() {toggleSelected($parents)}">

    <div class="oj-row">
      <div class="oj-col oj-sm-4">
        <img class="demo-circular demo-employee-photo" data-bind="attr: {src: getPhoto(cardData.empId)}"/>
      </div>
      <div class="oj-col oj-sm-8">
        <div class="demo-employee-name" data-bind="text: cardData.firstName+ ' ' + cardData.lastName"></div>
        <div class="demo-employee-title" data-bind="text: cardData.title"></div>
        <div class="demo-employee-dept" data-bind="text: cardData.deptName"></div>
      </div>
    </div>

    <div class="oj-row">
      <div class="oj-col oj-sm-4">
        <div class="demo-employee-tenure" data-bind="text: getTenure(cardData)"></div>
        <div class="demo-employee-tenure-label">Tenure</div>
      </div>
      <div class="oj-col oj-sm-4">
        <div class="demo-employee-perf" data-bind="text:cardData.rating, style: {color: cardData.rating < 3 ? '#e95b54' : '#309fdb'}"></div>
        <div class="demo-employee-perf-label">Rating</div>
      </div>
      <div class="oj-col oj-sm-4">
        <div class="demo-employee-perf" data-bind="text:cardData.potential, style: {color: cardData.potential < 3 ? '#e95b54' : '#309fdb'}"></div>
        <div class="demo-employee-perf-label">Potential</div>
      </div>
    </div>

    <div class="oj-row">
        
      <!-- span to indicate whether card has been clicked into isSelected state -->
      <span style="float:left; color: green; font-size: 36px; margin-top: 20px" data-bind="visible: cardData.isSelected" class="fa fa-lg fa-check-circle-o">is selected</span>

      <span style="float: right;">
        <a data-bind="attr:{href: '#'}" role="img" title="Send this employee an email" class="demo-employee-email-icon"></a>
        <a data-bind="click: function(data, event){}, clickBubble: false" role="img" title="View this employees team members" class="demo-employee-org-icon"></a>
      </span>
    </div>
  </div>
</div>
{% endhighlight %}

To the WorkBetter code I added a span of green text that only shows when the card's viewModel has an `isSelected` attribute set to `true`. We'll see this is set by a `toggleSelected()` function in the viewModel.

Also see how all of the fields' `text` data is being bound to members of a `cardData` object in the backing viewModel. This is actually the data that got passed from the parent view when we called ojModule with the `params: $data` option. We unpack it in the viewModel code below.

There is one other difference from the WorkBetter code: we've added the click handler for that `toggleSelected()` to the entire card panel's div. But see how it's not a normal `data-bind` reference? Instead, we're calling anonymous function to pass in the context binding for `$parents` to our `toggleSelected` click handler. We need this `$parents` context inside of the viewModel, and I'll explain why next.

####cards-modules/card.js viewModel
{% highlight js %}
define(['ojs/ojcore', 'knockout'
], function (oj, ko) {

  function cardContentViewModel($params) {
    var self = this;
    // assign the passed-in $params to a viewModel variable
    self.cardData = $params;
    
    // add a flag to the card to track "selected" state
    self.cardData.isSelected = ko.observable(false);

    // click handler for toggling a card's selected state
    // gets $parents passed in from view's binding 
    self.toggleSelected = function ($parents) {
      self.cardData.isSelected(!self.cardData.isSelected());
     
      // we need to tell the grandparent that we changed the data inside the array
      // since the grandparent's allPeople observableArray won't fire subscriptions otherwise
      $parents[1].allPeople.valueHasMutated();
    };

    // functions stolen from WorkBetter for card data calculations
    self.getPhoto = function (empId) {
      var src;
      if (empId < 188) {
          src = 'css/images/people/' + empId + '.png';
      } else {
          src = 'css/images/people/nopic.png';
      }
      return src;
    };

    self.getTenure = function (emp) {
      var now = new Date().getFullYear();
      var hired = new Date(emp.hireDate).getFullYear();
      var diff = now - hired;
      return diff;
    };
  }

  return cardContentViewModel;
});
{% endhighlight %}

This looks like a normal viewModel but for a few differences. One is that we're expecting a `$params` parameter when our viewModel function is called. This will be the current record from the `allPeople` array when the parent calls our ojModule. We immediately assign `$params` to a local viewModel variable (`self.cardData`) so that we can access it via normal data binding in the view.

But again: because JavaScript is passing in a copy of the pointer to the object that was this iteration of the `allPeople` array, then if we modify the data it will be changing the same object that the parent owns. This means we can make a change to the state of the object here in the submodule, but use that same data up in the parent module.

That's what we do when we add an `isSelected` member to the data object. This is a locally-maintained flag that holds the 'has it been clicked' status for this card. The selected status is toggled by the `toggleSelected()` function.

But that toggle function takes a parameter of its own: the `$parents` binding from the card view. We need this in order to reach up to the parent's `allPeople` observable array and tell it that we've mutated the values of its members, otherwise the computed observable `computedCount` will never fire. Why not? 

Because we're not really modifying the observable array `allPeople`...we're modifying a member of that array. The array still has the same number of items and thus the array itself doesn't seem changed, especially to the parent viewModel. We've only modified one of the members of one of the array's items, and so the computedObservable subscription never fires. But we can wake it up by explicitly telling it to recalculate by means of the `$parents[1].allPeople.valueHasMutated();` call.

(aside:  We've been calling it the "parent" viewModel, so why do we need to reach up to the *grand*parent with `$parents[1]` instead of the parent with `$parents[0]`?  It's because of the `foreach` in the parent module's view: it introduces another layer when it iterates over the array before calling our submodule.)

##Taking it for a spin
With all the pieces now in place we can show data added and modifed in the submodule yet also accessible from the parent viewModel. 

When we click on a card, it locally toggles its selected state and the green "is selected" text is revealed. Meanwhile up at the parent the computed observable fires and the parent counts up how many array items are in isSelected state and displays that number via a binding in the parent's view.

<div class="full zoomable"><img src="/images/20160225/clicking_isSelected.gif"></div>

With this same `params` passing technique, a single data object could be passed around to multiple modules in a view, all of them modifying the same object in memory. 

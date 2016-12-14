---
layout: post

title: "Return a Promise from handleActivated for Data-bound ojModules"

excerpt: "Use ojModule's handleActivated to fetch and return your data, postponing the DOM until the promise resolves"

tags: [Oracle JET, JavaScript, Dev]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

In a [recent post we talked up the ojModule life cycle](http://likeahouseafire.com/2016/12/09/ojmodule-lifecycle-functions/) and the function hooks that run at various points while a module is being brought in or dismissed. One of those hooks can be used to load data from an external source and have the module wait to build its view until the data has arrived.

## ojModule life cycle recap

The [ojModule component](http://www.oracle.com/webfolder/technetwork/jet/jetCookbook.html?component=ojModule&demo=simpleNavigation) is a foundational part of Oracle JET that works hand-in-hand with the [ojRouter](http://www.oracle.com/webfolder/technetwork/jet/jetCookbook.html?component=router&demo=simple).  ojModules make up the "pages" in single-page apps that the Yeoman templates generate like [NavDrawer](http://www.oracle.com/webfolder/technetwork/jet/public_samples/JET-Template-Web-NavDrawer/public_html/index.html) and [NavBar](http://www.oracle.com/webfolder/technetwork/jet/public_samples/JET-Template-Web-NavBar/public_html/index.html), or they can even be subcomponents [like the panels of an ojTrain's](http://likeahouseafire.com/2016/01/23/modules-are-your-friends/) "in-page" component.

Thanks to the [module life cycle](https://docs.oracle.com/middleware/jet202/jet/developer/GUID-ABB82BD1-9B65-44D2-AE43-81A0A3A44453.htm#JETDG-GUID-ABB82BD1-9B65-44D2-AE43-81A0A3A44453), ojModules have a neat trick if you want to load them up with fetched data. Normally a call to a data endpoint won't return in time before the DOM starts generating, or at least you can't rely on it to do so since it's an async call. If you have observables waiting for the data you have to play tricks:  like [setting a flag to hide the data-bound components](http://jsjim.blogspot.com/2016/04/help-im-using-asynchronous-javascript.html) in the HTML view until the data is ready; or, doing all the viewModel work [in a callback function that follows after your data is ready](https://blogs.oracle.com/geertjan/entry/simple_json_and_oracle_jet); or, [calling valueHasMutated() on an observableArray to wake it up](https://community.oracle.com/message/13781951#13781951) and have it refresh the HTML after the data arrives.

## handleActivated to the rescue

When you use the Yeoman generator to build a NavBar or NavDrawer app, you'll see this code in the generated viewModels:

{% highlight js %}
/**
* Optional ViewModel method invoked when this ViewModel is about to be
* used for the View transition.  The application can put data fetch logic
* here that can return a Promise which will delay the handleAttached function
* call below until the Promise is resolved.
* @param {Object} info - An object with the following key-value pairs:
* @param {Node} info.element - DOM element or where the binding is attached. This may be a 'virtual' element (comment node).
* @param {Function} info.valueAccessor - The binding's value accessor.
* @return {Promise|undefined} - If the callback returns a Promise, the next phase (attaching DOM) will be delayed until
* the promise is resolved
*/
self.handleActivated = function(info) {
// Implement if needed
};
{% endhighlight %}

It's this line in the comment that interests us: *"put data fetch logic here that can return a Promise which will delay the handleAttached function call below until the Promise is resolved."* So if we want to hold off on rendering the DOM until our data arrives, we can use this hook and pause the rendering until we have data.

When would we want to do that? Not necessarily for table data: there's no need to delay the DOM since we can data-bind to a collection and the table will show a "fetching data" message until the data comes back. Most of the other data-bound JET components also gracefully take data updates and dynamically re-render just fine.

But there are other times when you *do* want to delay the DOM. Perhaps you don't want users staring at unpopulated form data fields.  You might need to manipulate the returned data before rendering any of it in the browser, or perhaps need to have the data in-hand to determine the layout of the module. Another example: on a form-centric project we had combo boxes with [options binding](http://www.oracle.com/webfolder/technetwork/jet/jetCookbook.html?component=combobox&demo=optionsBinding) that got their valid lists-of-values from REST APIs, and we didn't want the combo boxes to be empty during that brief moment while waiting for the ajax calls to come back from the server.

## Fixing a Filmstrip with remote data

Or how about a JET component that won't render right without the proper data in-place before the DOM is built? Take the [Film Strip](http://www.oracle.com/webfolder/technetwork/jet/jetCookbook.html?component=filmStrip&demo=filmStripNavArrows) as an example. If the data that backs the `<!-- ko foreach: -->` loop isn't available when the DOM is rendering, the filmstrip will be laid out as if there were no items in the strip. When the data arrives a few hundred milliseconds later, the filmstrip will then try to jam them all in there and the layout will look horrible:

<div class="full zoomable"><img src="/images/20161212/jammed-up-after-data-arrived-too-late.png"></div>

But if we just delay the DOM rendering with a `Promise` and the `handleActivated` function, we give our filmstrip a chance to see data exists in the observableArray before the view iterates over the `forEach`, and thus it renders properly:

<div class="full zoomable"><img src="/images/20161212/ahh-thats-better-waiting-on-a-promise-to-render.png"></div>

Here's the trick: we put the call to retrieve the data from the remote REST endpoint into the `handleActivated` function, but we also wrap it in a `new Promise()` and return the whole kit and caboodle. The ojModule then knows to delay the DOM rendering until the promise resolves, and so the ojFilmStrip waits for our data.

Here's the code for the viewModel and view:

#### filmstrip.js
{% highlight js %}

define(['ojs/ojcore', 'knockout', 'jquery', 
         'ojs/ojknockout', 'ojs/ojfilmstrip'
], function (oj, ko, $) {

  function FilmStripViewModel() {

    var self = this;

    self.users = ko.observableArray();      
      
/*  This won't work, because the data comes back too late for the view to pass
//   it to the ojFilmStrip it in the ko.forEach
//    $.ajax({
//        url: 'https://jsonplaceholder.typicode.com/users',
//        method: 'GET'
//      }).then(function(data) {
//  
//          data.forEach(function(user){
//          self.users.push(user);
//        });
//
//    });
*/
   
  
    self.handleActivated = function(info) {
    
      return new Promise(function(resolve, reject) {

/*    This works great, because the ojModule waits to build the view until
//     the promise resolves */
        $.ajax({
          url: 'https://jsonplaceholder.typicode.com/users',
          method: 'GET'
          }).then(function(data) {
            data.forEach(function(user){
              self.users.push(user);
            });

            resolve();
        });

      });
        
    };    

  }

  return FilmStripViewModel;
});


{% endhighlight %}

#### filmstrip.html
{% highlight js %}


<h1>Filmstrip</h1>
<div id="filmstrip-navarrows-example" class="oj-flex oj-sm-justify-content-center">

  <div id="filmStripDiv" class="oj-panel oj-flex-item" style="margin: 20px; max-width: 450px">
    <div id="filmStrip" 
         aria-label="Set of users from REST API" 
         data-bind="ojComponent: {
           component: 'ojFilmStrip', 
           arrowPlacement: 'adjacent', 
           arrowVisibility: 'auto' }">
      <!-- ko foreach: users -->
      <div class="oj-panel oj-panel-alt2 demo-filmstrip-item" >
        <span data-bind="text: name"></span>
      </div>
      <!-- /ko -->
    </div><!-- end filmStrip -->
  </div> <!-- end filmStripDiv -->

</div>

{% endhighlight %}






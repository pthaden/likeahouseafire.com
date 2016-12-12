---
layout: post

title: "Leveraging the ojModule Life&nbsp;Cycle Functions"

excerpt: "Understanding the four horsemen of the ojModule templates: handleActivated, handleAttached, handleBindingsApplied, and handleDetached"

tags: [JET, Dev, JavaScript]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

Srikantha Bhadrasetty asked [a good question on an old blog post about using JET Cookbook code](http://likeahouseafire.com/2016/01/20/pouring-jet-cookbook-into-quickstart/#comment-3040014952): How do you reuse the more complicated recipes that lean on tools like jQuery's DOMReady function?

In that [particular cookbook entry](http://www.oracle.com/webfolder/technetwork/jet/jetCookbook.html?component=tree&demo=treeDnD) there is a jQuery function that waits for the DOM to be built before registering the two `treeModels` and adding drag and drop event listeners to three pink `div`s. If we refactor all the cookbook code into one of the QuickStart's RequireJS-based ojModules but leave that code in the jQuery DOMReady function, it will fire at the wrong time and the ojModule won't load. Your console will show an error like:

{% highlight bash %}
Uncaught (in promise) ReferenceError: Unable to process binding "ojComponent: {
           component:'ojTree', ..."
{% endhighlight %}

[The solution](https://gist.github.com/pthaden/78825aca766280ea18c7e53d674c19f7) is to move the code out of the jQuery DOMReady `$(function(){ ... })` and into appropriate ojModule life cycle functions, like `self.handleAttached` and `self.handleBindingsApplied`.

## What is the ojModule ViewModel Life Cycle?

When you use ojModule to load your views and viewModels (as the QuickStart is configured to do with the help of the Router), you can hook into listeners that fire at defined spots in the ojModule's life cycle. These are documented in the [JET Developers Guide](https://docs.oracle.com/middleware/jet202/jet/developer/GUID-ABB82BD1-9B65-44D2-AE43-81A0A3A44453.htm#JETDG-GUID-ABB82BD1-9B65-44D2-AE43-81A0A3A44453) and in the [ojModule JSdocs](http://docs.oracle.com/middleware/jet220/jet/reference-jet/ojModule.html).

If you've used the Yeoman generator to build a NavDrawer or NavBar template project, you've probably already seen the placeholders for some of these life cycle functions in the template code, with a *ton* of comments explaining their use and an empty `// Implement if needed` function body for each:


<div class="full zoomable"><img src="/images/20161209/implement_if_needed.png"></div>

If you don't need them in your viewModel code, it's safe to delete these function stubs. When ojModule comes to the respective points in time to call the appropriate life cycle function, it just keeps going if the callback is null or if it's an empty function like in the template code.

But if we have our own code that needs to run at a specific point, such as before or after the corresponding view's DOM gets built in the browser or even before or after the Knockout bindings fire, we can leverage these specially-named functions and put our code inside of them.

For fun, I threw some [Chrome timeline markers](https://developers.google.com/web/tools/chrome-devtools/console/track-executions#marking-the-timeline) into my code like so:

{% highlight js %}
function lifecycleContentViewModel() {
  console.timeStamp("viewModel Started");

  var self = this;

  self.handleActivated = function(info) {
    console.timeStamp("handleActivated Started");
    // Implement if needed
  }; 
  ...
{% endhighlight %}

This makes little orange circles that correspond to timestamp markers at the top of the timeline in the Developer Tools. You can see in the graphic below that there's a dependable order the life cycle functions fire in and that it happens in the context of loading the current ojModule, [long before a jQuery DOMReady or window.load event would fire](https://www.kirupa.com/html5/running_your_code_at_the_right_time.htm) (this image is a composite of me hovering the mouse over each circle, one at a time):


<div class="full zoomable"><img src="/images/20161209/lifecycle_timeline.png"></div>

## Which one to use?

So this brings us back to the [cookbook code for the drag-and-drop example](http://www.oracle.com/webfolder/technetwork/jet/jetCookbook.html?component=tree&demo=treeDnD). How do we decide what recipe code goes into which life cycle function?

By identifying the timing points for when our code needs to run, we can pick the right life cycle function to stick it into. The comments in the Yeoman-generated template viewModels and the documentation help you decide which to pick, and here's some thoughts on the four of them:

<div class="full">
<style>
    td{padding-left: 1em ;
  text-indent: -1em ;
}
</style>
<table>
  <thead>
    <tr>
      <th width="27%"><strong>function</strong></th>
      <th width="30%"><strong>when it fires</strong></th>
      <th width="40%"><strong>what it’s good for</strong></th>
    </tr>
  </thead>
  <tbody style="  text-align: left; vertical-align: text-top;">
    <tr>
      <td>handleActivated</td>
      <td>right before view is built</td>
      <td>fetching data needed for the view's elements</td>
    </tr>
    <tr>
      <td>handleAttached</td>
      <td>right after view is attached to the DOM</td>
      <td>modifying HTML elements via JavaScript; any time you might want to use a $('selector') and be able to trust the element is there</td>
    </tr>
    <tr>
      <td>handleBindingsApplied</td>
      <td>after the ko.applyBindings has run against the view </td>
      <td>programatically work with the ojComponents post-binding </td>
    </tr>
    <tr>
      <td>handleDetached</td>
      <td>after view is detached from the DOM</td>
      <td>cleaning up </td>
    </tr>
  </tbody>
</table>
</div>

There are a few additional functions listed in the docs that fire at other points in the ojModule life cycle, but these are the four that are stubbed out in the NavDrawer and NavBar templates.

Going back to the problem code wrapped in the jQuery DOMReady function from the previously linked drag-and-drop cookbook recipe. I'll annotate it with comments based on what we need to do:

{% highlight js %}
$(function() {
    
/* these new models need to be created after the DOM is created but before the bindings are applied, so that they're in place when the KO bindings fire  */
  var vmTree1  = new treeModel("tree1") ;
  var vmTree2  = new treeModel("tree2") ;
/* */
    
/* these cookbook lines need to be removed, because oj.Router.sync() in main.js will apply the bindings and we can't do it twice */
  ko.applyBindings(vmTree1, document.getElementById('tree1'));
  ko.applyBindings(vmTree2, document.getElementById('tree2'));
/* */
    
/* these event handlers can't be added until the divs are in the DOM and the ojComponents are ready, so also must run after view is attached */
  for (var i = 1; i <= 3; i++)
  {
    var div = document.getElementById("dragdiv" + i) ;

    div.addEventListener("dragstart", dragDivStart, false) ;
    div.addEventListener("dragend",   dragDivEnd,   false) ;
  }   
/* */

});
{% endhighlight %}



For our code, we need the `self.handleAttached` to setup the models that build the two trees. Looking at the `treeModel()` function, it's obvious the DOM needs to already be in place so that the constructor can build the ojTree structure on top of the two existing `tree1` and `tree2` divs.

We pick the `self.handleBindingsApplied` function to run the `for()` loop that registers the dragstart and dragend events since in the original code the `ojTree`s are already bound, although they technically could run earlier in the `handleAttached` function because they only rely on having the `div`s present in the DOM.

Again, have a look at the full view and viewModel code for a working example of the drag-and-drop cookbook recipe re-purposed for the NavDrawer ojModule pattern:

<script src="https://gist.github.com/pthaden/78825aca766280ea18c7e53d674c19f7.js"></script>


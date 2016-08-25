---
layout: post

title: "Swapping in External Data Sources for the JET Cookbook Examples"

excerpt: "How to point to your own data files on GitHub Gists when modifying the Oracle JET Cookbook samples"

tags: [Oracle JET, Dev]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

The new layout and format for the [Cookbook for Oracle JET 2.1.0](http://www.oracle.com/webfolder/technetwork/jet/jetCookbook.html) is *gorgeous*. As before, it's loaded with lots of working sample code, which you can change in-browser and see the results immediately.

But what if you're working with a data collection and you want to test the cookbook recipe with some custom data? You're only able to edit the HTML view and JS viewModel of a cookbook entry, not any of the supporting files.

This was a fun exercise to see if we could host sample data files on Github. Plus I learned something about the way JET's Cookbook uses tools like `MockPagingRESTServer()` and how to overcome caching issues. 

If you look at the JS tab of a cookbook sample that retrieves data, you'll see that it's just fetching a URL with `$getJSON()`. For example, in the [List View using oj.Collection sample](http://www.oracle.com/webfolder/technetwork/jet/jetCookbook.html?component=listView&demo=collectionListView), it's a relative URL to a tweets.json file on the Cookbook file system that you can path out to [retrieve the raw data with this link](http://www.oracle.com/webfolder/technetwork/jet/cookbook/dataCollections/listView/collectionListView/tweets.json).

<div class="full zoomable"><img src="/images/20160824/listViewgetJSON.png"></div>

So we can change the path of the `$getJSON()` call in the Cookbook to retrieve `tweets.json` from a different location. And we have the format and content of the original JSON file which we could upload to another server after modifying it. But where can we host our modified JSON file?

##How about as a GitHub Gist?

We can create a public Gist file, upload our JSON there and then point to the gist's raw content from the Cookbook's JS tab. Really, our modified JSON could be hosted anywhere we can stick a file that our browser can find, whether on the internet, our intranet or even on our localhost. Or we could point the cookbook code at an existing endpoint, including fake ones like [JSON Placeholder](http://jsonplaceholder.typicode.com/).

But Github gives us an option to stick our own custom JSON data file in a public place.  We point the Cookbook code at the file itself using a "raw" URL link (as opposed to the fancy Github webpage with all of its chrome). And, Github is nice enough to set the CORS headers so we don't run into cross-origin issues loading data from a different server than the Cookbook host.

So download the tweets.json file and modify it to match your needs. Now login to Github and click the Gist link at the top of the page, or instead navigate straight to [gist.github.com](https://gist.github.com). Click `New gist`, name the gist and paste your JSON into the editor, and then click `Create public gist`.

After the gist is saved, you'll see a `Raw` button you can click to get a URL like this one:

{% highlight bash %}
https://gist.githubusercontent.com/pthaden/ab957da62672b4a305da5b653615830c/raw/d2004aa6793e240cc6565786a71c4fbe6a844c55/tweets.json

{% endhighlight %}

We'll replace the URL in the `$.getJSON()` call inside the Cookbook recipe with this gist URL. 

##But it doesn't work!
If you've jumped ahead and already pressed `Apply Changes` after pasting in your URL to the [ojCollection List View](http://www.oracle.com/webfolder/technetwork/jet/jetCookbook.html?component=listView&demo=collectionListView) example, you may notice that nothing updated in the UI. We chose this recipe on purpose, because it leverages the `MockPagingRESTServer`.

This is a chunk of JavaScript that some of the Cookbook entries use to simulate a proper REST endpoint. It fools the recipe's ojCollection into thinking there's a real server out there that can get more data if needed at a mock URL such as ` /context-root/ojet/Tweets?limit=15&offset=0&totalResults=true`. 

The problem is that the MockServers are cached in memory when you click `Apply Changes` in the Cookbook. So the browser does indeed go out to getJSON() your new data from Github, but the ojCollection that is backing the HTML List View hits the same MockServer URL as before and it uses the cached data.

I haven't figured a way to flush out the MockServer's data, but an approach that did work is to rename the URL it builds internally by editing the object passed to the `new MockPagingRESTServer()` constructor. In our List View recipe, we'll change the "Tweets" entries to something like "myTweets" before clicking Apply Changes. Here's the changes I made to the JS tab of the recipe:


{% highlight javascript %}

require(['ojs/ojcore', 'knockout', 'jquery', 'ojs/ojknockout', 'promise', 'ojs/ojlistview', 'ojs/ojcollectiontabledatasource', 'ojs/ojmodel', 'mockjax', 'mockpagingrest'],
function(oj, ko, $)
{     
  $(document).ready(
    function() 
    {
      $.getJSON("https://gist.githubusercontent.com/pthaden/ab957da62672b4a305da5b653615830c/raw/d2004aa6793e240cc6565786a71c4fbe6a844c55/tweets.json",
        function (data) 
        {
          // responseTime is only added so that the activity indicator is more noticeable
          var server = new MockPagingRESTServer({"myTweets": data}, {collProp:"myTweets", id:"source", responseTime:1000});

          var model = oj.Model.extend({
                idAttribute: 'source'
          });

          var collection = new oj.Collection(null, {
                url: server.getURL(),
                fetchSize: 15,
                model: model
          });

          ko.applyBindings({
              dataSource: new oj.CollectionTableDataSource(collection)
              }, document.getElementById('listview'));
        });      
      }
  );
});	

{% endhighlight %}

And it works now, with custom data from my gist!


<div class="full zoomable"><img src="/images/20160824/workingrecipe.png"></div>





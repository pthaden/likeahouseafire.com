---
layout: post

title: Applescript Workflow Fails When No Safari Window Open
cover_image: automator_javascript_error.jpg

excerpt: "The action “Run AppleScript” encountered an error...."

tags: [Dev, REST]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

Found an error with an AppleScript Workflow that I've had installed to my ~/Library/Services for a long time, but only now noticed that it doesn't work when there are no Safari windows open.

My little workflow is something I've hacked from others' work for automating an internal employee directory that doesn't have any APIs but does let you hit a search URL which executes and returns in a browser window.  If you want to look up someone mentioned in an email or other document you simply highlight the text, right click and choose the workflow from the Services context menu.

Now I'll admit that I keep a whole lot of windows open on my desktop.  So it's pretty rare that I've closed every window in Safari.  Even Safari itself will open a blank window of Top Sites when it starts up if you had all windows closed in the last session, and thus my workflow has never broken down until I discovered this unique (for me) use case:  Safari is running, but all windows are closed.  Only then would I get a non-descript message from  Automator:

`The action “Run AppleScript” encountered an error`

Thanks to [this StackOverflow answer](http://stackoverflow.com/a/9082574) I learned that the JavaScript in my workflow needs a window to do its work in.  When there's no window open in Safari there's no destination for my JavaScript.  I found that `reopen` works much better for me, and once I replaced the `activate` code my little service works fine.

Here's the code with my little regex that extracts out only an email address if it finds one in the text you highlight, but otherwise just submits the text you highlighted:

{% highlight applescript %}
on run {inText}
	tell application "Safari"
		reopen -- unlike activate, this opens a window
		set searchTerm to inText
		set xNow to do JavaScript " 
		function extractEmails ( text ){
			return text.match(/([a-zA-Z0-9._-]+@[a-zA-Z0-9._-]+\\.[a-zA-Z0-9._-]+)/gi);
		};
		var x,
		z,
		i;
		x = '" & searchTerm & "';
		if (x) {
		if (extractEmails(x)) {
			x = (extractEmails(x));
			};
		     z = 1;    
		for (i = 0; i < 1; ++i) {
			sndT = 'https://people.us.oracle.com/pls/oracle/f?p=8000:1::::RP,RIR:P1_SEARCH,P1_SEARCH_TYPE:' + x + ',People';
			popup_window = window.open(sndT);
		}


		};
		" in document 1
	end tell
end run
{% endhighlight %}


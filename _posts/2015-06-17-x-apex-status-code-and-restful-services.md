---
layout: post

title: X-APEX-STATUS-CODE and APEX RESTful Services


excerpt: "Sending back the HTTP Status Code you want to send"

tags: [APEX, REST]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

When using the [APEX RESTful Services](http://www.oracle.com/technetwork/developer-tools/rest-data-services/documentation/listener-dev-guide-1979546.html) to develop endpoints, you can get pretty funky and create entire PL/SQL handlers for your RESTful endpoints. 

This is especially useful for POSTs because you can pluck off inbound HTTP headers and stuff them into parameters in the RESTful handler, and then reference those parameters as bind variables in your PL/SQL. You can also send back bind variables in the outbound response, either as headers or in the body.

We ran into an issue where we were using the header values to insert into a table. The problem came when a row already existed in the table: the SQL insert would fail, but the RESTful handler didn't handle it gracefully or pass on the error. Instead, it would throw a `500` status code and send back HTML generated by the RESTful listener. The HTML was generic and not very helpful, and we couldn't put the RESTful listener into debug mode because we were running on DBSchema Service (and who wants to send debug responses to consumers of their RESTful APIs, anyway?).

<div class="full zoomable"><img src="/images/20150617/500error.png"></div>


Instead, we wanted to send an HTTP status code that more accurately reflected the error on the insert, and perhaps even included a description in the message body. Turns out you can explicitly set the HTTP status code on the RESTful handler's response using the [documentation for X-APEX-STATUS-CODE](https://docs.oracle.com/cd/E21611_01/doc.11/e21058/rest_api.htm#AELIG7136). 

We were able to hack up the PL/SQL for our POST handler and get it to trap for the database constraint that was preventing duplicate rows with an `EXCEPTION` block. Now instead of a horrible HTML `500` error page, the service returns a nice `422` (seemed like the right error code for duplicate row) and it also returns a json text object for good measure:
 
<div class="full zoomable"><img src="/images/20150617/422error.png"></div>
 
The trick was to flesh out the code for the handler into a proper pl/sql block. There's also that special OUT parameter named X-APEX-STATUS-CODE that the APEX listener will put into the HTTP headers if you ask it real nice.
 
Here’s the code that did the trick:


{% highlight sql %}
/* original code: 
insert into im_favorites (server, engagement_name) values (:server_name, :engagement) */
 
/* fancy new code from here: http://www.oracle.com/technetwork/database/database-cloud/public/restful-wp-1844130.pdf */
 
DECLARE
--setup variables to hold the post-process queries from the database
 v_en im_favorites.engagement_name%type;
 v_existing_en im_favorites.engagement_name%type;
 
BEGIN
--regular insert from the parameters passed in on URL and header
 INSERT into im_favorites(server, engagement_name)
 values(:server_name, :engagement)
 returning engagement_name into v_en;
--set the http status code
 :status := 201;
--put the database's new data into the returned json
 :engagement_response := v_en;
 commit;
 
EXCEPTION
--if there's a row already in the database, will throw a dup_val_on_index exception
    when dup_val_on_index then
      select engagement_name
        into v_existing_en
        from im_favorites
        where server = :server_name;
--set the http status code to 422
      :status := 422;
--create a descriptive message to put in the returned json
      :engagement_response := 'Attempted to enter: ''' || :engagement || '''.  Already exists as: ''' || v_existing_en || ''' for server: ' || :server_name;
 
END;
{% endhighlight %}

And for good measure, here's the parameters as defined for the RESTful handler:

<div class="full zoomable"><img width="100%" src="/images/20150617/parameters.png"></div>

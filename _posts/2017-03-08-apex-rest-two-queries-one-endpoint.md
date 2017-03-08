---
layout: post

title: "APEX REST: Two Separate SQL Queries, One RESTful Endpoint"

excerpt: "Using PL/SQL and APEX_JSON to create just the payload we need from two separate potential queries in an APEX RESTful Resource Handler"

tags: [APEX, REST, PLSQL, Dev]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

Oracle [APEX RESTful Services are a great way to quickly build an API endpoint](http://www.oracle.com/webfolder/technetwork/tutorials/obe/db/apex/r51/restful_web_services/restful_web_services.html#section1s1). Upload a spreadsheet of data, create a SQL query, and stick it in a GET request on the Cloud with output as JSON. Boom, done.

Well, the defaults are *alright:* the meat of your sql query is returned in an `items: []` array, but it's not too hard to massage that out in your JavaScript front-end.  And you'll get the results in batches as driven by the pagination size parameter on your resource handler, so there's that. 

But sometimes you want to do some manipulating of the payload that gets returned from APEX so that it takes a particular format and doesn't need parsed on the client side. Or you might want to throw some logic in the code that runs on the server side, perhaps driven by the bind variables you pick up from a request payload or querystring parameter.

##PL/SQL to the rescue

The APEX RESTful Services UI serves as a pretty wrapper for the power of the [Oracle RESTful Data Services](http://www.oracle.com/technetwork/developer-tools/rest-data-services/overview/index.html) engine. As [documented on Tim Hall's ORACLE-BASE](https://oracle-base.com/articles/misc/oracle-rest-data-services-ords-create-basic-rest-web-services-using-plsql), ORDS lets you do all sorts of cool things if you leverage the right packages. We can take these concepts and stuff them into a APEX RESTful Service, setting our GET Handlers to use PL/SQL for Source Type instead of Query or Query One Row.

There are lots of ways to use PL/SQL for an APEX RESTful GET Handler, but they all revolve around the ability to write directly out to the wire with your response.  So instead of the automatic JSON formatting you get when you use a SQL Query or Query One Row, with PL/SQL you have to format your output yourself. For example, the APEX HR example Resource Handler uses `sys.htp.print()` to output HTML back to the client.

For my use case below, I was able to combine the ORACLE-BASE suggestion to use [the `APEX_JSON` package](https://docs.oracle.com/cd/E59726_01/doc.50/e39149/apex_json.htm#AEAPI29635) and some PL/SQL `IF THEN ELSE` logic to run one bit of SQL versus another bit depending on the length of the querystring. The `APEX_JSON.write()` procedure let me define the root node of my payload as `suggestions:` and dispense with the standard format returned by ORDS.



##Use case: Run one of two separate SQL queries depending on the length of the query parameter

I had a flat, denormalized spreadsheet of data uploaded into a PRODUCT_MASTER table, but with a two-part key: a sku-prefix and a sku-suffix.  I wanted to create an endpoint I could use for an [autocomplete plugin](https://github.com/devbridge/jQuery-Autocomplete) that would send the user-typed characters as a query parameter on the URL (the 'XXX' in `apex.oracle.com/endpoint?query=XXX`), and it expected to get JSON back in the form:

{% highlight sql %}
{"suggestions": [
    {
      "value": "Product Description",
      "data": "Product_ID"
    }, 
    ...
  ]
}
{% endhighlight %}

The issue was that if the querystring included three or fewer characters, I only wanted to search the `sku_prefix` column.  But if the querystring was four characters or longer, I wanted to use the first three characters to match the `sku_prefix` and the rest of the querystring to match the `sku_suffix` column.

So I needed two queries to be defined, but only one to run.  And some logic to choose which one based on the length of the querystring parameter passed in as a bind variable.  

The code below is what worked after defining a GET Resource Handler with the URI template: `products?query={query}`. 

N.B. See how the column aliases are surrounded by "doublequotes"?  This is so the JSON is in lowercase; otherwise your column names with be UPPERCASED and then your JSON node names will be all shouty.  

{% highlight sql %}

DECLARE
l_cursor SYS_REFCURSOR;
l_querylength NUMBER;

BEGIN 
l_querylength := LENGTH(:query);

IF l_querylength <= 3 THEN

  OPEN l_cursor FOR
    SELECT DISTINCT sku_prefix || ': ' || description_prefix as "value", 
      sku_prefix as "data", 
      description_prefix as "description_prefix",
      null as "description_suffix"  
      from product_master 
      where lower(sku_prefix) like '%' || lower(:query) || '%';

ELSE

  OPEN l_cursor FOR
    SELECT sku_prefix || sku_suffix  || ': ' || description_prefix || '-' || description_suffix as "value", 
      sku_prefix || sku_suffix as "data", 
      description_prefix as "description_prefix",
      description_suffix as "description_suffix" 
      from product_master 
      where lower(sku_prefix) like '%' || lower(SUBSTR( :query, 1 , 3  )) || '%'
      and lower(sku_suffix) like lower(SUBSTR( :query, 4  )) || '%';

END IF;

APEX_JSON.open_object;
APEX_JSON.write('suggestions', l_cursor);
APEX_JSON.close_object;

END;
{% endhighlight %}










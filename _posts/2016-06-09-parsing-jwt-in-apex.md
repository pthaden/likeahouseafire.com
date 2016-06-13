---
layout: post

title: "Parsing a Sales Cloud JWT after passing it into APEX"

excerpt: "Decoding the Java Web Token from Sales Cloud so that we can use it in APEX application items"

tags: [APEX, PL/SQL, Dev]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

The [JSON Web Token (JWT) Sales Cloud can generate](https://cloud.oracle.com/developer/solutions?scenarioid=1383852819711&solutionid=1385148959574) is useful for mashups because it can be used to call back into OSC web services as a bearer token in the Authorization header. We've [used it before in Application Express](http://likeahouseafire.com/2016/01/21/creating-osc-calendar-activity-rest/#headers-for-your-rest-calls) to pull and push data in and out of RESTful APIs.

But the JWT is more than a security tool. Buried inside the encoded token are JSON objects with details about the claims it represents. One of the data elements that Sales Cloud sends over is the logged-in user name.

On a recent project we wanted this user name in the APEX app we were writing to extend Sales Cloud. We thought it'd be nice to use in the UI wherever the `&APP_USER.` application item would normally show. We also needed it to stripe the rows of our APEX database with a user's details, so that the locally persisted data history wouldn't mix in with other users' history.

##Getting to know the JWT

The first thing is to use App Composer to create a link to our APEX app that includes a JWT. [Others](https://blogs.oracle.com/angelo/entry/jwt_token_security_with_fusion) [have](https://blogs.oracle.com/fadevrel/entry/using_jwt_to_secure_your) [explained](https://blogs.oracle.com/fadevrel/entry/using_jwt_tokens_with_rest) [this](http://www.oracle.com/technetwork/indexes/samplecode/cloud-samples-2203466.html) so much better, but here's some Groovy code that creates a URL with a JWT as an APEX page parameter.

{% highlight groovy %}
// generate a JWT for the active session
def jwt = new oracle.apps.fnd.applcore.common.SecuredTokenBean().getTrustToken()
// concatenate the JWT as a parameter on an APEX URL
return 'https://myDBSchemaCloudInstance.db.us2.oraclecloudapps.com/apex/f?p=40200039:1:::::P1_OPPTY,P1_JWT:' + OptyId + ',' + jwt 
{% endhighlight %}

The URL that gets generated has a huge base64-encoded token on it. Actually, it's three separate base64 strings, separated by periods:

<div class="highlight"><pre><code>
<span style="color: #fb015b">eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsIng1dCI6IlhnQnFTeHlCOFBRMHBKemhpdFRST2pzQXc1WSJ9</span>.<span style="color: #d63aff">eyJleHAiOjE0NjU1NDU0NTYsImlzcyI6Ind3dy5vcmFjbGUuY29tIiwicHJuIjoiSk9ITi5EVU5CQVIiLCJpYXQiOjE0NjU1MzEwNTZ9</span>.<span style="color: #00b9f1">amFgDLAmor8owD6aE00k5CcYFpkgYDbrogvGRBuuPdEQ_MYN_yc2ULP4n4jjVNcNiUNufeGcbjO9haaw8UHUbdq2oF64XSEalgTXO_daPGDeiI0SSpwExSNN9Y6Ycmi8GmV8M1o4BCHHvspL9GObUhvZRY0H5vnBKnAE7dXkJaP5i0M67t1ea4R9yBKaIfRnXvkW98RqddJUA5tlgGurLQW2pkFzQ-jBKS0tpZNqUy1wYfV5D-GmXRnBYEtQPnpxP3buYvTW84JmhYtlZCOU8CdWMckqugpPfsxs3c3b9S4a9NMifnBVYXVODLPkzkQKuZeDjZJn-x57CyciEy9CSQ</span>
</code></pre></div>

The first section is the JWT header, the middle is the actual payload, and the last section is the signature that Sales Cloud uses to confirm that our JWT is legit.  You can paste that JWT into a [decoder like this one](https://jwt.io/) to see the different parts in their pre-encoded state.

We normally use the entire JWT as a bearer token in order to access Sales Cloud APIs while our session is still alive. But remember that each section of the JWT is itself simply a base64 string.  For fun, [use a base64 decoder](https://www.base64decode.org/) to paste only the middle payload section and turn it back into JSON:

{% highlight json %}
{
  "exp":1465545456,
  "iss":"www.oracle.com",
  "prn":"JOHN.DUNBAR",
  "iat":1465531056
}
{% endhighlight %}

##Parsing out the JWT's info

That `"prn"` node has the logged-in username (the [previously-linked Fusion Dev Relations blog](https://blogs.oracle.com/fadevrel/entry/using_jwt_to_secure_your) explains what the other nodes in the payload mean). Sure, we could get that username by calling back into Sales Cloud and hitting the [`UserDetailsService`](http://docs.oracle.com/cd/E60665_01/salescs_gs/CSAPP/usertoken009.htm#CSAPP7147), but why bother? The info we are looking for is *right there* in the JWT, it's just locked up in a base64-encoded JSON object.

We can use some PL/SQL to decode everything and grab the Sales Cloud username out of the JWT.  Here's the steps we need to follow:

1. grab the entire JWT and determine where the two delimiter periods are in the string
2. use those markers to lift out the middle payload section
3. base64-decode it into a string of JSON
4. use the JSON parser to pluck out the `prn` node

Here's that PL/SQL code.  It assumes that the JWT has come over on the URL and that it's been stuffed into a hidden page item named `P1_JWT`.  There's some fallback code that returns the APEX `APP_USER` item if there's no JWT to parse.

{% highlight sql %}

declare
l_jwt varchar2(4000);
l_start number;
l_end number;
l_middle varchar2(4000);
l_json varchar2(4000);
l_output varchar2(4000);
l_jwt_user varchar2(4000);
l_return varchar2(4000);

begin


if :P1_JWT IS NULL THEN 
return :APP_USER;

ELSE
--JWT is made of of three parts, separated by periods .
--we want to glean the username out of the middle one
-- see https://blogs.oracle.com/fadevrel/entry/using_jwt_to_secure_your
l_jwt := :P1_JWT;

--find the location of the two period markers
l_start := INSTR( l_jwt, '.' );
l_end := INSTR( l_jwt, '.', l_start+1);

--take the middle substring 
l_middle := SUBSTR( l_jwt, l_start+1 , (l_end - l_start)-1);

--need to base64 decode the middle string to get a json string
l_json := utl_raw.cast_to_varchar2(UTL_ENCODE.BASE64_DECODE( utl_raw.cast_to_raw( l_middle)));


--use the APEX parser and then find the username at the 'prn' node
apex_json.parse(l_json);
l_jwt_user := initcap(apex_json.get_varchar2(p_path => 'prn'));

return l_jwt_user;


END IF;
end
{% endhighlight %}

Since this was a one-shot decode, I first created an Application Item Shared Component named `JWT_USER` and then put this code directly in a new computation on my Page 1's Pre-Rendering >> Before Header >> Computations.  The computation's SQL populated the application item with its return value, which could then be used elsewhere using the `&JWT_USER.` substitution string syntax. A better approach for the code above might be to create a function and make this SQL more reusable, but it gets the job done.

In this screenshot I've unhidden the two page items that "catch" the passed-in URL parameters and also created another display-only item to show the computed JWT_User field can be used wherever needed.

<div class="full zoomable"><img src="/images/20160609/jwtfields.png"></div>





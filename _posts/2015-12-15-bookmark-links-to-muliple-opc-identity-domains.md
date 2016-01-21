---
layout: post

title: Bookmark Links to Multiple Oracle Public Cloud Identity Domains


excerpt: "Deeplinks straight to your services, bypassing the identity domain page and leveraging 1Password"

tags: [Oracle Cloud]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

As a sales consultant I have access to more than one Oracle Public Cloud instance, most of them trial environments and each one in its own identity domain.

In the fall of 2015 the Oracle Public Cloud implemented a "feature" in the sign-on flow that prompted for an identity domain on its own page before asking for your username and password. Beyond adding an extra page to the sign-on process, this new form had a checkbox that would remember your identity domain for future logins and store it in a cookie. Bad for those of us who need to sign into more than one domain if you accidentally checked that box (see [Problems Switching Identity Domains When Signing In to Oracle Java Cloud Service](https://docs.oracle.com/cloud/latest/jcs_gs/JSCUG/GUID-57C58820-B690-4C1C-AAA4-4C3615227FB5.htm#JSCUG-GUID-57C58820-B690-4C1C-AAA4-4C3615227FB5) for help if you need to delete this cookie).

<div class="full zoomable"><img src="/images/20151215/opcsignin.png"></div>

So now I'm extra careful and I have to keep track of and paste the proper identity domain into the sign-on window, immediately followed by another page with the right username and password. That's a lot of screens and page refreshes and it's an error prone process just to get access to a service's console.

Until today, when I discovered URLs that take on this format:

{% highlight bash %}

https://myservices.us2.oraclecloud.com/mycloud/your_identity_domain_name/faces/dashboard.jspx

{% endhighlight %}

So you can edit that URL to match your_identity_domain_name and use it to go straight to your services home page after entering the right password. No more interstitial page prompting for the identity domain. No navigating through the cloud.oracle.com sign-on pages. Plus, that URL is bookmarkable.

This is great, because I had earlier tried to bookmark direct links in to the service consoles for my JCS-SX and APEX services, such as: 
{% highlight bash %}
https://javatrial_number_db-identity_domain_name.db.us2.oraclecloudapps.com/apex/
{% endhighlight %}

 I couldn't get these to work reliably because they assume you're already authenticated to the identity domain with an active session--if you are the bookmarks would work great, but if you're not it would stop at that same set of signin pages prompting for the identity domain and then the username/password so the bookmark wasn't saving me any steps or time.

And the mycloud link above even works with my metered service domains as well, signing me straight in.  Once I'm authenticated to the domain then bookmarks such as 
{% highlight bash %}dbaas.oraclecloud.com/dbaas/faces/dbRunner.jspx{% endhighlight %}
 work right too.

<div class="full zoomable"><img src="/images/20151215/myhome.png"></div>

The best part is that I can store these links in [my password manager](https://agilebits.com/onepassword) and let the "open and fill" feature take me right to the MyServices home page for the identity domain. I've even kept the old bookmarks that point straight to the consoles for after I'm authenticated.

<div class="full zoomable"><img src="/images/20151215/1password.png"></div>

Click that website link and 1Password logs you straight in. Combined with [Alfred](https://www.alfredapp.com/help/features/1password/) or another password-manager-integrated launcher and my Oracle Cloud is just few keystrokes away.



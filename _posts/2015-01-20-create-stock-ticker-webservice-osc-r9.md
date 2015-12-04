---
layout: post

title: Create a Stock Ticker WebService in Oracle Sales Cloud R9


excerpt: "How to hook up a Sales Cloud to an external SOAP Web Service"

tags: [Sales Cloud, Web Services]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

How to hook up a Sales Cloud to an external SOAP Web Service:

## StockQuote Web Service

We'll use the StockQuote SOAP web service available online as of 20-Jan-2015 at:  [http://www.webservicex.net/stockquote.asmx?op=GetQuote](http://www.webservicex.net/stockquote.asmx?op=GetQuote)

<div class="full zoomable"><img src="/images/20150120/stockquote-web-service.png"></div>

## Create the Web Service in Sales Cloud App Composer

Log into Sales Cloud as matt.hooper and create and activate a sandbox.  

Launch App Composer and click on **Web Services** in the Common Setup tree.

Click the **Create Web Service Reference** button on the toolbar to create a new entry.





<div class="full zoomable"><img src="/images/20150120/create-the-web-service-in-sales-cloud-app-composer.png"></div>

## Create SOAP Web Service Connection

Fill in the details for the new web service:

Name:  **StockQuote**

WSDL URL:  [**http://www.webservicex.net/stockquote.asmx?wsdl**](http://www.webservicex.net/stockquote.asmx?wsdl)

Click **Read WSDL** and give it a moment to retrieve the service details.



<div class="full zoomable"><img src="/images/20150120/create-soap-web-service-connection.png"></div>

## Refine SOAP Connection Details

The Service should be automatically selected as [{http://www.webserviceX.NET/}StockQuote]({http://www.webserviceX.NET/}StockQuote) 

Change the **Port** to StockQuoteSoap12.

Click **None** for the Security Scheme and then click **Save and Close**

<div class="full zoomable"><img src="/images/20150120/refine-soap-connection-details.png"></div>

## Add a custom field to the Account Object

Navigate to the Application Common, Standard Objects tree and select **Account >> Fields**.

In the Custom tab, click on the **Create a Custom Field** button in the toolbar, then choose **Formula** and click OK.

<div class="full zoomable"><img src="/images/20150120/add-a-custom-field-to-the-account-object.png"></div>

## Formula Field Attributes

Create the field with the following details:

Formula Type:  **Text**

Display Label: **Latest Stock Price**

Display Width: **30** characters

Display Type:  **Multiline Text Area**

Name: **LatestStockPrice**

Click **Next**

<div class="full zoomable"><img src="/images/20150120/formula-field-attributes.png"></div>

## Build the formula

Use the palette to build the formula, or simplly paste this text into the expression editor:

{% highlight groovy %}
/*

 * StockTicker field formula

 * last updated 20-Jan-2015

 * see [http://www.webservicex.net/stockquote.asmx?op=GetQuote](http://www.webservicex.net/stockquote.asmx?op=GetQuote)

 *

 */



// intialize return variable

def lastPrice = "UNKNOWN"



// build SOAP call with Account's StockSymbol field

// wrapped in a nvl() in case the StockSymbol field is empty

def out = adf.webServices.StockQuote.GetQuote(nvl(StockSymbol,""))



// these will be used to parse the CDATA returned by the call

def sLast = '<Last>'

def eLast = '</Last>'

def sChange = '<Change>'

def eChange = '</Change>'

def sTime = '<Time>'

def eTime = '</Time>'

def sDate = '<Date>'

def eDate = '</Date>'



// find the starting position for last price

def sPosition = out.indexOf(sLast)

def ePosition = out.indexOf(eLast)



// parse out just the sections of the out string between the tags only if <Last> was found

if ( sPosition != -1 ) {

  

  lastPrice = out.substring(sPosition + 6, ePosition) 

  

  sPosition = out.indexOf(sChange)

  ePosition = out.indexOf(eChange)

  lastPrice += ' (' + out.substring(sPosition + 8, ePosition)+ ')\n'



  sPosition = out.indexOf(sTime)

  ePosition = out.indexOf(eTime)

  lastPrice += ' as of ' + out.substring(sPosition + 6, ePosition)

  

  sPosition = out.indexOf(sDate)

  ePosition = out.indexOf(eDate)

  lastPrice += ',  ' + out.substring(sPosition + 6, ePosition)

  

}

return lastPrice
{% endhighlight %}

<div class="full zoomable"><img src="/images/20150120/build-the-formula.png"></div>


## Place the Formula Field on the Right SUI Page

Navigate to **Account >> Pages** in the App Composer.

Under **Details Page Layouts**, highlight the **Default Layout** and click the **Edit Layout** button



<div class="full zoomable"><img src="/images/20150120/place-the-formula-field-on-the-right-sui-page.png"></div>

## Edit the Profile Subtab

Click on the **Profile** sidebar subtab, and then click the **Edit** pencil next to **Summary**

<div class="full zoomable"><img src="/images/20150120/edit-the-profile-subtab.png"></div>

## Position Field on Page

Scroll down the Available Fields to **Latest Stock Price** and click the right shuttle button to move it over.

Use the **Up** button to positiion the new field underneath **Stock Symbol**.

Click **Save and Close**.

<div class="full zoomable"><img src="/images/20150120/position-field-on-page.png"></div>

## Test your new field

Navigate back to the home page, and then into **Accounts**.

Search for an account name such as **A. C. Networks** or define a saved search.  Click on the account name link.

On the Account subtabs, click the **Profile** tab (second from the top).

The first time in, your Latest Stock Price will be UNKNOWN, because there is no ticker in the Stock Symbol field.



<div class="full zoomable"><img src="/images/20150120/test-your-new-field.png"></div>

## Ticker Data!

Fill in the **Stock Symbol** field with a valid stock ticker symbol. 

Click **Save and Close **to return to the Accounts listing.

Click again on the account name link and then on the Profile subtab.

Note the updated stock price info in the **Latest Stock Price** field!



<div class="full zoomable"><img src="/images/20150120/ticker-data-.png"></div>


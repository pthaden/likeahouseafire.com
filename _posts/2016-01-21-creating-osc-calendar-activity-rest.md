---
layout: post

title: "Creating Oracle Sales Cloud Calendar Activities Via APEX REST Calls"

excerpt: "Notes gleaned from building APEX RESTful calls into Fusion"

tags: [APEX, Sales Cloud, Dev]

author:
  name: Paul Thaden
  twitter: pthaden
  gplus: 111502137181170252947 
  bio: Dad to twin boys and twin girls; Retooling in my 40s around front-end dev and JavaScript; Oracle CX Apps Sales Consultant; all-around guy
  image: Avatar.png
---

Recently I used Application Express 5.0's "Group Calendar" packaged app to simulate a scheduling system.  We wanted to show how external systems can be launched from within Sales Cloud and then turn around and create meeting appointments via RESTful web service calls back into Fusion, and the packaged app was the quickest way to prototype a 3rd-party "scheduling" system that can make REST calls. 

I hijacked the Create Event button procedure that already existed in this packaged app, adding functionality to have the Fusion APIs create a corresponding meeting in Sales Cloud and attaching a contact to the appointment.

Here are some random notes picked up during this assignment for refining on future projects:

(tl;dr:  you can use APEX to do cool REST calls and query/create via Fusion APIs)

* TOC
{:toc}

## R10 RESTful documentation
The new [R10 RESTful APIs are documented](https://docs.oracle.com/cloud/latest/salescs_gs/FAAPS/api-Activities.html) in a fancy new format, as opposed to the SOAP API format that is [available in the OER](https://fusionappsoer.oracle.com/oer/index.jsp).

There are [really good R10 RESTful API intro write-ups](https://blogs.oracle.com/fadevrel/entry/r10_sales_rest_api_concepts) on the FADevRel blog. There are also some [older articles on integrating to Fusion with APEX](https://blogs.oracle.com/fadevrel/tags/apex), and although the Fusion blog entries are mostly SOAP-oriented there is at least one newer one about [creating a REST request in PL/SQL](https://blogs.oracle.com/fadevrel/entry/integrating_with_fusion_application_using18) (albeit not to Fusion; read on for that!).

And there's the monstrously-large [Oracle Sales Cloud
Using RESTful Web Services](https://support.oracle.com/epmos/faces/DocContentDisplay?id=1981941.1) whitepaper available on MOS.

## Using APEX to make the RESTful calls
I used [`apex_web_service.make_rest_request`](https://docs.oracle.com/database/121/AEAPI/apex_web_service.htm#AEAPI1955) to send my requests back to Sales Cloud from a PL/SQL package so that it would fit in with the existing scaffolding in the packaged app. I originally wanted to model the Fusion RESTful APIs as [Web Service References in Application Express](http://www.oracle.com/webfolder/technetwork/tutorials/obe/db/apex/r50/Restful%20Services/restful_services.html#section2) and even got the Fusion API set up as a APEX Shared Component, but ended up rewriting the call as a PL/SQL package to match the existing Group Calendar code.

Still, the APEX Web Service References has a nice test harness to let tweak your headers and see the JSON response from your call. It helped me validate that the payload I was creating worked from APEX. Click here for a [3.4MB animated GIF of the testing harness proving it could create an appointment in Sales Cloud](/images/20160121/test_APEX_rest_create-appointment.gif).

<div class="full"><a href="/images/20160121/test_APEX_rest_create-appointment.gif"><img src="/images/20160121/test_APEX_rest_create-appointment_thumbnail.png"></a></div>


## Tests Tooling

For a RESTful client to draft that payload I used [Paw](https://luckymarmot.com/paw) as a change of pace from [Postman](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en) (I like them both, Paw was just something new to try). These tools were quicker for iterating over changes I made to the request payload.

When I went to implement the call in APEX as a PL/SQL package, I also created a logging table that I inserted the response (as APEX saw it) in a timestamped row. This helped me troubleshoot the more complicated, multi-stage calls that came later. 

{% highlight sql %}

--select dbms_metadata.get_ddl('TABLE','PDT_OSC_CAL_API_LOG') from dual; 
 CREATE TABLE "PDT_OSC_CAL_API_LOG" ( "RESPONSE_MSG" CLOB, "CREATED" DATE, "CREATED_BY" VARCHAR2(100), "ID" NUMBER, "P_BODY" VARCHAR2(4000), CONSTRAINT "PDT_OSC_CAL_API_PK" PRIMARY KEY ("ID") ) 

--select dbms_metadata.get_ddl('TRIGGER','BI_PDT_OSC_CAL_API_LOG') from dual; 
 CREATE OR REPLACE TRIGGER "BI_PDT_OSC_CAL_API_LOG" before insert on "PDT_OSC_CAL_API_LOG" for each row 
 begin 
   if :NEW."ID" is null then 
     select to_number(sys_guid(),'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX') into :new.id from dual;
     :NEW."CREATED" := sysdate; 
     :NEW."CREATED_BY" := V('APP_USER');
   end if; 
 end; 
 ALTER TRIGGER "BI_PDT_OSC_CAL_API_LOG" ENABLE
{% endhighlight %}


## Headers for Your REST calls
One of the first things I learned was that the R10 Fusion RESTful APIs require a special `Content-Type` header: `application/vnd.oracle.adf.resourceitem+json`. You also need to send some form of `Authorization` header: I used Basic Auth for testing but for the APEX app I could use Bearer Auth because we passed in a [JWT token](https://blogs.oracle.com/angelo/entry/jwt_token_security_with_fusion) on the URL generated in Sales Cloud pointing at our APEX app.

{% highlight sql %}
      -- Headers necessary for Fusion RESTful API
      apex_web_service.g_request_headers(1).name := 'Content-Type';
      apex_web_service.g_request_headers(1).Value := 'application/vnd.oracle.adf.resourceitem+json';
      apex_web_service.g_request_headers(2).name := 'Authorization';
      apex_web_service.g_request_headers(2).Value := 'Bearer ' || p_jwt_token;
{% endhighlight %}


## Activities are different
Since I wanted to create a meeting appointment, a [little background on the Sales Cloud Activity model](https://blogs.oracle.com/fadevrel/entry/release_9_the_activity_redesign) was helpful. I used [GETs](https://docs.oracle.com/cloud/latest/salescs_gs/FAAPS/op-salesApi-resources-11.1.10-activities-get.html) to query existing meeting appointments and figure out the formats for fields (note the ISO 8601 date format and the Base64-encoded description). I was able to pare down a POST's payload body sent to `/salesApi/resources/latest/activities` to be just a few fields (not shown are the two headers: Basic Auth and Content-type: application/vnd.oracle.adf.resourceitem+json):

{% highlight json %}
{
  "ActivityEndDate": "2016-01-08T19:30:00-08:00",
  "Subject": "Test RESTful Appointment",
  "ActivityFunctionCode": "APPOINTMENT",
  "ActivityTypeCode": "MEETING",
  "ActivityStartDate": "2016-01-08T18:30:00-08:00",
  "OwnerId": 300000047342468,
  "ActivityDescription": "SGVyZSBpcyBhbiBhY3Rpdml0eSBkZXNjcmlwdGlvbiB0aGF0IEkgY291bGQgY3JlYXRlIGluIHRoZSBzYW1lIHJlcXVlc3QgYXMgdGhlIGFjdGl2aXR5IGl0c2VsZjsgSSBkaWRuJ3QgaGF2ZSB0byBtYWtlIGEgc2Vjb25kIGNhbGwgd2l0aCB0aGUganVzdC1jcmVhdGVkIGFjdGl2aXR5IElE"
}
{% endhighlight %}

## Whittle down your response payload
The response object that comes back from a Fusion REST call is huge, but it doesn't have to be. [Two query parameters can help](https://docs.oracle.com/cloud/latest/salescs_gs/FAAPS/op-salesApi-resources-11.1.10-activities-%7BActivityNumber%7D-get.html): `?onlyData` (so you don't get all the nested `links[]` arrays) and `?fields=Attribute1,Attribute2` (so that your main object only has the fields you're looking for). 

## Watch out for too-huge responses in APEX
This response pruning ability was handy, because [there's a bug in APEX's `APEX_JSON_PARSE` when running on 11g](http://www.talkapex.com/2015/05/apexjsonparse-issue-with-clobs-and-11g.html) that I ran into on our DBSchema instance. JSON responses larger than 8191 characters failed to parse, even though I could see a valid response was coming back in my log table. Turns out it's really easy to get larger-than 8K+ responses from a Fusion REST call.

## Adding Contacts to a meeting are a two-phase REST call 
I needed to parse the CLOB response as JSON because I wanted to tease out the `ActivityNumber` field and use it to build the URL for a follow-up POST call. Our flow into the APEX app was to start from a Contacts screen, so in addition to the [JWT Token](https://blogs.oracle.com/fadevrel/entry/using_jwt_to_secure_your) we also send in the Contact_ID of the Fusion record we were looking at.  When we created the appointment meeting via REST we also wanted to add the Contact as an attachment. 

This necessitated a two-part REST call: the first to create the activity and the second to use the returned ID in another [POST URL to create an activity contact](https://docs.oracle.com/cloud/latest/salescs_gs/FAAPS/op-salesApi-resources-11.1.10-activities-%7BActivityNumber%7D-child-ActivityContact-post.html) at `/salesApi/resources/latest/activities/{ActivityNumber}/child/ActivityContact`. The body of that POST was only the Contact_ID that we were passed in as a parameter on the URL generated by App Composer.

## APEX will let you do random GETs
Since I had been carrying that Contact_ID around from screen to screen in APEX, I thought a nice touch was to call back in to Fusion to pick up some additional details about the contact, such as full name and email address. 

I used an After Header Process (after Load Data) on the APEX page to pre-populate two fields with live Fusion contact data. It might make more sense to pull Fusion data in differently for use in other APEX pages, but the one-off GET did the trick for this APEX form.

{% highlight sql %}
DECLARE
   l_response_clob    clob;

BEGIN
      -- Headers necessary for Fusion RESTful API
      apex_web_service.g_request_headers(1).name := 'Content-Type';
      apex_web_service.g_request_headers(1).Value := 'application/vnd.oracle.adf.resourceitem+json';
      apex_web_service.g_request_headers(2).name := 'Authorization';
      apex_web_service.g_request_headers(2).Value := 'Bearer ' || :JWT_TOKEN;


      -- make the request using parameters
      l_response_clob := apex_web_service.make_rest_request(
              p_url => 'https://adc2-fap1370-crm.oracledemos.com/crmCommonApi/resources/latest/contacts?q=PartyId=' || :CONTACT_ID || '&onlyData&fields=ContactName,EmailAddress',
              p_http_method => 'GET'
      );

      -- log it
      insert into PDT_OSC_CAL_API_LOG(RESPONSE_MSG, P_BODY)
      values(l_response_clob, 'jwt: ' || :JWT_TOKEN ||' contact_id: ' || :CONTACT_ID);
      commit;


      -- parse the response to put contact data into APEX fields
      apex_json.parse(l_response_clob);
      :P10_CONTACT_PERSON := apex_json.get_varchar2(p_path => 'items[1].ContactName'); 
      :P10_CONTACT_EMAIL := apex_json.get_varchar2(p_path => 'items[1].EmailAddress');
END
{% endhighlight %}



## Mirroring the existing APEX code
Finally, here's my custom package that mirrored the APEX process that runs when CREATE was pressed on the existing APEX page. I just added a call to my custom package right after the call to `EBA_ca_api.create_event`, using the same parameters.

{% highlight sql %}

create or replace package body PDT_OSC_cal_api
as

--this signature mirrors the create_event in the Group Calendar packaged app; I didn't end up using all these fields
procedure create_event (
   p_event_name       varchar2,
   p_type_id          number,
   p_new_event_type   varchar2,
   p_event_date_time  timestamp with local time zone,
   p_duration         number,
   p_event_desc       varchar2,
   p_contact_person   varchar2,
   p_contact_email    varchar2,
   p_display_time     varchar2,
   p_location         varchar2,
   p_link_name_1      varchar2,
   p_link_url_1       varchar2,
   p_link_name_2      varchar2,
   p_link_url_2       varchar2,
   p_link_name_3      varchar2,
   p_link_url_3       varchar2,
   p_tags             varchar2 default null,
   -- here are two fields I added to the signature: they are passed in as parameters on the Sales Cloud-generated link and I passed them around as application items
   p_contact_id       varchar2,
   p_jwt_token        varchar2,
   --
   p_recur_flag       varchar2,
   p_recur_freq       varchar2,
   p_recur_end_date   timestamp with local time zone )
is
   l_event_type_id    number  default null;
   l_series_id        number;
   l_response_clob    clob;
   --convert p_duration (hours) into timestamp for use in p_body
   l_event_end_date   timestamp with local time zone := p_event_date_time + (p_duration * 1/(24));
   --clean up and convert p_tags into Base64 for use in ActivityDescription
   --alas the REPLACE()s make newlines if needed 
   --but you cant have multi-line JSON strings and something in the cast chain was splitting and inserting CRLFs
   l_activitydesc     varchar2(32767) := REPLACE(
                              REPLACE( 
                              utl_raw.cast_to_varchar2(utl_encode.base64_encode(utl_raw.cast_to_raw('Interest in CSG: '|| REPLACE(p_tags, ':', ', ')))), 
                              CHR(13), 
                              '\n' ),
                           CHR(10), 
                           '' ) ;
   l_pbody            varchar2(4000);
   l_new_activity_id  varchar2(30);

begin
       
   if p_recur_flag = 'Y' then
      -- TODO: write recurring event logic in the future; code structure was 'borrowed' liberally from the APEX packaged app codebase
      NULL;
   else
      
      -- Headers necessary for Fusion RESTful API
      apex_web_service.g_request_headers(1).name := 'Content-Type';
      apex_web_service.g_request_headers(1).Value := 'application/vnd.oracle.adf.resourceitem+json';
      apex_web_service.g_request_headers(2).name := 'Authorization';
      apex_web_service.g_request_headers(2).Value := 'Bearer ' || p_jwt_token;

      --build the payload, hardcoded to go to lisa.jones' calendar because I didn't build out lookups in APEX against live Fusion OwnerIds
      l_pbody := '{
                "ActivityEndDate": "' || to_char(l_event_end_date, 'yyyy-mm-dd') ||'T'|| to_char(l_event_end_date, 'hh24:mi:ss') || '-08:00",
                "Subject": "' || p_event_name || '",
                "ActivityFunctionCode": "APPOINTMENT",
                "ActivityTypeCode": "MEETING",
                "ActivityStartDate": "' || to_char(p_event_date_time, 'yyyy-mm-dd') ||'T'|| to_char(p_event_date_time, 'hh24:mi:ss') || '-08:00",
                "OwnerId": 300000047342468,
                "ActivityDescription": "'|| l_activitydesc ||'"
              }';

      -- make the request using ?onlyData parameter
      l_response_clob := apex_web_service.make_rest_request(
              p_url => 'https://origin-adc2-fap1370-crm.oracledemos.com/salesApi/resources/latest/activities?onlyData',
              p_http_method => 'POST',
              p_body  => l_pbody
      );

      -- log it
      insert into PDT_OSC_CAL_API_LOG(RESPONSE_MSG, P_BODY)
      values(l_response_clob, l_pbody);
      commit;

      -- parse the response so we can attach a contact
      apex_json.parse(l_response_clob);
      l_new_activity_id := apex_json.get_varchar2(p_path => 'ActivityNumber');

      -- make the follow-up POST request to add a contactID to the just-created activity
      l_pbody := '{    
              "ContactId" : ' || p_contact_id || '
      }';

      l_response_clob := apex_web_service.make_rest_request(
              p_url => 'https://origin-adc2-fap1370-crm.oracledemos.com/salesApi/resources/latest/activities/' || l_new_activity_id || '/child/ActivityContact?onlyData',
              p_http_method => 'POST',
              p_body  => l_pbody
      );
      
      -- log it
      insert into PDT_OSC_CAL_API_LOG(RESPONSE_MSG, P_BODY)
      values(l_response_clob, l_pbody);
      

   end if;

   commit;
end create_event;

end PDT_OSC_cal_api;
{% endhighlight %}


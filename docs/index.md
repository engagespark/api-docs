This page documents the [engageSPARK](https://www.engagespark.com) API.
You can use them to automate Voice and 2-way SMS even further.

If you have any questions, please don't hesitate [to reach out](https://www.engagespark.com/contact-us/).

* TOC
{:toc}

# Subscription API

Do you want to send SMS or calls to a phone number on demand?
Maybe you want to integrate SMS or calls into your app?
Then this API is for you!

For example, you may have a running IVR Poll that interviews job applicants.
When a new applicant contacts you, this API allows you to send the interview 
call to the applicants phone.

On subscription, the SMS or call will be dispatched immediately.

For more information and other ways to subscribe, see the [subscription on the Contacts page](https://www.engagespark.com/support/subscribe-add-contacts-existing-campaign/) and the list of [APIs and other integrations](https://www.engagespark.com/support/how-can-i-use-your-api/).

How it works
------------

You'll send a POST request to the Subscription URL of an Engagement.
That POST request contains JSON with the details about the contacts
to subscribe.

The contacts can either be new or existing ones. 

Prerequisites
-------------

1. First, you need **a running Engagement** to which you want to subscribe new contacts.
   Engagements are running, 
   if they show up in your [Engagement list](https://start.engagespark.com/engagements)
   and have a green status of `RUNNING`, as in the following image:
  
    ![Shows the RUNNING status of a SMS Poll.]({{ site.baseurl }}/images/subscription-running-engagement.png)
  
    If you don't have a running Engagement yet, please [create it here](https://start.engagespark.com/engagement/create).
  
1. You need to create the **Subscription URL**, see below.

1. As with all APIs, you need your **API key**, see [Authentication](#authentication) below.

The Subscription URL
--------------------

The generic URL looks like this:
   
   ```
   https://start.engagespark.com/api/v1/engagements/{ENGAGEMENT_ID}/contacts/
   ```
   
   So, if your Engagement ID is `1234`, then your subscription URL would look like this:
   
   ```
   https://start.engagespark.com/api/v1/engagements/1234/contacts/
   ```
   
   Tip: You can find the Engagement ID in the URL when opening the report of the Engagement.
   For example, the Engagement of this report `https://start.engagespark.com/engagement/1234/report/` has the ID `1234`.

HTTP Method & Headers
---------------------

The HTTP method is `POST`, and you'll need to set the following HTTP Headers:

    Authorization: Token {API_KEY}
    Content-type: application/json

Please don't forget to replace your {API_KEY} with your API token, see [Authentication](#authentication).

Subscribe a single phone number
-------------------------------

To subscribe a single phone number, send this JSON payload with your POST request:

```
{"fullPhoneNumber": "{PHONENUMBER}"}
```

Replace `{PHONENUMBER}` with your phone number. That's it.

See the [section on phonenumbers below](#phonenumber-formats) on details of the supported phonenumber formats.

Note that by default the Subscription API will create a new contact every time.
The contact will belong to your *current organization*. 
To change this, see the `organizationId` field below.

Subscribing multiple new contacts
---------------------------------

Here's how you can create multiple contacts at the same and subscribe them to the Engagement.


```json
[{
    "firstName": "Juan",
    "lastName": "Dela Cruz",
    "fullPhoneNumber": "639438249629",
    "organizationId": 853,
    "groups": [321]
}, {
    "firstName": "John",
    "lastName": "Doe",
    "fullPhoneNumber": "639438249629",
    "organizationId": 853,
    "groups": [321]
}, {
    "firstName": "Jane",
    "lastName": "Doe",
    "fullPhoneNumber": "639438249629",
    "organizationId": 853,
    "groups": [321]
}]
```


As you can see, you're able to provide all standard fields, 
as well as the organization that the contacts should be created under
and the groups the contacts should be part of.

If you don't specify an organization, your *current organization* will be used.

Note that by default the Subscription API will create a new contact every time.

Subscribe existing contacts
---------------------------

Here is how you subscribe existing contacts, using their ID.

```json
{
  "contacts": [{
    "id": 17569341
  }, {
    "id": 17569342
  }, {
    "id": 17569343
  }]
}
```

You can find out the ID of existing contacts by [downloading the contacts](https://start.engagespark.com/contacts) as CSV or Excel files.

Response
--------

Here is an example response from successfully subscribing 3 contacts.

HTTP Status: `201 CREATED`

```json
{
  "data": [
    {
      "language": "",
      "fullPhoneNumber": "639438249631",
      "firstName": "Jose",
      "organizationId": 853,
      "lastName": "Rizal",
      "contactType": "NORMAL",
      "id": 17620219,
      "deleted": "",
      "carrierId": 952,
      "phoneNumber": "9438249631",
      "groups": [],
      "createdDate": "2015-12-23T17:13:41+08:00",
      "dialingCode": "63",
      "customFields": {},
      "email": ""
    },
    {
      "language": "",
      "fullPhoneNumber": "639438249631",
      "firstName": "Jose",
      "organizationId": 853,
      "lastName": "Rizal",
      "contactType": "NORMAL",
      "id": 17620219,
      "deleted": "",
      "carrierId": 952,
      "phoneNumber": "9438249631",
      "groups": [],
      "createdDate": "2015-12-23T17:13:41+08:00",
      "dialingCode": "63",
      "customFields": {},
      "email": ""
    },
    {
      "language": "",
      "fullPhoneNumber": "639438249631",
      "firstName": "Jose",
      "organizationId": 853,
      "lastName": "Rizal",
      "contactType": "NORMAL",
      "id": 17620219,
      "deleted": "",
      "carrierId": 952,
      "phoneNumber": "9438249631",
      "groups": [],
      "createdDate": "2015-12-23T17:13:41+08:00",
      "dialingCode": "63",
      "customFields": {},
      "email": ""
    }
  ]
}
```

Re-use existing contacts when possible
--------------------------------------

When passing new contacts, the Subscription API can still try to use existing contacts.
This has the advantage of being able to groups calls to the same person.
For example, the Contacts Reports of the IVR Poll
will automatically highlight calls from the same contact.

How does the Subscription API determine existing contacts?
For each contact, the phonenumber is used to find existing contacts.
Then, one contact out of the possibly many found is selected for re-use.
Currently, the newest contact is chosen.

Subscribing contacts that are not found will be created.

To use this setting, the GET parameter `try_using_existing` must be set to `newest`.

Example for a subscription API URL which enable contact re-use:

    https://start.engagespark.com/api/v1/engagements/1234/contacts/?try_using_existing=newest

**Current catch**: All other parameters of the contacts, whose existing pendant is found, are discarded.
So, even if a name or group is passed, will be ignored if a contact is re-used.

Complete Examples
-----------------

Using the command-line tool curl, here's how you would subscribe the phone number
`63123456789` to the Engagement with ID `1234` and the API token `thisismyapitoken123`.
We would try to find an existing contact with that phonenumber and, if none exists, create a new one.

```shell
curl -H 'Content-Type: application/json' -H 'Authorization: Token thisismyapitoken123' -d '{"fullPhoneNumber": "63123456789"}' https://start.engagespark.com/api/v1/engagements/1234/contacts/?try_using_existing=newest
```

# Authentication

An **API Key** is needed to authenticate yourself. Find it in [your profile](https://start.engagespark.com/profile)
under `Account Settings` > `API Key`.

In the HTTP request, include an HTTP `Authorization` header with the value `Token {API_KEY}`. Replace `{API_KEY}` with the one you obtained from above.

e.g. `Token 8eddfd73d459b03505ac1497feea87d1b8c84cbd`


# Phonenumber Formats

Phone numbers in the API need to have an **international dialing code**.

For example, for a number from the Philippines (with the dialing code `63`), `6312345678` would be correct, whereas `12345678` would be missing the dialing code. If you are not sure about the dialing code of the country you want to send calls or SMS to, check this [Wikipedia article on dialing codes](https://en.wikipedia.org/wiki/List_of_country_calling_codes#Ordered_by_code).

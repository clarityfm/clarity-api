# Clarity API

The Clarity API is in beta and is still subject to change. Major breakage is unlikely, but response parameters may yet be renamed, added or removed.

Although not specified in the [Endpoints](#endpoints) documentation, all Clarity API calls require an `oauth_token` for authorization and to establish identity.  More information about registering an OAuth client and obtaining a token can be found in the [OAuth](#oauth) section.

## OAuth

Clarity is an OAuth provider.  Presently we implement a subset of v10 of the OAuth2 draft specification, found at http://tools.ietf.org/html/draft-ietf-oauth-v2-10. Rack applications can use the [omniauth-clarity](http://github.com/clarityfm/omniauth-clarity) gem for painless integration.

### Registering an Application

OAuth Applications can be registered at http://api.clarity.fm/applications.  Upon creation, you will be issued a client_id and a client_secret, both of which are required for authorization requests.  Store your secret someplace safe, as it's not possible to obtain it again after the initial application creation.  It can be regenerated, but this will require updating all clients.

### Requesting an oauth_token

```
GET https://clarity.fm/oauth/authorize?
    client_id=...&
    response_type=code
```

```
HTTP 302 http://your/callback/uri?code=...
```

If successful, we'll redirect to the redirect URI that was specified when the application was created, passing a code parameter in the query string.  This can then be used to request an oauth_token:

```
POST https://clarity.fm/oauth/authorize?
     client_id=...&
     client_secret=...&
     code=...&
     grant_type=authorization_code
```

```
  HTTP 200
  Body: oauth_token
```

If all goes well, we'll respond with the oauth_token, which must be passed as a parameter on all API requests.

If you're using Clarity to provide authorization for your application, you'll likely want to call the [GET /v1/me](#GETme) endpoint to obtain information about the current member.

```
  GET https://api.clarity.fm/v1/me?oauth_token=...
```

## Endpoints

#### Members

* [GET /v1/members](#get-v1members)
* [GET /v1/me](#get-v1me)

#### Calls

* [GET /v1/calls/requested](#get-v1callsrequested)
* [GET /v1/calls/callback](#get-v1callscallback)
* [GET /v1/calls/completed](#get-v1callscompleted)
* [GET /v1/calls/canceled](#get-v1callscanceled)
* [GET /v1/calls/:id](#get-v1callsid)
* [POST /v1/calls/:id/connect](#post-v1callsidconnect)
* [POST /v1/calls](#post-v1calls)

#### Needs

* [GET /v1/needs](#get-v1needs)
* [POST /v1/needs](#post-v1needs)
* [POST /v1/needs/:id/fulfill](#post-v1needsidfulfill)

#### Search

* [GET /v1/search](#get-v1search)

#### Organizations

* [GET /v1/organizations](#get-v1organizations)
* [GET /v1/organizations/:id](#get-v1organizationsid)

#### Messages

* [GET /v1/messages](#get-v1messages)
* [GET /v1/messages/:member_id](#get-v1messagesmember_id)
* [POST /v1/messages/:member_id](#post-v1messagesmember_id)

-----------------

## Members

### GET /v1/members

* **id**: _(optional)_ A Clarity id or screen name
* **twitter**: _(optional)_ A twitter handle
* **email**: _(optional)_ An email address

Get member information by id, screen name, Twitter handle or email address.

```
GET /v1/members/danmartell
GET /v1/members/7
GET /v1/members?id=danmartell
GET /v1/members?twitter=danmartell
GET /v1/members?email=dan@larity.fm
```

```javascript
{
  "amount_generated_for_charity": 1137.15,
  "average_call_duration": 750,
  "average_response_time": 89544,
  "bio": "Canadian Entrepreneur.  Founder of Clarity.fm - Angel Investor (15 tech startups). Co-Founder of Flowtown (Acquired 11'), Founder of Spheric Technologies (Acquired 08'), Mentor @ 500Startup, GrowLabs & theC100.org",
  "charity_logo": "http://s3.amazonaws.com/clarityfm-staging/charities/logos/1/original/charity_water.jpeg?1327007697",
  "charity_name": "charity: water",
  "charity_url": "http://www.charitywater.org/",
  "clarity_url": "http://clarity.fm/danmartell",
  "created_at": "2011-10-20T15:18:51Z",
  "expertise_body": "I've been an entrepreneur for over 10+ years.  I've started 5 companies, the first 2 were complete failures and eventually got it right in 2004.  I've bootstrapped technology companies without funding from a small town in Canada, as well as built, raised venture capital and sold a company in San Francisco.  I love technology, product and marketing.",
  "facebook_profile_url": "https://facebook.com/829175116",
  "first_name": "Dan",
  "gender": "male",
  "hourly_rate": "0.0",
  "id": 7,
  "image_url": "http://s3.amazonaws.com/clarityfm-staging/users/pictures/7/profile_square/BIO%20-%20Headshot.jpg?1332179519",
  "last_active_at": "2012-08-02T02:19:59Z",
  "last_name": "Martell",
  "linkedin_profile_url": "http://www.linkedin.com/x/profile/7qvyw2754dt5/8bn1Mfx4Ce",
  "location": "San Francisco, California",
  "profile_page_views_count": 2645,
  "public_social_connections": {
    "facebook": true,
    "linkedin": true,
    "twitter": true
  },
  "screen_name": "danmartell",
  "timezone": "Atlantic Time (Canada)",
  "timezone_label": "ADT",
  "timezone_offset": -3.0,
  "total_calls": 450,
  "twitter_profile_url": "http://twitter.com/account/redirect_by_id?id=10638782",
  "updated_at": "2012-08-03T11:56:53Z",
  "verified_phone_number": true
}
```

### GET /v1/me

[GET /v1/members](#get-v1members) for the current member.

## Calls

### GET /v1/calls/requested

Gets all pending calls where the authenticated member is the seeker.

```
GET /v1/calls/requested
```

```javascript
[
  {
    "created_at": "2012-08-03T12:10:46Z",
    "canceled_by": null,
    "duration": null,
    "expert_id": 7,
    "expires_at": "2012-08-06T12:10:46Z",
    "hourly_rate": 0.0,
    "id": 5802,
    "reason": "Social media marketing",
    "seeker_id": 6,
    "status": "pending",
    "updated_at": "2012-08-03T12:10:46Z"
  }
]
```

### GET /v1/calls/callback

Gets all pending calls where the authenticated member is the expert.  These can be connected with [POST /v1/calls/:id/connect](#post-v1callsidconnect).

```
GET /v1/calls/callback
```

```javascript
[
  {
    "created_at": "2012-08-05T11:16:23Z",
    "canceled_by": null,
    "duration": null,
    "expert_id": 6,
    "expires_at": "2012-08-06T11:16:23Z",
    "hourly_rate": 100.0,
    "id": 5802,
    "reason": "pitching advice",
    "seeker_id": 7,
    "status": "pending",
    "updated_at": "2012-08-05T11:16:23Z"
  }
]
```

### GET /v1/calls/completed

Show the current member's completed calls.  This includes both expert calls and seeker calls.

```
GET /v1/calls/completed
```

```javascript
[
  {
    "created_at": "2012-08-03T12:10:46Z",
    "canceled_by": null,
    "duration": 479,
    "expert_id": 7,
    "expires_at": "2012-08-06T12:10:46Z",
    "hourly_rate": 0.0,
    "id": 5802,
    "reason": "Social media marketing",
    "seeker_id": 6,
    "status": "completed",
    "updated_at": "2012-08-05T11:22:51Z"
  },
  {
    "created_at": "2012-08-05T11:16:23Z",
    "canceled_by": null,
    "duration": 1123,
    "expert_id": 6,
    "expires_at": "2012-08-06T11:16:23Z",
    "hourly_rate": 100.0,
    "id": 5802,
    "reason": "pitching advice",
    "seeker_id": 7,
    "status": "completed",
    "updated_at": "2012-08-09T10:12:16Z"
  }
]
```

### GET /v1/calls/canceled

Show the current member's canceled calls.  This includes both expert calls and seeker calls.

```
GET /v1/calls/canceled
```

```javascript
[
  {
    "created_at": "2012-08-03T12:10:46Z",
    "canceled_by": 7,
    "duration": null,
    "expert_id": 7,
    "expires_at": "2012-08-06T12:10:46Z",
    "hourly_rate": 0.0,
    "id": 5802,
    "reason": "Social media marketing",
    "seeker_id": 6,
    "status": "canceled",
    "updated_at": "2012-08-05T11:22:51Z"
  },
  {
    "created_at": "2012-08-05T11:16:23Z",
    "canceled_by": 7,
    "duration": null,
    "expert_id": 6,
    "expires_at": "2012-08-06T11:16:23Z",
    "hourly_rate": 100.0,
    "id": 5802,
    "reason": "pitching advice",
    "seeker_id": 7,
    "status": "canceled",
    "updated_at": "2012-08-09T10:12:16Z"
  }
]
```

### GET /v1/calls/:id

* **id**: The id of the call to fetch

Get information about the specific call.  Can be called multiple times to track an in progress call.

A call can in one of many different states during its lifetime.  States may be added and removed as we adapt our call flows, but a stable subset of the possible states are:

* pending
* dialing_seeker
* seeker_answered
* dialing_expert
* expert_answered
* seeker_unavailable
* expert_unavailable
* call_completed
* canceled

```
GET /v1/calls/:id
```

```javascript
{
  "created_at": "2012-08-05T11:16:23Z",
  "canceled_by": 7,
  "duration": null,
  "expert_id": 6,
  "expires_at": "2012-08-06T11:16:23Z",
  "hourly_rate": 100.0,
  "id": 5802,
  "reason": "pitching advice",
  "seeker_id": 7,
  "status": "expert_answered",
  "updated_at": "2012-08-09T10:12:16Z"
}
```

### POST /v1/calls/:id/connect

* **id**: The id of the call to connect. Must be from [GET /v1/calls/callback](get-v1callscallback).

Initiate a call. [GET /v1/calls/:id](#get-v1callsid) may be called repeatedly to track its progress.

```
POST /v1/calls/:id/connect
```

```javascript
{
  "created_at": "2012-08-05T11:16:23Z",
  "canceled_by": 7,
  "duration": null,
  "expert_id": 6,
  "expires_at": "2012-08-06T11:16:23Z",
  "hourly_rate": 100.0,
  "id": 5802,
  "reason": "pitching advice",
  "seeker_id": 7,
  "status": "dialing_expert",
  "updated_at": "2012-08-09T10:12:16Z"
}
```

### POST /v1/calls

* **expert_id**: The id of the expert to connect with.
* **expires_in**: The expiry time in hours.
* **reason**: The reason for the call (max 140 chars)

Creates a new call request.

```
POST /v1/calls
```

```javascript
{
  "created_at": "2012-08-05T11:16:23Z",
  "canceled_by": 7,
  "duration": null,
  "expert_id": 6,
  "expires_at": "2012-08-06T11:16:23Z",
  "hourly_rate": 100.0,
  "id": 5802,
  "reason": "pitching advice",
  "seeker_id": 7,
  "updated_at": "2012-08-09T10:12:16Z"
}
```

## Needs

### GET /v1/needs

A list of all posted needs for the members of your organizations.

```
GET /v1/needs
```

```javascript
[
  {
    "created_at": "2012-08-04T00:28:03Z",
    "description": "Would be awesome if you guys added Shiv Singh (Global Head of Digital, PepsiCo Beverages) and Gary Vaynerchuck.",
    "paid": true,
    "requester_id": 771
  }
]
```

### POST /v1/needs

* **description**: Describe your need
* **paid**: Are you willing to pay?


Create a new need, viewable by members of your organizations.

```
POST /v1/needs
```

```javascript
[
  {
    "created_at": "2012-08-04T00:28:03Z",
    "description": "Would be awesome if you guys added Shiv Singh (Global Head of Digital, PepsiCo Beverages) and Gary Vaynerchuck.",
    "paid": true,
    "requester_id": 771
  }
]
```

### POST /v1/needs/:id/fulfill

* **id**: the id of the need to fulfill

**NOTE:** this endpoint hasn't been finalized and will likely change in the future.

Fulfills a need by creating a pending call request with the member who posted the need.

```
POST /v1/needs/:id/fulfill
```~

```javascript
{
  "created_at": "2012-08-03T12:10:46Z",
  "canceled_by": null,
  "duration": null,
  "expert_id": 7,
  "expires_at": "2012-08-06T12:10:46Z",
  "hourly_rate": 0.0,
  "id": 5802,
  "reason": "Social media marketing",
  "seeker_id": 6,
  "status": "pending",
  "updated_at": "2012-08-03T12:10:46Z"
}
```

## Organizations

### GET /v1/organizations

Get a list of organizations that the current member is part of.

```
GET /v1/organizations
```

``` javascript
[
  {
    "name": "Startup Festival",
    "description": "International Startup Festival",
    "featured": true,
    "free_calls": false,
    "clarity_url": "http://clarity.fm/startupweekend",
    "image_url": "https://s3.amazonaws.com/clarityfm-production/networks/logos/17/list_square/logo.png?1340721323",
    "member_ids": [172, 5840, 10204]
  }
]
```

### GET /v1/organizations/:id

* **id**: the id of the organization to fetch

Get an organization with a list of the members that are part of the organization for the current user.

```
GET /v1/organizations/:id
```

```javascript
{
  "name": "Startup Festival",
  "description": "International Startup Festival",
  "featured": true,
  "free_calls": false,
  "clarity_url": "http://clarity.fm/startupweekend",
  "image_url": "https://s3.amazonaws.com/clarityfm-production/networks/logos/17/list_square/logo.png?1340721323",
  "members": [
    {
      "bio": "Founding member at Clarity. Code aficionado. Lives in an igloo and hunts beavers for food. Occasional unicycle riding scuba diving hippie. Current life ambition: finish reading Gödel, Escher, Bach.",
      "clarity_url": "http://clarity.fm/justinvaillancourt",
      "first_name": "Justin",
      "hourly_rate": 0.0,
      "id": 66,
      "image_url": "http://s3.amazonaws.com/clarityfm-staging/users/pictures/66/profile_square/picture?1331917625",
      "last_name": "Vaillancourt",
      "location": "Moncton, New Brunswick",
      "name": "Justin Vaillancourt",
      "screen_name": "justinvaillancourt"
    },
    {
      "bio": "Like all things web development - rails, ruby, javascript, html, css, user experience. Working on Clarity to help you help others.",
      "clarity_url": "http://clarity.fm/vincentroy",
      "first_name": "Vincent",
      "hourly_rate": 20.0,
      "id": 940,
      "image_url": "http://s3.amazonaws.com/clarityfm-staging/users/pictures/940/profile_square/vincent-avatar.jpg?1339414662",
      "last_name": "Roy",
      "location": "Moncton, New Brunswick",
      "name": "Vincent Roy",
      "screen_name": "vincentroy"
    }
  ]
}
```


## Messages

### GET /v1/messages

Get a list of conversations that the current member is part of.

```
GET /v1/messages
```

```javascript
[
  {
    "unread_message_count": 1,
    last_message: {
      "created_at": "2012-08-05T11:16:23Z",
      "read": false,
      "message": "Not much",
      "member_id": 940
    },
    other_member: {
      "bio": "Like all things web development - rails, ruby, javascript, html, css, user experience. Working on Clarity to help you help others.",
      "clarity_url": "http://clarity.fm/vincentroy",
      "first_name": "Vincent",
      "hourly_rate": 20.0,
      "id": 940,
      "image_url": "http://s3.amazonaws.com/clarityfm-staging/users/pictures/940/profile_square/vincent-avatar.jpg?1339414662",
      "last_name": "Roy",
      "location": "Moncton, New Brunswick",
      "name": "Vincent Roy",
      "screen_name": "vincentroy"
    }
  }
]
```


### GET /v1/messages/:member_id

Get the last 50 messages that were sent between the current member and the specified member.

```
GET /v1/messages/940
```

```javascript
{
  other_member: {
    "bio": "Like all things web development - rails, ruby, javascript, html, css, user experience. Working on Clarity to help you help others.",
    "clarity_url": "http://clarity.fm/vincentroy",
    "first_name": "Vincent",
    "hourly_rate": 20.0,
    "id": 940,
    "image_url": "http://s3.amazonaws.com/clarityfm-staging/users/pictures/940/profile_square/vincent-avatar.jpg?1339414662",
    "last_name": "Roy",
    "location": "Moncton, New Brunswick",
    "name": "Vincent Roy",
    "screen_name": "vincentroy"
  },
  messages: [
    {
      "created_at": "2012-08-05T11:16:23Z",
      "read": false,
      "message": "Not much",
      "member_id": 940
    },
    {
      "created_at": "2012-08-04T14:34:23Z",
      "read": true,
      "message": "Hey, what's up?",
      "member_id": 172
    }
  }
]
```

### POST /v1/messages/:member_id

Send a message from the current member to the specified member.

* **message**: The message

```
POST /v1/messages/940
```

```javascript
{
  "created_at": "2012-08-05T11:16:23Z",
  "read": false,
  "message": "Not much",
  "member_id": 940
}
```


## Search

### GET /v1/search

Search for an Expert. Returns 10 results at a time.

* **q**: A search term
* **offset**: _(optional)_ An offset, used to page through the results

```
GET /v1/search?q=moncton
GET /v1/search/moncton
```

```javascript
{
  "matches": 73,
  "members": [
    {
      "bio": "Founding member at Clarity. Code aficionado. Lives in an igloo and hunts beavers for food. Occasional unicycle riding scuba diving hippie. Current life ambition: finish reading Gödel, Escher, Bach.",
      "clarity_url": "http://clarity.fm/justinvaillancourt",
      "first_name": "Justin",
      "hourly_rate": 0.0,
      "id": 66,
      "image_url": "http://s3.amazonaws.com/clarityfm-staging/users/pictures/66/profile_square/picture?1331917625",
      "last_name": "Vaillancourt",
      "location": "Moncton, New Brunswick",
      "name": "Justin Vaillancourt",
      "screen_name": "justinvaillancourt"
    },
    {
      "bio": "Like all things web development - rails, ruby, javascript, html, css, user experience. Working on Clarity to help you help others.",
      "clarity_url": "http://clarity.fm/vincentroy",
      "first_name": "Vincent",
      "hourly_rate": 20.0,
      "id": 940,
      "image_url": "http://s3.amazonaws.com/clarityfm-staging/users/pictures/940/profile_square/vincent-avatar.jpg?1339414662",
      "last_name": "Roy",
      "location": "Moncton, New Brunswick",
      "name": "Vincent Roy",
      "screen_name": "vincentroy"
    }
  ]
}
```


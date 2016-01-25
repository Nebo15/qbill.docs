---
title: QBill API Reference

language_tabs:
  - shell

search: true
---

# Introduction

Welcome to the QBilling API. It will provide you access to universal billing system that is usage-agnostic, what makes it suitable for financial companies for storing accounts and transferring money, games with inner currency or anyone else who have balance and any actions with it.

# Key Features

### Data Retention Policy

We believe that vendor-lock is a bad thing, thats why you are free to download all data from your account in a JSON format.

Also you can remove all you data from our servers. After your confirm data remove process in Dashboard we will keep everything for additional 30 days, so your account will be protected from accidental removals. After 30 days all data will be scrubbed from our servers and impossible to restore.

### API-oriented

We are API-centered, that means that we are trying to make it simple, easy to understand, and yet very powerful.

### Crossintegrations

We support hustle-free integration with oAuth providers. See more at [Authentication](#authentication) section.

Also we support custom webhook integrations in a SOA-style.

# Interacting with API

Our API is organized around [REST](http://en.wikipedia.org/wiki/Representational_State_Transfer). It has predictable, resource-oriented URLs, and uses HTTP response codes to indicate API errors. We use built-in HTTP features, like HTTP authentication and HTTP verbs, which are understood by off-the-shelf HTTP clients. We support cross-origin resource sharing, allowing you to interact securely with our API from a client-side web application (though you should never expose your secret API key in any public website's client-side code).

### HTTP Verbs

As per RESTful design patterns, API implements following HTTP verbs:

- ```HEAD``` - Can be issued against any resource to get just the HTTP header info;
- ```GET``` - Read resources;
- ```POST``` - Create new resources;
- ```PUT``` - Replace resources or collections. For PUT requests with no body attribute, be sure to set the Content-Length header to zero;
- ```DELETE``` - Remove resources.


## Response structure

Response consist of 4 main objects in root:

- ```meta``` - URL of the requested resource; current status; error and error messages; root object type; idempotency key; request id.
- ```paging``` - pagination data;
- ```urgent``` - notifications and counters;
- ```data``` - root response data object;
- ```sandbox``` - data provided by ```sandbox``` environment, for eg. otp token.

```json
{
  "meta": {
    "code": "XXX",
    "url": "https://qbill.ly/transactions/",
    "type": "list",
    "error": {
      "type": "form_validation_failed",
      "params": {"submit_date": "2015-01-24"},
      "fields": [
        {"field_id": "username", "rule": "required"},
        {"field_id": "email", "rule": "empty"}
      ],
      "message": "You specified incorrect content type. See https://docs.qbill.ly/#content-type."
    },
    "request_id": "qudk48fFlaP",
    "idempotency_id": "iXXekd88DKqo"
  },
  "urgent": {
    "notifications": ["Read new emails!"],
    "counters": {
      "unseen_payments": 10
    }
  }
  "data": {

  },
  "paging": {
    "cursors": {
      "after": "MTAxNTExOTQ1MjAwNzI5NDE=",
      "before": "NDMyNzQyODI3OTQw"
    },
    "has_more": true,
    "limit": 50
  },
  "sandbox": {
    "otp_code": "39384"
  }
}

```

All responses have ```object``` field that contains object type. For eg. transactions have ```object``` that equals to ```transaction```.

## Authentication

To use our service you need to authenticate your application. Additionally you can use our oAuth cross-integration for authenticating your clients.

### Application

Authenticate your account when using the API by including your secret API key in the request. You can manage your API keys in the dashboard.

Authentication to the API is performed via [HTTP Basic Auth](http://en.wikipedia.org/wiki/Basic_access_authentication). Provide your API key as the basic auth username value. You do not need to provide a password.

<aside class="warning">
Your API keys carry all the privileges, so be sure to keep them secret! Do not share your secret API keys in publicly accessible areas such GitHub, client-side code, and so forth.
</aside>

```curl
curl https://example.com/resource \
   -u WgLodNU5wCdbSw4f:
```

### Client

(TODO: Move this to pre-flight webhooks.)

This is an option. To make it work specify oAuth provider in your Dashboard. When "Client Authentication" is turned on, our API expects additional key provided as HTTP Basic Auth password.

<aside class="notice">
Requests without oAuth token will return HTTP 401 error code.
</aside>

Currently we support 3 oAuth providers: Facebook, Google and a custom endpoint, that can be entered by you.

```curl
curl https://example.com/resource \
   -u WgLodNU5wCdbSw4f:<oAuthToken>
```

After receiving oAuth token we will send a GET request to selected oAuth endpoint to validate this token. Our response differs for every response made by oAuth endpoint:

oAuth HTTP Code | Action
--------- | -----------
2XX | Continue processing your request.
301, 302, 307, 308 | Follow the redirect. And continue checking response code.
403 | Return HTTP 403 Forbidden
401, 402 and other 4XX | Return HTTP 401 Unauthorized
5XX | Return HTTP 412 Precondition Failed


```
GET <oAuthProviderURI>?access_token=<oAuthToken>&action=<APIProjectID>.<APIUserID>.<APIEntity>.<HTTPVerb>
```

## Versioning

All API calls should be made with a X-API-Version header which guarantees that your call is using the correct API version. Version is passed in as a date (UTC) of the implementation in YYYY-MM-DD format.

If no version is passed, the newest will be used and a warning will be shown. Under no circumstances should you always pass in the current date as that will return the current version which might break your implementation.

## Pagination

All top-level API resources with root type ```list``` have support of pagination over a "list" API methods. These methods share a common structure, taking at least these three parameters: ```limit```, ```starting_after```, and ```ending_before```.

API utilizes cursor-based pagination via the ```starting_after``` and ```ending_before``` parameters. Both take an existing object ID value (see below). The ```ending_before``` parameter returns objects created before the named object, in descending chronological order. The ```starting_after``` parameter returns objects created after the named object, in ascending chronological order. If both parameters are provided, only ending_before is used.

Arguments:

- ```limit``` (optional) - A limit on the number of objects to be returned, between 1 and 100. Default: 50;
- ```starting_after``` (optional) - A cursor for use in pagination. ```starting_after``` is an object ID that defines your place in the list. For instance, if you make a list request and receive 100 objects, ending with ```obj_foo```, your subsequent call can include ```starting_after=obj_foo``` in order to fetch the next page of the list;
- ```ending_before``` (optional) - A cursor for use in pagination. ```ending_before``` is an object ID that defines your place in the list. For instance, if you make a list request and receive 100 objects, starting with ```obj_bar```, your subsequent call can include ```ending_before=obj_bar``` in order to fetch the previous page of the list.

## Content Type

We support 3 content types that should be send in a ```Content-Type``` header:

- ```application/json``` - to response in JSON format;
- ```application/xml``` - to receive response in XML format;
- ```text/csv``` - to receive response in CSV format.

<aside class="notice">
For CSV we will return only content in a root ```data``` object, without any additional meta (for eg. pagination) to make it prettier in spreadsheets viewers.
</aside>

To simplify documentation all samples will be provided with JSON content type responses.

## Errors

All errors is returned in JSON format if another ```Content-Type``` is not specified. This means that your will receive JSON for requests with HTTP 415 code when incorrect ```Content-Type``` is provided.

### Application Error Types

> Parameter | Description
--------- | -----------
ID | The ID to retrieve

### HTTP status codes

> HTTP Code | Description
--------- | -----------
```200``` | Everything worked as expected;
```400``` | Bad Request. The request was unacceptable, often due to missing a required parameter. Or request contains invalid JSON;
```401``` | Unauthorized. No valid API key provided or API key doesn't match project;
```402``` | The parameters were valid but the request failed;
```404``` | Not Found. The requested resource doesn't exist;
```415``` | Incorrect Content-Type HTTP header;
```429``` | Too Many Requests. Rate limit is exceeded;
```500```, ```502```, ```503```, ```504``` | Server Errors. Something went wrong on our end. (These are rare.)

## Rate Limits (Throttling)

We throttle our APIs by default to ensure maximum performance for all developers.

Rate limiting of the API is primarily considered on a per-user basis — or more accurately, per access token in your control. Rate limits are determined globally for the entire application.

Right now rate limit is 5000 calls every 15 minutes, but this value may be adjusted at our discretion.

All responses have 3 additional headers:

HTTP Header | Description
--------- | -----------
X-RateLimit-Limit | Current rate limit for your application;
X-RateLimit-Remaining | Remaining rate limit for your application;
X-RateLimit-Reset | The time at which the current rate limit window resets in [UTC epoch seconds](http://en.wikipedia.org/wiki/Unix_time).

<aside class="notice">
When limit is exceeded all requests will return ```HTTP 402``` status code.
</aside>

```
X-RateLimit-Limit: 5000
X-RateLimit-Remaining: 4966
X-RateLimit-Reset: 1372700873
```

## Cross Origin Resource Sharing

The API supports Cross Origin Resource Sharing (CORS) for AJAX requests from any origin. You can read the [CORS W3C Recommendation](http://www.w3.org/TR/cors/), or [this intro](http://code.google.com/p/html5security/wiki/CrossOriginRequestSecurity) from the HTML 5 Security Guide.

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: Authorization, Content-Type, If-Match, If-Modified-Since, If-None-Match, If-Unmodified-Since, X-Requested-With
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Expose-Headers: ETag, Link, X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset, X-OAuth-Scopes, X-Accepted-OAuth-Scopes
Access-Control-Allow-Credentials: true
```

## Timezones

Some requests allow for specifying timestamps or generate timestamps with time zone information. We apply the following rules, in order of priority, to determine timezone information for API calls.

Explicitly provide an ISO 8601 timestamp with timezone information to use this feature.

It is possible to supply a Time-Zone header which defines a timezone according to the list of names from the [Olson database](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).

```shell
curl -H "Time-Zone: Europe/Amsterdam" ..
curl -H "Time-Zone: Europe/Kiev" ..
```

This means that we will return all datetime fields in corresponding timezone. Default timezone is UTC.

## Request ID's

Each API request has an associated request identifier. You can find this value in the response headers, under ```X-Request-ID``` or inside ```meta``` object in response data. You can also find request identifiers in the URLs of individual request logs in your Dashboard. If you need to contact us about a specific request, providing the request identifier will ensure the fastest possible resolution.

## Limiting Response Fields

By default, all the fields in a node are returned when you make a query. You can choose the fields you want returned with the "fields" query parameter. This is really useful for making your API calls more efficient and fast.

```curl
/resource?fields=id,name,balance
```

## Expanding Response Fields

Many objects contain the ID of a related object in their response properties. For example a ```Transfer``` can have an associated ```User``` object linked by a ```sender``` field. Those objects can be expanded inline with the ```expand``` request parameter. Objects that can be expanded are noted in this documentation. This parameter is available on all API requests, and applies to the response of that request only.

You can nest expand requests with the dot property. For example, requesting ```sender.payments``` on a ```Transfer``` list will expand the ```sender``` property into a full ```User``` object, and will then expand the ```payments``` property of that ```User``` into a full ```Transactions``` collection.

You can expand multiple objects at once by identifying multiple items divided by comma.

You can expand object by querying list, expands will be applied on all matched elements.

Expanded collections return up to 25 elements, limit can be specified in brackets.

```curl
/users?expand=payments(5)
```

## Ordering Lists and Collections

By default, all collections are ordered in ascending chronological order. You can specify different order by providing the "order" query parameter.

```curl
/resource?order=payment.created_at(reverse_chronological)
```

## Testing

All accounts have test project that is created for by default. Just use test API secret for your test environment. We don't have any policy for data retention in test accounts and we can drop all data once a while. You can do it manually from your dashboard.

# Accounts

Account is a one a base entities that represents any object with a balance. This balance can be in any currency. If you have multiple currencies, they will be converted automatically.

## List all Accounts

List all accounts with a filter. Filter can represent any field that is available in the response, except entities that needs to be expanded.

(TODO: Add less than, greater than, text search).

```
GET /v1/accounts?filters=[]
```

> For analytical purposes response can be grouped by a field

```
GET /v1/accounts?filters=[]&group_by=[]
```

> For analytical purposes response can return aggregation counts

```
GET /v1/accounts?filters=[]&get_count_by=[]
```

## Create an Account

Account can have any custom metadata, for eg. ```account_type``` that can be ```user``` and ```system```. Also you can link your internal ID to a user to find them by an ```external_id``` field.

(TODO: Specify a custom ID?)

```
POST /v1/accounts
```

## Get all Account data

```
GET /v1/accounts/:id
```

## Funding an Account

(TODO: Create a default project-wide accounts that can be funded, and all user accounts should be funded by a transfer.)

(TODO: Move this to a Fundings entity?)

(TODO: Allow subpayments on funding to charge front fee?)

We recommend to fund only system accounts and to avoid direct funding for user-related accounts. This simplifies accounting of your system's money flow.

```
POST /v1/accounts/:account_id/fund
{
  total: 1000
}
```

## Freezing and Unfreezing an Account

Accounts can't be deleted, but can be freezed to prevent its future usage.

```
POST /v1/accounts/:id/freeze
```

You can unfreeze account to continue using it later.

```
POST /v1/accounts/:id/unfreeze
```

# Transactions

Transaction is a main operation with account balance that covers both payments and transfers.

Transactions can be ```internal``` and ```external```. We recommend you to skip ```internal``` transactions for all lists that will be visible for your customers, since then carry only technical purposes.

## Creating Transaction

There are two flows for creating a transaction:
- Instant - create hold and immediately convert it to charge;
- Delayed - create hold and manually convert it to charge.

A single payment can have multiple charges that will look like a single transaction for an account, but it will create multiple technical transactions. This is useful to charge fees.

### One-Step Transaction

```
POST /v1/accounts/:account_id/transactions
{
  total: 100,
  charges: [
    {subtotal: 90, destination: "<service_account_id>", meta: {service_id: 1, service_name: 'Cellular Topup'}}
    {subtotal: 10, destination: "<fees_account_id>", meta: {for: "service_payment", service_id 1}}
  ],
  meta: {}
}
```

### Multi-Step Transaction

(TODO: Transfer with a currency conversion.)

Hold money on target account.

```
POST /v1/accounts/:account_id/holds
```

> While money is on-hold you can change any payment details, for eg. to refund some part of money

```
PUT /v1/holds/:transaction_id
{
  total: 20.00
}
```

> After hold payment can be completed to commit balance change and turn hold into charge or declined to remove hold and return funds to available balance.

```
POST /v1/transactions/:transaction_id/complete
```

```
POST /v1/transactions/:transaction_id/decline
```

## List all Transactions

List all transactions with a filter. Filter can represent any field that is available in the response, except entities that needs to be expanded.

(TODO: Add less than, greater than, text search).

```
GET /v1/transactions?filters=[]
```

> For analytical purposes response can be grouped by a field

```
GET /v1/transactions?filters=[]&group_by=[]
```

> For analytical purposes response can return aggregation counts

```
GET /v1/transactions?filters=[]&get_count_by=[]
```

## Rollback a Transaction

To a rollback we will create a new Transaction with a ```type=internal``` and a ```refference_id=<originalTransactionID>``` fields.

## Create Refund for a Transaction

Refund is similar to a Rollback, but you need to specify refund total for every account that received funds. This allows you to refund funds by keeping the fees or to refund it with all the fees.

# Holds

# Currencies and Rates

# Events

# Webhooks

## General Infromation

### Events

All actions will trigger creation of notification, that will be send to all your webhooks that is subscribed for this kind of events.

If notification is failed to deliver we will retry it for the next 24 hours. Retry time is increased each time request is made: 5, 15, 30 and 60 minutes. (60 minutes is a maximum timeout.)

### Pre-Flight Requests

To make API easier to integrate with other systems we can synchronously pass all requested data and all available metadata to endpoint specified in a Dashboard.

Service will act differently based on HTTP response of this endpoint:

oAuth HTTP Code | Action
--------- | -----------
2XX | Continue processing your request.
301, 307, 308 | Follow the redirect. And continue checking response code.
401, 402, 4XX | Return same HTTP code
5XX and all other codes | Return HTTP 412 Precondition Failed

This webhooks is extremely useful for antifraud purposes, when another system needs to validate transaction before they are completed.

# Requests

# SQL Queries

# Backups

# Best Practices

## Currency flows

We encourage you to use right accounting models inside your system. Separate all accounts by a ```type```, lets call them ```system``` and ```client``` accounts.

This will help you to calculate losses and revenue in a right, predictable way.

## Token and ID lengths and formats

In order to avoid interruptions in processing, it's best to make minimal assumptions about what our gateway-generated tokens and identifiers will look like in the future. The length and format of these identifiers – including payment method tokens and transaction IDs – can change at any time, with or without advance notice. However, it is safe to assume that they will remain 1 to 64 alphanumeric characters.

## Geographic Redundancy and Optimization

To ensure that you will always have the lowest response time we can provide, we are automatically detecting nearest datacenter to you, so all your projects have master servers in it. To migrate data to different region please contact our support team.

## Idempotent Requests

The API supports idempotency for safely retrying write requests without accidentally performing the same operation twice. For example, if a request to create a charge fails due to a network connection error, you can retry the request with the same idempotency key to guarantee that only a single charge is created.

To perform an idempotent request, attach a unique key to any ```POST```, ```DELETE``` or ```PUT``` request made to the API via the ```Idempotency-Key: <key>``` header.

How you create unique keys is completely up to you. We suggest using random strings or UUIDs. We'll always send back the same response for requests made with the same key. However, you cannot use the same key with different request parameters. The keys expire after 24 hours.

## Conditional requests

Most responses return an ETag header. Many responses also return a Last-Modified header. You can use the values of these headers to make subsequent requests to those resources using the ```If-None-Match``` and ```If-Modified-Since``` headers, respectively. If the resource has not changed, the server will return a 304 Not Modified.

Also note: making a conditional request and receiving a 304 response does not count against your Rate Limit, so we encourage you to use it whenever possible.

## SSL certificates

We recommend that all users obtain an SSL certificate and serve any data that is stored in our service over HTTPS.

We don't have a specific preferred vendor for SSL certificates, but we recommend that you stick with a well-known provider (e.g. Network Solutions, GoDaddy, Namecheap). Generally speaking, most certificates will be similar, so it's up to you to determine what fits your needs best.

## HTTP Redirects

API uses HTTP redirection where appropriate. Clients should assume that any request may result in a redirection. Receiving an HTTP redirection is not an error and clients should follow that redirect. Redirect responses will have a Location header field which contains the URI of the resource to which the client should repeat the requests.

Status codes:

- ```301``` - Permanent redirection. The URI you used to make the request has been superseded by the one specified in the Location header field. This and all future requests to this resource should be directed to the new URI.
- ```302, 307``` - Temporary redirection. The request should be repeated verbatim to the URI specified in the Location header field but clients should continue to use the original URI for future requests.

Other redirection status codes may be used in accordance with the HTTP 1.1 spec.

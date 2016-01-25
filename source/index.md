---
title: QBill API Reference

language_tabs:
  - shell

search: true
---

# Introduction

Welcome to the QBilling API. It will provide you access to universal billing system that is usage-agnostic, what makes it suitable for financial companies for storing accounts and transferring money, games with inner currency or anyone else who have balance and any actions with it.

# Data Retention Policy

We believe that vendor-lock is a bad thing, thats why you are free to download all data from your account in JSON format and to remove all you data from our servers.

After your confirm data remove process we will keep everything for additional 30 days, so your account will be protected from accidental removals. After 30 days all data will be scrubbed for our servers and impossible to restore.

# Interacting with API

Our API is organized around [REST](http://en.wikipedia.org/wiki/Representational_State_Transfer). It has predictable, resource-oriented URLs, and uses HTTP response codes to indicate API errors. We use built-in HTTP features, like HTTP authentication and HTTP verbs, which are understood by off-the-shelf HTTP clients. We support cross-origin resource sharing, allowing you to interact securely with our API from a client-side web application (though you should never expose your secret API key in any public website's client-side code).

## Response structure

Response consist 4 main objects in root:
- ```meta``` - URL of the requested resource; current status; error and error messages; root object type; idempotency key; request id; .
- ```paging``` - pagination data
- ```urgent``` - notifications and counters;
- ```data``` - root response data object;
- ```sandbox``` - data provided by ```sandbox``` environment, for eg. otp token.

, status, errors, test, type (collection), has_more, notifications, counters

```json
{
  "meta": {
    "code": "200",
    "url": "https://qbill.ly/transactions/",
    "type": "list",
    "error": {
      "type": "some_error_type",
      "message": "No records at specified date in transactions collection",
      "param": "2015-01-24"
    }
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
    "has_more": false,
    "limit": 50
  },
  "sandbox": {
    "otp_code": "39384"
  }
}

```

All responses have ```object``` field that contains object type. For eg. collections have ```object``` that equals to ```collection```.

## Authentication

Authenticate your account when using the API by including your secret API key in the request. You can manage your API keys in the dashboard.

! Your API keys carry many privileges, so be sure to keep them secret! Do not share your secret API keys in publicly accessible areas such GitHub, client-side code, and so forth.

Authentication to the API is performed via [HTTP Basic Auth](http://en.wikipedia.org/wiki/Basic_access_authentication). Provide your API key as the basic auth username value. You do not need to provide a password.

```curl
curl https://example.com/resource \
   -u WgLodNU5wCdbSw4f:
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
- ``` ``` (optional) - A cursor for use in pagination. ```ending_before``` is an object ID that defines your place in the list. For instance, if you make a list request and receive 100 objects, starting with ```obj_bar```, your subsequent call can include ```ending_before=obj_bar``` in order to fetch the previous page of the list.

## HTTP Verbs

As per RESTful design patterns, API implements following HTTP verbs:
- ```HEAD``` - Can be issued against any resource to get just the HTTP header info;
- ```GET``` - Read resources;
- ```POST``` - Create new resources;
- ```PUT``` - Replace resources or collections. For PUT requests with no body attribute, be sure to set the Content-Length header to zero;
- ```DELETE``` - Remove resources.

## HTTP Redirects

API uses HTTP redirection where appropriate. Clients should assume that any request may result in a redirection. Receiving an HTTP redirection is not an error and clients should follow that redirect. Redirect responses will have a Location header field which contains the URI of the resource to which the client should repeat the requests.

Status codes:
- ```301``` - Permanent redirection. The URI you used to make the request has been superseded by the one specified in the Location header field. This and all future requests to this resource should be directed to the new URI.
- ```302, 307``` - Temporary redirection. The request should be repeated verbatim to the URI specified in the Location header field but clients should continue to use the original URI for future requests.

Other redirection status codes may be used in accordance with the HTTP 1.1 spec.

## Content Types

We support 3 content types:
- ```application/json``` - to response in JSON format;
- ```application/xml``` - to receive response in XML format;
- ```text/csv``` - to receive response in CSV format.

Internally we stick with JSON, and convert it to XML or CSV when necessary, so response structure for JSON and XML content types will be pretty much the same. But for CSV we won't return any additional meta (for eg. pagination) to make it prettier in spreadsheets viewers.

To simplify documentation all samples will be provided with JSON ```content-type``` responses.

## Errors

All errors is returned in JSON format if another ```content-type``` is not specified. This means that your will receive JSON for requests with incorrect ```content-type``` header.

### Application Error Types

Parameter | Description
--------- | -----------
ID | The ID to retrieve

### HTTP status codes

- ```200``` - Everything worked as expected;
- ```400``` - Bad Request. The request was unacceptable, often due to missing a required parameter. Or request contains invalid JSON;
- ```401``` - Unauthorized. No valid API key provided or API key doesn't match project;
- ```402``` - The parameters were valid but the request failed;
- ```404``` - Not Found. The requested resource doesn't exist;
- ```415``` - Incorrect Content-Type HTTP header;
- ```429``` - Too Many Requests. Rate limit is exceeded;
- ```500, 502, 503, 504``` - Server Errors. Something went wrong on our end. (These are rare.)

## Rate Limits (Throttling)

We throttle our APIs by default to ensure maximum performance for all developers.

Rate limiting of the API is primarily considered on a per-user basis — or more accurately, per access token in your control. Rate limits are determined globally for the entire application.

Right now rate limit is 5000 calls every 15 minutes, but this value may be adjusted at our discretion.

All responses have 3 additional headers:
- ```X-RateLimit-Limit``` - current rate limit for your application;
- ```X-RateLimit-Remaining``` - remaining rate limit for your application;
- ```X-RateLimit-Reset``` - the time at which the current rate limit window resets in [UTC epoch seconds](http://en.wikipedia.org/wiki/Unix_time).

When limit is exceeded all requests will return ```HTTP 402``` status code.

```json
X-RateLimit-Limit: 5000
X-RateLimit-Remaining: 4966
X-RateLimit-Reset: 1372700873
```

## Conditional requests

Most responses return an ETag header. Many responses also return a Last-Modified header. You can use the values of these headers to make subsequent requests to those resources using the ```If-None-Match``` and ```If-Modified-Since``` headers, respectively. If the resource has not changed, the server will return a 304 Not Modified.

Also note: making a conditional request and receiving a 304 response does not count against your Rate Limit, so we encourage you to use it whenever possible.

## Cross Origin Resource Sharing

The API supports Cross Origin Resource Sharing (CORS) for AJAX requests from any origin. You can read the [CORS W3C Recommendation](http://www.w3.org/TR/cors/), or [this intro](http://code.google.com/p/html5security/wiki/CrossOriginRequestSecurity) from the HTML 5 Security Guide.

```json
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: Authorization, Content-Type, If-Match, If-Modified-Since, If-None-Match, If-Unmodified-Since, X-QBill-OTP, X-Requested-With
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Expose-Headers: ETag, Link, X-QBill-OTP, X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset, X-OAuth-Scopes, X-Accepted-OAuth-Scopes
Access-Control-Allow-Credentials: true
```

## Timezones

Some requests allow for specifying timestamps or generate timestamps with time zone information. We apply the following rules, in order of priority, to determine timezone information for API calls.

Explicitly provide an ISO 8601 timestamp with timezone information to use this feature.

It is possible to supply a Time-Zone header which defines a timezone according to the list of names from the [Olson database](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).

```json
curl -H "Time-Zone: Europe/Amsterdam" ..
curl -H "Time-Zone: Europe/Kiev" ..
```

This means that we generate a timestamp for the moment your API call is made in the timezone this header defines. For example, the Contents API generates a git commit for each addition or change and uses the current time as the timestamp. This header will determine the timezone used for generating that current timestamp.

_Using the last known timezone for the user_

If no Time-Zone header is specified and you make an authenticated call to the API, we use the last known timezone for the authenticated user. The last known timezone is updated whenever you browse the GitHub website.

_UTC_

If the steps above don't result in any information, we use UTC as the timezone to create the git commit.

## Request IDs

Each API request has an associated request identifier. You can find this value in the response headers, under Request-Id. You can also find request identifiers in the URLs of individual request logs in your Dashboard. If you need to contact us about a specific request, providing the request identifier will ensure the fastest possible resolution.

## Expanding Objects

Many objects contain the ID of a related object in their response properties. For example, a Charge may have an associated Customer ID. Those objects can be expanded inline with the expand request parameter. Objects that can be expanded are noted in this documentation. This parameter is available on all API requests, and applies to the response of that request only.

You can nest expand requests with the dot property. For example, requesting invoice.customer on a charge will expand the invoice property into a full Invoice object, and will then expand the customer property on that invoice into a full Customer object.

You can expand multiple objects at once by identifying multiple items in the expand array.

## Limiting response fields

By default, all the fields in a node are returned when you make a query. You can choose the fields you want returned with the "fields" query parameter. This is really useful for making your API calls more efficient and fast.

```/resource?fields=id,name,balance```

## Ordering response data

By default, all collections are ordered in ascending chronological order. You can specify different order by providing the "order" query parameter.

```/resource?order=payment.created_at(reverse_chronological)```

## Testing

All accounts have test project that is created for by default. Just use test API secret for your test environment. We don't have any policy for data retention in test accounts and we can drop all data once a while. You can do it manually from your dashboard.

## Webhooks

We send notifications.

## Geographic Optimization

To ensure that you will always have the lowest response time we can provide, we are automatically detecting closest datacenter to you, so all your projects have master servers in it. To migrate data to different region please contact our support team.

# Best Practices

## Token and ID lengths and formats

In order to avoid interruptions in processing, it's best to make minimal assumptions about what our gateway-generated tokens and identifiers will look like in the future. The length and format of these identifiers – including payment method tokens and transaction IDs – can change at any time, with or without advance notice. However, it is safe to assume that they will remain 1 to 64 alphanumeric characters.

## Idempotent Requests

The API supports idempotency for safely retrying write requests without accidentally performing the same operation twice. For example, if a request to create a charge fails due to a network connection error, you can retry the request with the same idempotency key to guarantee that only a single charge is created.

To perform an idempotent request, attach a unique key to any ```POST```, ```DELETE``` or ```PUT``` request made to the API via the ```Idempotency-Key: <key>``` header.

How you create unique keys is completely up to you. We suggest using random strings or UUIDs. We'll always send back the same response for requests made with the same key. However, you cannot use the same key with different request parameters. The keys expire after 24 hours.


## SSL certificates

We recommend that all users obtain an SSL certificate and serve any data that is stored in our service over HTTPS.

We don't have a specific preferred vendor for SSL certificates, but we recommend that you stick with a well-known provider (e.g. Network Solutions, GoDaddy, Namecheap). Generally speaking, most certificates will be similar, so it's up to you to determine what fits your needs best.

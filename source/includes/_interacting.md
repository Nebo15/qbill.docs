# Interacting with API

Our API is organized around [REST](http://en.wikipedia.org/wiki/Representational_State_Transfer). It has predictable, resource-oriented URLs, and uses HTTP response codes to indicate API errors. We use built-in HTTP features, like HTTP authentication and HTTP verbs, which are understood by off-the-shelf HTTP clients. We support cross-origin resource sharing, allowing you to interact securely with our API from a client-side web application (though you should never expose your secret API key in any public website's client-side code).

### HTTP Verbs

As per RESTful design patterns, API implements following HTTP verbs:

- ```HEAD``` - Can be issued against any resource to get just the HTTP header info.
- ```GET``` - Read resources.
- ```POST``` - Create new resources.
- ```PUT``` - Replace resources or collections. For PUT requests with no body attribute, be sure to set the Content-Length header to zero.
- ```DELETE``` - Remove resources.

## Response structure

Response consist of 5 main root objects:

- ```meta``` - URL of the requested resource (```url``` ); current status (```code```); error and error messages (```error```); root object type (```type```); idempotency key (```idempotency_id```); request id (```request_id```).
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
    "unseen_payments": 10
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
Your API keys carry all the privileges, so be sure to keep them secret! Do not share your secret API keys in publicly accessible areas such GitHub, client-side code, and so forth. Paranoid people even don't store it in the configuration files, by keeping them in [memcache](https://en.wikipedia.org/wiki/Memcached).
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
403 | Return HTTP 403 Forbidden.
401, 402 and other 4XX | Return HTTP 401 Unauthorized.
5XX | Return HTTP 412 Precondition Failed.


```
GET <oAuthProviderURI>?access_token=<oAuthToken>&action=<APIProjectID>.<APIUserID>.<APIEntity>.<HTTPVerb>
```

## Pagination

All top-level API resources with root type ```list``` have support of pagination over a "list" API methods. These methods share a common structure, taking at least these three parameters: ```limit```, ```starting_after```, and ```ending_before```.

API utilizes cursor-based pagination via the ```starting_after``` and ```ending_before``` parameters. Both take an existing object ID value (see below). The ```ending_before``` parameter returns objects created before the named object, in descending chronological order. The ```starting_after``` parameter returns objects created after the named object, in ascending chronological order. If both parameters are provided, only ending_before is used.

Arguments:

- ```limit``` (optional) - A limit on the number of objects to be returned, between 1 and 100. Default: 50;
- ```starting_after``` (optional) - A cursor for use in pagination. ```starting_after``` is an object ID that defines your place in the list. For instance, if you make a list request and receive 100 objects, ending with ```obj_foo```, your subsequent call can include ```starting_after=obj_foo``` in order to fetch the next page of the list;
- ```ending_before``` (optional) - A cursor for use in pagination. ```ending_before``` is an object ID that defines your place in the list. For instance, if you make a list request and receive 100 objects, starting with ```obj_bar```, your subsequent call can include ```ending_before=obj_bar``` in order to fetch the previous page of the list.

## Content Type

(TODO: Remove XML?)

We support 3 content types that should be send in a ```Content-Type``` header:

- ```application/json``` - Response is sent in a JSON format.
- ```application/xml``` - Response is sent in a  XML format.
- ```text/csv``` - Response is sent in a CSV format.

<aside class="notice">
For CSV we will return only content in a root ```data``` object, without any additional meta (for eg. pagination) to make it prettier in spreadsheets viewers.
</aside>

<aside class="notice">
```Content-Type``` doesn't affect on webhooks format, they are always sent in JSON.
</aside>

To simplify documentation all samples will be provided with JSON content type responses.

## Errors

All errors is returned in JSON format if another ```Content-Type``` is not specified. This means that your will receive JSON for requests with HTTP 415 code when incorrect ```Content-Type``` is provided.

### Error Object

Error description is returned as part of ```meta``` root object. It have following fields:

Parameter | Description
--------- | -----------
type | Error type. You should return human-readable error message based on this field as a key. Type descriptions is listed in next section.
invalid | Collection of validation errors for your request.
invalid[].field_id | ID of invalid field.
invalid[].rule | Failed rule for invalid field. Supported rules: ```required```, ```not_empty```, ```type:integer```, ```type:float```, ```type:string```, ```type:boolean```.
invalid[].params | Optional parameters that can be used in a human-readable error message, to make it easier to understand. Usually it contains limit values for failed validator.
message | Human readable message for API developer.

```json
{
  "meta": {
    "error": {
      "type": "form_validation_failed",
      "invalid": [
        {"field_id": "username", "rule": "required"},
        {"field_id": "email", "rule": "empty"}
        {"field_id": "surname", "rule": "max_length", params:{max_lenght: 5}}
      ],
      "message": "You specified incorrect content type. See https://docs.qbill.ly/#content-type."
    }
  }
}

```

### Application Error Types

Parameter | Description
--------- | -----------
ID | The ID to retrieve

### HTTP status codes

HTTP Code | Description
--------- | -----------
```200``` | Everything worked as expected.
```400``` | Bad Request. The request was unacceptable, often due to missing a required parameter. Or request contains invalid JSON.
```401``` | Unauthorized. No valid API key provided or API key doesn't match project.
```402``` | The parameters were valid but the request failed.
```403``` | Source or destination account is freezed.
```404``` | Not Found. The requested resource doesn't exist.
```415``` | Incorrect Content-Type HTTP header.
```429``` | Too Many Requests. Rate limit is exceeded.
```500```, ```502```, ```503```, ```504``` | Server Errors. Something went wrong on our end. (These are rare.)

## Rate Limits (Throttling)

We throttle our APIs by default to ensure maximum performance for all developers. All projects share a same rate limit, to avoid API-consuming fraud.

Rate limiting of the API is primarily considered on a per-user basis — or more accurately, per access token in your control. Rate limits are determined globally for the entire application.

Right now rate limit is 5000 calls every 15 minutes, but this value may be adjusted at our discretion.

All responses have 3 additional headers:

HTTP Header | Description
--------- | -----------
X-RateLimit-Limit | Current rate limit for your application.
X-RateLimit-Remaining | Remaining rate limit for your application.
X-RateLimit-Reset | The time at which the current rate limit window resets in [UTC epoch seconds](http://en.wikipedia.org/wiki/Unix_time).

<aside class="notice">
When limit is exceeded all requests will return ```HTTP 402``` status code with corresponding error in ```mete`` response object.
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

## Metadata

We support ```metadata``` field for every objects in our API. It allows the API developer to store custom information related to accounts and transactions. This information can be for example:

- Internal order ID.
- Customers name and email address.

Metadata field supports key-value pairs with the following limitations:

- Up to 24 keys.
- Up to 100 characters for the key (alphanumeric characters, hyphens and underscores).
- Up to 500 characters for the value.
- String, integer, decimals and boolean values only. All other types will be converted into strings.

## Key objects

We have list of key objects that is accessible trough our API:

- ```Accounts``` - Represents a customer object with a balance.
- ```Fundings``` - Represents fundings into system. All top-ups is grouped in this list, to have all money income in a single place.
- ```Transfers``` - Represents all charges and transfer operations in system. (Move of money from one account into another.)
- ```Holds``` - Represents all holds on account balances. Hold is a similar of ```Transfers```, with key difference: hold decreases available balance for an ```Account```, but doesn't really decrease balance itself. ```Hold``` can be declined or turned into ```Transfer```.
- ```Webhooks``` - List of all webhooks and their historical data.
- ```Events``` - List of all events that was created in our API.
- ```Requests``` - List of all incoming requests to the API.
- ```Currencies``` - List of custom currencies.
- ```Settings``` - List of settings for API ```Project```.
- ```Projects``` - List of available projects (they have different access scope and API keys).
- ```Backup``` - Data retention endpoint that allows to download all data of your projects.

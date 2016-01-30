# Interacting with API

Our API is organized around [REST](http://en.wikipedia.org/wiki/Representational_State_Transfer). It has predictable, resource-oriented URLs, and uses HTTP response codes to indicate API errors. We use built-in HTTP features, like HTTP authentication and HTTP verbs, which are understood by off-the-shelf HTTP clients. We support cross-origin resource sharing, allowing you to interact securely with our API from a client-side web application (though you should never expose your secret API key in any public website's client-side code).

### HTTP Verbs

As per RESTful design patterns, API implements following HTTP verbs:

- ```HEAD``` - Can be issued against any resource to get just the HTTP header info.
- ```GET``` - Read resources.
- ```POST``` - Create new resources.
- ```PUT``` - Replace resources (basically field values) or collections.
- ```DELETE``` - Remove resources.

### HTTP status codes

HTTP Code | Description
--------- | -----------
```200``` | Everything worked as expected.
```400``` | Bad Request. The request was unacceptable, often due to missing a required parameter. Or request contains invalid JSON. Duplicate idempotency key.
```401``` | Unauthorized. No valid API key provided or API key doesn't match project.
```402``` | The parameters were valid but the request failed.
```403``` | Source or destination account is disabled.
```404``` | Not Found. The requested resource doesn't exist.
```415``` | Incorrect ```Content-Type``` HTTP header.
```429``` | Too Many Requests. Rate limit is exceeded.
```500```, ```502```, ```503```, ```504``` | Server Errors. Something went wrong on our end. (These are rare.)

## Authentication

To use our service you need to authenticate your application. Authentication to the API is performed via [HTTP Basic Auth](http://en.wikipedia.org/wiki/Basic_access_authentication). Provide your API key as the basic auth username value. You do not need to provide a password. You can manage your API keys in the Dashboard.

```curl
curl https://example.com/resource \
   -u WgLodNU5wCdbSw4f:
```

<aside class="notice">
Even trough we support different scopes for tokens, you should simply use token from a Project settings in your Dashboard. (This token have a ```poject``` access scope.) All following mentions of token should read as "project scope token".
</aside>

<aside class="warning">
Your API keys carry all the privileges, so be sure to keep them secret! Do not share your secret API keys in publicly accessible areas such GitHub, client-side code, and so forth. Paranoid people even don't store it in the configuration files, by keeping them in [memcache](https://en.wikipedia.org/wiki/Memcached).
</aside>

Additionally you can create oAuth cross-integration for authenticating your clients and making requests directly to our API. You can find more info at [Integrating oAuth provider](#integrating-oauth-provider) section.

## Response structure

Response can consist of 5 root properties:

- ```meta``` - URL of the requested resource (```url``` ); requested resource type (```type```); current status (```code```); optional error description (```error```); idempotency key (```idempotency_id```); request id (```request_id```).
- ```urgent``` - Notifications and counters.
- ```data``` - Requested resource.
- ```paging``` - Pagination data.
- ```sandbox``` - Optional data provided by ```sandbox``` environment, for development purposes.

```json
{
  "meta": {
    "url": "https://qbill.ly/transactions/",
    "type": "list",
    "code": "200",
    "idempotency_id": "iXXekd88DKqo",
    "request_id": "qudk48fFlaP"
  },
  "urgent": {
    "notifications": ["Read new emails!"],
    "unseen_payments": 10
  },
  "data": {
    <...>
  },
  "paging": {
    "limit": 50,
    "cursors": {
      "after": "MTAxNTExOTQ1MjAwNzI5NDE=",
      "before": "NDMyNzQyODI3OTQw"
    },
    "has_more": true
  },
  "sandbox": {
    "debug_varibale": "39384"
  }
}

```

## Errors

All errors is returned in JSON format if another ```Content-Type``` is not specified. This means that your will receive JSON for requests with HTTP 415 code when incorrect ```Content-Type``` is provided.

### Error Object Properties

You can find all necessary information about occurred error in a response ```meta.error``` property. It can have following fields:

Fields | Description
--------- | -----------
type | General error type. You should return human-readable error message based on this field as a key. Type descriptions is listed in next section.
invalid | Collection of validation errors for your request.
invalid[].entry_type | Type of invalid field.
invalid[].entry_id | ID of invalid field.
invalid[].rule | Failed rule for invalid field. You can find supported validation rules at [Request Validators](#request-validators) table.
invalid[].params | Optional parameters that can be used in a human-readable error message, to make it easier to understand. Usually it contains limit values for failed validator.
message | Human readable message for API developer.

```json
{
  "meta": {
    "error": {
      "type": "form_validation_failed",
      "invalid": [
        {"entry_type": "field", "entry_id": "username", "rule": "min:6", "params":{"lenght": 4}},
        {"entry_type": "field", "entry_id": "email", "rule": "empty"},
        {"entry_type": "header", "entry_id":"Timezone", "rule": "timezone"},
        {"entry_type": "request", "entry_id":null, "rule": "json"}
      ],
      "message": "Validation failed. Return human-readable error message. You find all possible validation rules at https://docs.qbill.ly/#request-validators."
    }
  }
}

```

### Error Types

Parameter | Description
--------- | -----------
ID | The ID to retrieve
duplicated_idempotency_key |

### Request Data Validators

All invalid request data is listed in ```meta.error.invalid``` object. There are different types of entries:

- ```header``` - Response HTTP header.
- ```request``` - Response JSON object.
- ```field``` - Response field.

List of possible validation rules:

Validator | Description
------------------------- | -----------
```active_url``` | The field under validation must be a valid URL according to the checkdnsrr PHP function.
```after:<date>``` | The field under validation must be a value after a given date. The dates will be passed into the strtotime PHP function. Sample rule: ```date|after:tomorrow```.
```before:<date>``` | The field under validation must be a value preceding the given date. The dates will be passed into the PHP strtotime function. Sample rule: ```date|before:today```.
```alpha``` | The field under validation must be entirely alphabetic characters.
```alpha_dash``` | The field under validation may have alpha-numeric characters, as well as dashes and underscores.
```alpha_num``` | The field under validation must be entirely alpha-numeric characters.
```between:<min>,<max>``` | The field under validation must have a size between the given <min> and <max>. Strings, numerics, and files are evaluated in the same fashion as the size rule.
```boolean``` | The field under validation must be able to be cast as a boolean. Accepted input are ```true```, ```false```, ```1```, ```0```, ```"1"```, and ```"0"```.
```confirmed``` | The field under validation must have a matching field of ```foo_confirmation```. For example, if the field under validation is ```password```, a matching ```password_confirmation``` field must be present in the input.
```date``` | A valid data in ISO 8601 format.
```digits:<value>``` | The field under validation must be numeric and must have an exact length of value.
```digits_between:<min>,<max>``` | The field under validation must have a length between the given min and max.
```email``` | The field under validation must be formatted as an e-mail address.
```in:<foo>,<bar>,<...>``` | The field under validation must be included in the given list of values.
```json``` | The field under validation must be a valid JSON string.
```max:<value>``` | The field under validation must be less than or equal to a maximum value. Strings, numerics, and files are evaluated in the same fashion as the size rule.
```min:<value>``` | The field under validation must have a minimum value. Strings, numerics, and files are evaluated in the same fashion as the size rule.
```not_in:<foo>,<bar>,<...>``` | The field under validation must not be included in the given list of values.
```numeric``` | The field under validation must be numeric.
```regex:<pattern>``` | The field under validation must match the given regular expression.
```required``` | The field under validation must be present in the input data and not empty. A field is considered "empty" is one of the following conditions are true: the value is null; the value is an empty string; the value is an empty array or empty Countable object; the value is an uploaded file with no path.
```size:<value>``` | The field under validation must have a size matching the given value. For string data, value corresponds to the number of characters. For numeric data, value corresponds to a given integer value. For files, size corresponds to the file size in kilobytes.
```string``` | The field under validation must be a string.
```timezone``` | The field under validation must be a valid timezone identifier according to the timezone_identifiers_list PHP function.
```url``` | The field under validation must be a valid URL according to PHP's ```filter_var``` function.
```otp_code``` | A valid OTP code.
```password_strong``` | Password field that should include at least one latin letter in lowercase, one letter in uppercase and one number. Min lenght is set by another rule.
----------------------------- |

## Pagination

All top-level API resources with root type ```list``` have support of pagination over a "list" API methods. These methods share a common structure, taking at least these three parameters: ```limit```, ```starting_after```, and ```ending_before```.

API utilizes cursor-based pagination via the ```starting_after``` and ```ending_before``` parameters. Both take an existing object ID value (see below). The ```ending_before``` parameter returns objects created before the named object, in descending chronological order. The ```starting_after``` parameter returns objects created after the named object, in ascending chronological order. If both parameters are provided, only ending_before is used.

Arguments:

- ```limit``` (optional) - A limit on the number of objects to be returned, between 1 and 100. Default: 50;
- ```starting_after``` (optional) - A cursor for use in pagination. ```starting_after``` is an object ID that defines your place in the list. For instance, if you make a list request and receive 100 objects, ending with ```obj_foo```, your subsequent call can include ```starting_after=obj_foo``` in order to fetch the next page of the list;
- ```ending_before``` (optional) - A cursor for use in pagination. ```ending_before``` is an object ID that defines your place in the list. For instance, if you make a list request and receive 100 objects, starting with ```obj_bar```, your subsequent call can include ```ending_before=obj_bar``` in order to fetch the previous page of the list.

## Content Type

You should send ```Content-Type``` header in all your requests, otherwise API will return HTTP 415 status. Right now we support two content types:

- ```application/json``` - Response is sent in a JSON format.
- ```text/csv``` - Response is trimmed to a requested resource and sent in a CSV format.

This header doesn't affect any outgoing requests, they are always sent in JSON format.

<aside class="notice">
To make CSV look better in spreadsheets viewers we are stripping all response metadata (everything except data that is returned in ```data``` field).
</aside>

## Rate Limits (Throttling)

We throttle our APIs by default to ensure maximum performance for all developers.

Rate limiting of the API is primarily considered on a per-consumer basis. All your projects share a same rate limit, to avoid API-consuming fraud. Rate limits depend on your account type.

Currently free accounts is rate limited to 1000 API calls every 15 minutes, but this value may be adjusted at our discretion.

For your convenience, all requests is sent with 3 additional headers:

HTTP Header | Description
------------- | -----------
X-RateLimit-Limit | Current rate limit for your application.
X-RateLimit-Remaining | Remaining rate limit for your application.
X-RateLimit-Reset | The time at which the current rate limit window resets in [UTC epoch seconds](http://en.wikipedia.org/wiki/Unix_time).

<aside class="notice">
When limit is exceeded all requests will return ```HTTP 429``` status code with corresponding error in ```meta`` response object.
</aside>

```
X-RateLimit-Limit: 5000
X-RateLimit-Remaining: 4966
X-RateLimit-Reset: 1372700873
```

## Cross Origin Resource Sharing

The API supports Cross Origin Resource Sharing (CORS) for AJAX requests from any origin. You can read the [CORS W3C Recommendation](http://www.w3.org/TR/cors/), or [this intro](http://code.google.com/p/html5security/wiki/CrossOriginRequestSecurity) from the HTML 5 Security Guide.

<aside type="notice">
Never publish your application or project tokens in any kind of Front-End applications. They carry all the project privileges, and would be exposed to a third-parties. You can find more info about client authentication in [Adding oAuth provider](#adding-oauth-provider) section.
</aside>

```
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: Authorization, Content-Type, If-Match, If-Modified-Since, If-None-Match, If-Unmodified-Since, X-Requested-With
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Expose-Headers: ETag, Link, X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset, X-Request-ID, X-Idempotency-Key
Access-Control-Allow-Credentials: true
```

## Timezones

All requests allow to provide a ```Time-Zone``` header to receive all date and time fields in a local timezone.

Explicitly provide an ISO 8601 timestamp with timezone information to use this feature. Also it is possible to supply a Time-Zone header which defines a timezone according to the list of names from the [Olson database](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).

```shell
curl -H "Time-Zone: Europe/Amsterdam" ..
curl -H "Time-Zone: Europe/Kiev" ..
```

## Request ID's

Each API request has an associated request identifier. You can find it in ```X-Request-ID``` response header or inside ```meta.request_id``` property of returned JSON.

You can also find request identifiers in the URLs of individual request logs in your Dashboard. If you need to contact us about a specific request, providing the request identifier will ensure the fastest possible resolution.

## Limiting Response Fields

By default, all the fields in a node are returned when you make a query. You can choose the fields you want returned with the ```fields``` query parameter. This is really useful for making your API calls more efficient and fast.

```curl
/resource?fields=id,name,balance
```

## Expanding Response Fields into Objects

Many response fields contain the ID of a related object in their values. For example a ```Transfer``` can have an associated ```User``` object linked by a ```sender``` field. Those objects can be expanded inline with the ```expand``` request parameter. Objects that can be expanded are noted in this documentation. This parameter is available on all API requests, and applies to the response of that request only.

You can expand object by querying list, expands will be applied on all matched elements. Expanded lists return up to 25 elements, limit can be specified in brackets.

You can expand multiple objects at once by identifying multiple items divided by comma.

```curl
/accounts?expand=transactions(5)
```

You can nest expand requests with the dot property. For example, requesting ```sender.payments``` on a ```Transfer``` list will expand the ```sender``` property into a full ```User``` object, and will then expand the ```payments``` property of that ```User``` into a full ```Transactions``` collection.

```curl
/accounts?expand=transactions(5).recipients
```

## Ordering Lists and Collections

By default, all collections are ordered in ascending chronological order. You can specify different order by providing the "order" query parameter.

```curl
/accounts?order=account.created_at(reverse_chronological)
```

Available orders:

- ```ascending_chronological``` - Ascending chronological order.
- ```reverse_chronological``` - Descending chronological order.

## Filtering Lists

You can filter all lists by a data. Filter can refer to any fields inside a ```data``` response property, except entities that was expanded. Also you can filter by a metadata value.

You should set filter as a [base64 encoded](https://en.wikipedia.org/wiki/Base64) JSON object string in a ```filter``` query parameter.

> Create a JSON object and convert it to string:

```bash
echo "{predicates:[<predicate_1>,<predicate_1>,...]}" | base64
```

> Result: e3ByZWRpY2F0ZXM6WzxwcmVkaWNhdGUxPiw8cHJlZGljYXRlMT4sLi4uXX0K

> Send encoded string as filter query parameter:

```
GET /v1/accounts?filter=e3ByZWRpY2F0ZXM6WzxwcmVkaWNhdGUxPiw8cHJlZGljYXRlMT4sLi4uXX0K
```

Predicate is an object with 4 fields:

- ```field``` - Resource filed that should be used for current filter predicate.
- ```comparison``` - Comparison type that is applied to a field value and ```value``` predicate property.
- ```value``` - Value that should be compared with resource field value.

Available comparison methods:

Name | Description
------------- | -----------
eq | Matches values that are equal to a specified value.
ne | Matches all values that are not equal to a specified value.
gt | Matches values that are greater than a specified value.
gte |  Matches values that are greater than or equal to a specified value.
lt | Matches values that are less than a specified value.
lte |  Matches values that are less than or equal to a specified value.
in | Matches any of the values specified in an array. Array is set to a ```value``` predicate property.
nin |  Matches none of the values specified in an array.
ewi | Matches all values that ends with a specified value.
swi | Matches all values that starts with a specified value.

```json
{
  predicates:[
    {
      "attribute":"balance",
      "comparison":"eq",
      "value":"27"
    },
    {
      "attribute":"metadata.user_id",
      "comparison":"in",
      "value":[1, 2, 3]
    }
  ]
}
```

To apply filters with logical rules you can add with logical type. Available types:

Name | Description
------------- | -----------
or | Joins query clauses with a logical OR returns all objects that match the conditions of either clause.
and | Joins query clauses with a logical AND returns all objects that match the conditions of both clauses.
not | Inverts the effect of a query expression and returns objects that do not match the query expression.
nor | Joins query clauses with a logical NOR returns all objects that fail to match both clauses.

```json
{
  "predicates":[
    {
      "attribute":"metadata.user_id",
      "comparison":"eq",
      "value":"3994kdkd8"
    },
    {
      "type":"or",
      "predicates":[
        {
          "attribute":"balance",
          "comparison":"gt",
          "value":"100"
        },
        {
          "attribute":"transactions.count",
          "comparison":"gt",
          "value":"10"
        }
      ]
    }
  ]
}
```

## Aggregating lists

All lists can be queried to get aggregations on given set of rules. When you are using aggregation default filtered period is one month. For example you can get aggregated count of ```Accounts``` to know how many accounts was created at each day for last month.

```
GET /accounts?aggregate_stategy=count&aggregate_field=count&tick=day
```

To use aggregation you need to specify at least 3 fields:

- ```aggregate_stategy``` - Aggregation strategy.
- ```aggregate_field``` - Field that will be used for aggregation.
- ```tick``` - Aggregation period particle. Default value: ```day```.

Available aggregation strategies:

Name | Description
------------- | -----------
count | Returns count of returned aggregated fields.
add | For integers and floats returns total for all field values. For strings returns concatenated string of all field values.
max | Returns maximum value of a field.
min | Returns minimum value of a field.
avg | Returns average value of a field.

Available ```tick``` values:

Name | Description
------------- | -----------
hour | Return aggregation for each hour in a filtered period.
day | Return aggregation for each day in a filtered period. Default value.
month | Return aggregation for each month in a filtered period.
year | Return aggregation for each year in a filtered period.

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
- String, integer, decimals and boolean values only. All other types will be converted into string.

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

## Batching Request

There are number of situations when you need to download few entities at once, for example when your customers have multiple accounts and you need to return balance for all of them in a single request. For this cases we have request multiplexing.

The batch API takes in an array of logical HTTP requests represented as JSON arrays - each request has a method (corresponding to HTTP method GET/PUT/POST/DELETE etc.), a uri (the portion of the URL after domain), optional headers array (corresponding to HTTP headers) and an optional body (for POST and PUT requests). The Batch API returns an array of logical HTTP responses represented as JSON arrays - each response has a status code, an optional headers array and an optional body (which is a JSON encoded string).

To make batched requests, you build a JSON object which describes each individual operation you'd like to perform and POST this to the batch API endpoint at https://batcher.qbill.co.

```
curl \
    -F ‘batch=[{“method”:”GET",“relative_url”:/accounts/:id1”},{“method”:”GET",“relative_url”:/accounts/:id1”}]’ \
    https://graph.facebook.com
```

As an alternative you can use simpler alias for a batching requests to the objects. Whenever you can provide an ID inside URL to retrieve resource, you can also provide multiple ID's separated by comma. For example, you can get two user accounts in one request.

```
GET /projects/:project_id/accounts/:id1,id2,...
```

> Response would be a list of requested items.

```
{
  data: {
    "<user1_id>": {"<user1>"},
    "<user2_id>": {"<user2>"}
  }
}

```

### Timeouts

Large or complex batches may timeout if it takes too long to complete all the requests within the batch. In such a circumstance, the result is a partially-completed batch. In partially-completed batches, responses from operations that complete successfully will look normal whereas responses for operations that are not completed will have corresponding error code in ```meta``` property.

### Limits

You can batch up to 10 request. Every request in batch will be counted as separate requests in Rate Limits.

## Request Flow

1. Save request data.
2. Run any pre-flight webhooks, for example for customer authentication. And if webhooks returned 2XX codes..
3. Check authentication. And if authorized..
3. Check rate limits for your application. And if they not exceeded..
4. Validate request data and complete request.
4.1. Generate event for this request.
4.2. Queue money flow (all operations with money is asynchronous).
5. Return API response
6. Save API response to ```Requests```
7. Schedule all webhooks configured for a ```Project```. If webhook failed, re-try it.
8. Record all webhooks statuses

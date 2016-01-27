# Best Practices

## Currency flows

We encourage you to use right accounting models inside your system. Separate all accounts by a ```type```, lets call them ```system``` and ```client``` accounts.

This will help you to calculate losses and revenue in a right, predictable way.

### Funding account

Every time you need to add a money to your system you should create a ```Funding```. Even trough they have a separate endpoint we strongly recommend to fund a technical accounts, rather that directly funding consumer accounts.

For example, you can create a system account for money inflow for each of your Payment Service Providers, and you would be able to list all

Also this allows you to charge front-fees on your money income.

## Token and ID lengths and formats

In order to avoid interruptions in processing, it's best to make minimal assumptions about what our gateway-generated tokens and identifiers will look like in the future. The length and format of these identifiers – including payment method tokens and transaction IDs – can change at any time, with or without advance notice. However, it is safe to assume that they will remain 1 to 64 upper- and lowercase alphanumeric characters with minuses (```-```) and underscores (```_```).

## Geographic Redundancy and Optimization

To ensure that you will always have the lowest response time we can provide, we are automatically detecting nearest datacenter to you, so all your projects have master servers in it. To migrate data to different region please contact our support team.

## Idempotent Requests

The API supports idempotency for safely retrying write requests without accidentally performing the same operation twice. For example, if a request to create a charge fails due to a network connection error, you can retry the request with the same idempotency key to guarantee that only a single charge is created.

To perform an idempotent request, attach a unique key to any ```POST```, ```DELETE``` or ```PUT``` request made to the API via the ```Idempotency-Key: <key>``` header.

How you create unique keys is completely up to you. We suggest using random strings or UUIDs. We'll always send back the same response for requests made with the same key. However, you cannot use the same key with different request parameters. The keys expire after 24 hours.

## Conditional requests

Most responses return an ETag header. Many responses also return a Last-Modified header. You can use the values of these headers to make subsequent requests to those resources using the ```If-None-Match``` and ```If-Modified-Since``` headers, respectively. If the resource has not changed, the server will return a 304 Not Modified.

Also note: making a conditional request and receiving a 304 response does not count against your Rate Limit, so we encourage you to use it whenever possible.

## Versioning

All API calls should be made with a X-API-Version header which guarantees that your call is using the correct API version. Version is passed in as a date (UTC) of the implementation in YYYY-MM-DD format.

If no version is passed, the newest will be used and a warning will be shown. Under no circumstances should you always pass in the current date as that will return the current version which might break your implementation.

## SSL certificates

We recommend that all users obtain an SSL certificate and serve any data that is stored in our service over HTTPS.

We don't have a specific preferred vendor for SSL certificates, but we recommend that you stick with a well-known provider (e.g. Network Solutions, GoDaddy, Namecheap). Generally speaking, most certificates will be similar, so it's up to you to determine what fits your needs best.

## HTTP Redirects

API uses HTTP redirection where appropriate. Clients should assume that any request may result in a redirection. Receiving an HTTP redirection is not an error and clients should follow that redirect. Redirect responses will have a Location header field which contains the URI of the resource to which the client should repeat the requests.

Status codes:

- ```301``` - Permanent redirection. The URI you used to make the request has been superseded by the one specified in the Location header field. This and all future requests to this resource should be directed to the new URI.
- ```302, 307``` - Temporary redirection. The request should be repeated verbatim to the URI specified in the Location header field but clients should continue to use the original URI for future requests.

Other redirection status codes may be used in accordance with the HTTP 1.1 spec.

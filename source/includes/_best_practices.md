# Best Practices

## Currency flows

We encourage you to use right accounting models inside your system. Separate all accounts by a ```type```, lets call them ```system``` and ```client``` accounts.

This will help you to calculate losses and revenue in a right, predictable way.

### Funding account

Every time you need to add a money to your system you should create a ```Funding```. Even trough they have a separate endpoint to load all fundings, we strongly recommend to fund a transit accounts first, rather that directly consumer accounts.

For example, you can create a system account for money inflow for each of your Payment Service Providers, and you would be able to list all transfers trough it, compare money flow of our PSP and QBill, and to charge front-fee by leaving some money on transit account or by batch-transferring it both to customer and "revenue" accounts.

Also we strongly recommend you to use ```Funding``` metadata power, by adding to it original ID of your PSP, original amount and other helpful information, to efficiently track money flow trough all the systems.

Additionally it would be easier for you to show your customer all transactions log, because all operations will be available trough ```Transfers``` list. Otherwise, your will need to merge two lists: ```Transfers``` and ```Fundings```.

### Revenue tracking

For revenue we recommend you to create special account for each of revenue-takers (you, your partners, etc.)

This will allow to find all transactions that is gained your revenue, and to easily understand how many you need to pay to your partners.

Finally if you want to clear revenue account once a while (for example, each time you sending money to a partner), you can simply send them to "/dev/null" account, that will accommodate all funds that should be terminated.

Direct reduction of account is not possible to keep you from common accounting mistakes.

### Charges

Any time you charge customer for any type your services, you should simply create a transfer to a revenue account with corresponding metadata (to show correct transaction information for your customers).

### Account Overdrafts

(TODO.)

## Integrating oAuth provider

Sometimes is more rational to make request to an API without any additional services that will proxy this requests. But by adding project token to your application you would make it very vulnerable to a third-party's.

For this cases we recommend you to request our API with a oAuth token and to add a pre-flght webhook to connect API to your oAuth provider. Flow should look something like this:

1. Your application requests API with ```Authorization: Bearer <token>``` header.
2. Pre-flight webhook is send to your oAuth endpoint.
3. oAuth endpoint validates Bearer token and if its valid returns ```HTTP 200``` status code with a ```X-Override-Authorization: Token <project_token>``` header, where ```<project_token>``` is a API project token issued in Dashboard.
4. Our API validates new authorization token and fulfills the request.

Sample oAuth provider can be found in our [GitHub]() account.

## Token, ID lengths and formats

In order to avoid interruptions in processing, it's best to make minimal assumptions about what our gateway-generated tokens and identifiers will look like in the future. The length and format of these identifiers – including payment method tokens and transaction IDs – can change at any time, with or without advance notice. However, it is safe to assume that they will remain 1 to 64 upper- and lowercase alphanumeric characters with minuses (```-```) and underscores (```_```).

We use ISO 8601 formats for all dates in our API.

We use E.123 telephone number in international notation for all phone numbers in our API.

All tokens have a human-readable prefix, so you can always see what scope it carries. Example: ```project-lksdlkfjf8ds8dfsl````.

All ID's have a human-readable prefix that carries first 3 characters from a entity name. Example: ```acc_ssj8988udj```.

## Geographic Redundancy and Optimization

To ensure that you will always have the lowest response time we can provide, we are automatically detecting nearest datacenter to you, so all your projects have master servers in it. To migrate data to a different region please contact our support team.

## Data Storage and Backup Policy

To ensure that you won't loose your data we use geographical redundant MongoDB replica sets. It means that at least one of your secondary DB's is hosted in another region, and will save all data in case main datacenter would be unavailable.

## Providing urgent data for your users

Sometimes you want to update account balance and notifications list on each request you made. You can provide additional HTTP header ```X-Urgent-Account-ID``` with an Account ID that should be queried. All result data will be in ```urgent``` response field.

```
X-Urgent-Data-ID: acc_3idjdjkd9
```

```
{
  "meta": {
  },
  "urgent": {
    "account": "acc_3idjdjkd9",
    "notifications": [],
    "unseen_payments": 0,
    "holds": 0,
    "balance": 0
  },
  "data": {
  }
}
```

## Idempotent Requests

The API supports idempotency for safely retrying write requests without accidentally performing the same operation twice. For example, if a request to create a charge fails due to a network connection error, you can retry the request with the same idempotency key to guarantee that only a single charge is created.

To perform an idempotent request, attach a unique key to any ```POST```, ```DELETE``` or ```PUT``` request made to the API via the ```Idempotency-Key: <key>``` header.

How you create unique keys is completely up to you. We suggest using random strings or UUIDs. We'll always send back the same response for requests made with the same key. However, you cannot use the same key with different request parameters (We will return ```HTTP 400``` error in this case). The keys expire after 24 hours.

## Conditional requests

Most responses return an ETag header. Many responses also return a Last-Modified header. You can use the values of these headers to make subsequent requests to those resources using the ```If-None-Match``` and ```If-Modified-Since``` headers, respectively. If the resource has not changed, the server will return a 304 Not Modified.

Also note: making a conditional request and receiving a 304 response does not count against your Rate Limit, so we encourage you to use it whenever possible.

## Versioning

All API calls should be made with a ```X-API-Version``` header which guarantees that your call is using the correct API version. Version is passed in as a date (UTC) of the implementation in YYYY-MM-DD format.

If no version is passed, the newest will be used and a warning will be shown. Under no circumstances should you always pass in the current date as that will return the current version which might break your implementation.

## SSL certificates

We recommend that all users obtain an SSL certificate and serve any data that is stored in our service over HTTPS. Also we don't allow to add a webhook that doesn't support SSL encryption.

We don't have a specific preferred vendor for SSL certificates, but we recommend that you stick with a well-known provider (e.g. Network Solutions, GoDaddy, Namecheap). Generally speaking, most certificates will be similar, so it's up to you to determine what fits your needs best.

You can test your server with a [SSL Server Test](https://www.ssllabs.com/ssltest/).

## HTTP Redirects

API uses HTTP redirection where appropriate. Clients should assume that any request may result in a redirection. Receiving an HTTP redirection is not an error and clients should follow that redirect. Redirect responses will have a Location header field which contains the URI of the resource to which the client should repeat the requests.

Status codes:

- ```301``` - Permanent redirection. The URI you used to make the request has been superseded by the one specified in the Location header field. This and all future requests to this resource should be directed to the new URI.
- ```302, 307``` - Temporary redirection. The request should be repeated verbatim to the URI specified in the Location header field but clients should continue to use the original URI for future requests.

Other redirection status codes may be used in accordance with the HTTP 1.1 spec.

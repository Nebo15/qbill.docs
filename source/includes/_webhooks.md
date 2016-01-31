# Webhooks

All webhooks is sent as ```POST``` request to a selected endpoint.

Right now we support two types of webhooks: pre-flight and event-based. Pre-flight webhooks is send on all requests and should respond synchronously. All event-based webhooks is sent when request is completed.

All actions will trigger creation of an ```Event``, that is be send to all your webhooks.

### Limits

Webhooks is also exceeding your rate limits, one for each request that was sent, so be careful and void creating too many of them.

Right now you can create two pre-flight webhooks and 5 event-based webhooks.

## Request structure

We are sending same data object as described in an ```Event``` section of this documentation.

```
<Request sample>
```

(TODO: Move this to an event structure.)
We add few additional HTTP headers for a request:

```
X-Application-ID: <application_id>
X-Project-ID: <project_id>
X-HTTP-Verb: <http_verb>
X-Requested-Object: <api_entity>
X-Requested-URI: <uri>
```

You can make sure that webhooks is coming from our back-end by validating ```X-Webhook-Secret``` header. It's a sha1 hash of ```request_id``` and ```project_api_token```.
```
addHeader('X-Webhook-Secret', sha1($request_id . $project_api_token));
```

## Event-Based Webhooks

If event-based webhook is failed to deliver we will retry it for the next 24 hours. Retry time is increased each time request is made: 5, 15, 30 and 60 minutes. (60 minutes is a maximum timeout.)

## Pre-Flight Webhooks

To make API easier to integrate with other systems we can synchronously pass all requested data and all available metadata to endpoint specified in a Dashboard.

Pre-Flight requests doesn't retry over time, of configured endpoint is not responding for 15 seconds we will return ```HTTP 412 Precondition Failed``` header.

Request webhooks is sent in same order they was created. You can re-order them in your Dasbhoard.

Service will act differently based on HTTP response of this endpoint:

oAuth HTTP Code | Action
--------- | -----------
2XX | Continue processing your request.
301, 307, 308 | Follow the redirect. And continue checking response code.
401, 402, 4XX | Return same HTTP code
5XX, timeout and all other codes | Return HTTP 412 Precondition Failed

This webhooks is extremely useful for antifraud purposes, when another system needs to validate transaction before they are completed.

### Modifying original request headers

Pre-Flight Webhooks can modify original request, for example to exchange oAuth user token to an Project token. This allow you to use our API directly from client devices and don't expose secret API tokens.

To modify request simply return header in following format: ```X-Override-<Header>```.

```
X-Override-Authorization: Token 3kdjd9d0uds080ujdsj38
```

Also you can add and save your internal request ID to a webhooks log by using same technique.

```
X-Override-External-Request-ID: 8383729938
```

You can modify following HTTP headers:

- Authorization
- Timezone
- X-Urgent-Account-ID

## List all Webhooks

## Add a Webhook

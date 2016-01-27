# Webhooks

Each webhook call is added to your rate limit, so be careful and void creating too many of them.

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

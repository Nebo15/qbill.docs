---
title: Project Palantir

language_tabs:
  - shell

toc_footers:
 - <a href='index.html'>Billing API</a>
 - <a href='index.html'>Applications API</a>

search: true
---

# Introduction

Palantir is our API Service Bus that helps to connect different applications with different API and with different data sources.

# Use Cases

## Enriching data

Sometimes you are receiving user data, but this data may be not enough for decision making or other business logic. In this case you can set-up a data model and to tell Palantir where should it get rest of data and what parameters this service consume.

For example, you have a lending site and you need to score your users before sending applicant data to decision-making system. You need to:

1. Setup Palantir data model called "Applicant".
2. Setup ```ON_POST``` trigger for "Applicant". (Also you need to setup a ```ON_CALLBACK_FAILIED``` trigger.)
3. Configure this triggers authorization type (lets say, ```HTTP Basic```), login and password, and API endpoint.
4. Describe what data this API consumes:
```
{
  applicant_id: "{{applicant.id}}",
  ssh: "{{applicant.ssn}}",
  idn: "{{applicant.idn}}",
  callback_url: "{{palantir.callback_url}}"
}
```
5. Describe how to map response
```
applicant.score = response.scoring.result;
```
6. Setup next webhook. ```ON_MODEL_FILLED```. And API endpoint where Palantir need to post data.

> Create applicant

```
POST /applicants
{
  name: "John Smith",
  ssh: "8383-29383-22",
  idn: "AA293823",
  birth_date: "1990-11-19"
}
```

> Palantir requests scroring API

```
POST https://example.com/score
{
  applicant_id: "123",
  ssh: "8383-29383-22",
  idn: "AA293823",
  callback_url: "https://palantir-example.com/callbacks/diuqng3bci382llkd"
}
```

> Receives async scoring result. It can make sync calls either.

> Lets check how applicant now looks like

```
GET /applicants/123
{
  applicant_id: "123",
  ssh: "8383-29383-22",
  idn: "AA293823",
  score: 87
}
```

## Data Multiplexing

There are cases when you need to send multiple webhooks when you receive some data. For example, on each user creation you want to send API request to Mixpanel and Intercom with corresponding events.

## API ACL

You can setup trigger on each request. (Webhooks from QBill.)

## Sync to Async

If you want to wait, its your decision :). You can create a sync APi endpoint that will make one or few webhook calls and return response when they will be accomplished.

# DB Structure

```
models: [
  {
    _id: "111",
    applicant_id: "123",
    ssh: "8383-29383-22",
    idn: "AA293823",
    score: 87
  }
]

triggers: [
  {
    model: 'applicants',
    triggers: [
      {
        _id: "010",
        on: "post",
        type: "async",
        endpoint: {...}
      },
      {
        _id: "010",
        on: "filled",
        type: "sync",
        endpoint: {...}
      }
    ]
  }
]
```

---
title: QBill API Reference

language_tabs:
  - shell

includes:
  - introduction
  - interacting
  - best_practices

search: true
---


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

Accounts support ```metadata``` object. See more info at [Metadata](#metadata) section.

```
POST /v1/accounts
{
  metadata: {
    external_id: 192838
  }
}
```

> Response

```
{
  "meta": {
    "code": "201",
    "url": "https://api.qbill.ly/accounts/acc_388djejje88du",
    "type": "account",
    "request_id": "qudk48fFdaP",
    "idempotency_id": "iXXedd88DKqo"
  },
  "urgent": {
    "notifications": [],
    "unseen_payments": 0,
    "holds": 0,
    "balance": 0
  }
  "data": {
    id: "acc_388djejje88du"
    balance: 0,
    metadata: {
      external_id: 192838
    }
  },
}
```

## Get all Account data

```
GET /v1/accounts/:id
```

## Disabling and Enabling an Account
Accounts can't be deleted, but can be disabled to prevent its future usage. Disabled account will always return HTTP 403 error.

### Disabling

```
POST /v1/accounts/:id/freeze
```

### Enabling

You can enable account to continue using it later.

```
POST /v1/accounts/:id/enable
```

# Charges

Charge is a main operation with account balance that covers both payments and transfers.

Charges can be ```internal``` and ```external```. We recommend you to skip ```internal``` Charges for all lists that will be visible for your customers, since then carry only technical purposes.

## Creating Charge

There are two flows for creating a Charge:
- Instant - create hold and immediately convert it to charge;
- Delayed - create hold and manually convert it to charge.

A single payment can have multiple charges that will look like a single Charge for an account, but it will create multiple technical Charges. This is useful to charge fees.

### Instant (One-Step) Charge

```
POST /v1/accounts/:account_id/Charges
{
  total: 100,
  charges: [
    {subtotal: 90, destination: "<service_account_id>", meta: {service_id: 1, service_name: 'Cellular Topup'}}
    {subtotal: 10, destination: "<fees_account_id>", meta: {for: "service_payment", service_id 1}}
  ],
  meta: {}
}
```

### Delayed (Multi-Step) Charge

(TODO: Transfer with a currency conversion.)

Hold money on target account.

```
POST /v1/accounts/:account_id/holds
```

> While money is on-hold you can change any payment details, for eg. to refund some part of money

```
PUT /v1/holds/:Charge_id
{
  total: 20.00
}
```

> After hold payment can be completed to commit balance change and turn hold into charge or declined to remove hold and return funds to available balance.

```
POST /v1/Charges/:Charge_id/complete
```

```
POST /v1/Charges/:Charge_id/decline
```

## List all Charges

List all Charges with a filter. Filter can represent any field that is available in the response, except entities that needs to be expanded.

(TODO: Add less than, greater than, text search).

```
GET /v1/Charges?filters=[]
```

> For analytical purposes response can be grouped by a field

```
GET /v1/Charges?filters=[]&group_by=[]
```

> For analytical purposes response can return aggregation counts

```
GET /v1/Charges?filters=[]&get_count_by=[]
```

## Rollback a Charge

To a rollback we will create a new Charge with a ```type=internal``` and a ```refference_id=<originalChargeID>``` fields.

## Create Refund for a Charge

Refund is similar to a Rollback, but you need to specify refund total for every account that received funds. This allows you to refund funds by keeping the fees or to refund it with all the fees.

## Transferring money between projects

(TODO: Should we handle it or leave it for developers?)

# Holds

## Funding an Account

(TODO: Move this to a Fundings entity?)

(TODO: Allow subpayments on funding to charge front fee?)

We recommend to fund only system accounts and to avoid direct funding for user-related accounts. This simplifies accounting of your system's money flow. You can find more info about best accounting practices at [Currency Flows](#currency-flows) section.

```
POST /v1/accounts/:account_id/fund
{
  total: 1000
}
```

## Create a Hold

## List all Holds

## Get all Account Holds

## Cancel a Hold

## Convert Hold into Charge

# Fundings

Funding allows to top-up any balance in a system. Basically this is an equivalent for an money emission, where all generated funds will be sent to specific system account.

## Create a Funding operation

## List all Funding operations

## Canceling a Funding operation

All Fundings can't be canceled, to do so just create a Charge and move money to system account.

# Currencies and Conversion Rates

Our system supports any currency with a custom conversion rates. To simplify conversion we have a base

## Creating a Currency

## List all Currencies

## Updating a Currency

### Setting a base Currency

### Changing Conversion Rates

# Events

## List all Events

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

## List all Requests

# SQL Queries

# Backups

## Download all data


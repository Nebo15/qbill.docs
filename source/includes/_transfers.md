# Transfer

Charge is a main operation with account balance that covers both payments and transfers.

Transfer can be ```internal``` and ```external```. We recommend you to skip ```internal``` Transfer for all lists that will be visible for your customers, since then carry only technical purposes.

## Creating Charge

There are two flows for creating a Charge:
- Instant - create hold and immediately convert it to charge;
- Delayed - create hold and manually convert it to charge.

A single payment can have multiple Transfer that will look like a single Charge for an account, but it will create multiple technical Transfer. This is useful to charge fees.

### Instant (One-Step) Charge

```
POST /v1/accounts/:account_id/Transfer
{
  total: 100,
  Transfer: [
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
POST /v1/transfers/:Charge_id/complete
```

```
POST /v1/transfers/:Charge_id/decline
```

## List all Transfer

List all Transfer with a filter. Filter can represent any field that is available in the response, except entities that needs to be expanded.

(TODO: Add less than, greater than, text search).

```
GET /v1/transfers?filters=[]
```

> For analytical purposes response can be grouped by a field

```
GET /v1/transfers?filters=[]&group_by=[]
```

> For analytical purposes response can return aggregation counts

```
GET /v1/transfers?filters=[]&get_count_by=[]
```

## Rollback a Charge

To a rollback we will create a new Charge with a ```type=internal``` and a ```refference_id=<originalChargeID>``` fields.

## Create Refund for a Charge

Refund is similar to a Rollback, but you need to specify refund total for every account that received funds. This allows you to refund funds by keeping the fees or to refund it with all the fees.

## Transferring money between projects

(TODO: Should we handle it or leave it for developers?)

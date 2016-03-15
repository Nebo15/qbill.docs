# Holds

(TODO: Allow subpayments on funding to charge front fee?)

(TODO: Holds should be part of Transfers API response, so consumers can return correct payment history to clients.)

(TODO: Transfer with a currency conversion.)

You should hold some amount from account balance whenever you create a multi-step payments.

## List all Holds

```
GET /projects/:project_id/holds
```

## Create a Hold

To create a hold you should provide at least two fields:

- ```transfer``` - List of Transfers that should be created when Hold is completed into a Transfer. This allows you to take fees from your customers.
- ```total``` - Total amount should be holded from an Account. Total should be exactly same as sum of all created transfers. (Otherwise we will return an appropriate error.)

You can attach any ```metadata``` to a Hold.

```
POST /projects/:project_id/holds
{
  transfer: [
    {subtotal: 90, destination: "<service_account_id>", metadata: {service_id: 1, service_name: 'Cellular Topup'}}
    {subtotal: 10, destination: "<fees_account_id>", metadata: {for: "service_payment", service_id 1}}
  ],
  total: 100,
  metadata: {
    desctiption: "Payment for a Cellular topup"
  }
}
```

## Changing a Hold

> While money is on-hold you can change any payment details, for eg. to refund some part of money

```
PUT /projects/:project_id/holds/:hold_id
{
  total: 20.00
}
```

## Cancel a Hold

> After hold payment can be completed to commit balance change and turn hold into charge or declined to remove hold and return funds to available balance.

```
POST /projects/:project_id/holds/:hold_id/decline
```

## Complete Hold into a Transfer

On hold completion we will add same ```metadata``` to a Transfer.

```
POST /projects/:project_id/holds/:hold_id/complete
```

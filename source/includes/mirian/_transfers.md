# Transfer

You should use Transfers whenever you accept a payment or transfer money from one account to another. Creating Transfer will decrease Account balance by transfer total.

## List all Transfers

```
GET /projects/:project_id/transfers
```

### Create a Transfer

To create a Transfer you need to specify at least two request fields:

- ```transfer``` - List of Transfers that should be created in a single Transfer. This allows you to take fees from your customers.
- ```total``` - Total amount of created transfers. Total should be exactly same as sum of all created transfers. (Otherwise we will return an appropriate error.)

You can attach any ```metadata``` to a Transfer.

```
POST /projects/:project_id/transfers
{
  total: 100,
  transfer: [
    {subtotal: 90, destination: "<service_account_id>", meta: {service_id: 1, service_name: 'Cellular Topup'}}
    {subtotal: 10, destination: "<fees_account_id>", meta: {for: "service_payment", service_id 1}}
  ],
  meta: {}
}
```

## Rollback a Transfer

On a Transfer rollback we will create new Transfer from a destination to a source accounts with same totals. This means that we will also return all the fees you charged from a Account. Created Transfer will have ```is_rollback``` field set to true and ```rollback_refference``` set to a original Transfer ID.

Also rollbacked transfer will have a ```rollback_transfer``` property that will hold rollback Transfer ID and a ```is_rollbacke``` field set to ```true```.

## Create Refund for a Transfer

Refund is similar to a Rollback, but you need to specify refund total for every account that received funds. This allows you to refund funds by keeping the fees or to refund it with all the fees.

## Transferring money between projects

(TODO: Should we handle it or leave it for developers?)

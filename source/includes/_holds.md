# Holds

## Funding an Account

(TODO: Move this to a Fundings entity?)

(TODO: Allow subpayments on funding to charge front fee?)

(TODO: Holds should be part of Transfers API response, so consumers can return correct payment history to clients.)

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

## Convert Hold into Transfer

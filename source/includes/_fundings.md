# Fundings

Funding allows to top-up any account balance in a system. (Basically this is an equivalent for an money emission.)

You can find best practices for [Funding an Account](http://nebo15.github.io/qbill.docs/#funding-account).

## Create a Funding Operation, Fund an Account

```
POST /projects/:project_id/fundings
{
  account_id: <account_id>,
  total: 1000
}
```

## List all Funding Operations

```
GET /projects/:project_id/fundings
```

## Canceling a Funding Operation

All Fundings can't be canceled, to do so just create a Transaction and move money to a system account. You can create some sort of ```/dev/null``` account for this purpose.

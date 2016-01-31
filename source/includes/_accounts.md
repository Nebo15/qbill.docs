# Accounts

Account is a one a base entities that represents any object with a balance. This balance can be in any currency, you can add a currency code in ```metadata``` object. (You find info at [Metadata](#metadata) section.)

(TODO: Add information about in-built currencies.)

(TODO: Add overdraft limit to an Account. ```overdraft``` field that hold maximum negative balance for this user.)

## List all Accounts

```
GET /projects/:project_id/accounts
```

## Create an Account

```
POST /projects/:project_id/accounts
{
  metadata: {
    external_id: 192838,
    currency_code: 'USD'
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
  },
  "data": {
    id: "acc_388djejje88du"
    balance: 0,
    metadata: {
      external_id: 192838,
      currency_code: 'USD'
    }
  }
}
```

## Get all Account data

```
GET /projects/:project_id/accounts/:id
```

## List all Account Funding Operations

This is a shortcut to [List all Fundings](#List all Fundings) with an filter based on account id.

```
GET /projects/:project_id/accounts/:id/fundings
```

## List all Account Holds

This is a shortcut to [List all Holds](#list-all-holds) with an filter based on account id.

```
GET /projects/:project_id/accounts/:id/holds
```

## List all Account Transfers

This is a shortcut to [List all Transfers](#list-all-transfers) with an filter based on account id.

```
GET /projects/:project_id/accounts/:id/transfers
```

## Disabling and Enabling an Account
Accounts can't be deleted, but can be disabled to prevent its future usage. Requesting or linking a disabled account will always result a ```HTTP 403``` error.

### Disabling

```
PUT /projects/:project_id/accounts/:id
{
  is_disabled: true
}
```

### Enabling

You can enable account to continue using it later.

```
PUT /projects/:project_id/accounts/:id
{
  is_disabled: false
}
```

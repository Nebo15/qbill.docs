# Accounts

Account is a one a base entities that represents any object with a balance. This balance can be in any currency. If you have multiple currencies, they will be converted automatically.

## List all Accounts

```
GET /v1/accounts
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
POST /v1/accounts/:id/disable
```

### Enabling

You can enable account to continue using it later.

```
POST /v1/accounts/:id/enable
```

---
title: QBill API Reference - Applications

language_tabs:
  - shell

toc_footers:
 - <a href='index.html'>Accounting API - Mirian</a>
 - <a href='app.html'>Projects API - Mirian SaaS</a>
 - <a href='arkenstone.html'>Deployment API - Arkenstone</a>
 - <a href='palantir.html'>API Service Bus - Palantir</a>
 - <a href='gandalf.html'>Risk Management - Gandalf</a>

search: true
---


# Tokenization

## Introduction

This is an API of a card tokenization service that was inspired by a [Visa Token Service](https://developer.visa.com/products/vts/reference#vts__vts____provision_token) API. It allows to exchange card data with a token, that can be later used to create transactions.

## Security

### Defining Card Token expiration date

### Data Storage Policy

Card Data | Policy
--------- | ------
PAN | Stored in DB for a token lifespan. It's stored in an encrypted format.
Expiration Date | Stored inside server's RAM memory and can be lost on reboots.
Holder | Stored inside server's RAM memory and can be lost on reboots.
CVV | Never stored. (**TODO: How to make payments without CVV?! Store it in RAM?**)

## Enroll PAN

### Enroll via API (internal)

### Enroll via iFrame

### Enroll via Tokenize.js

## Get Token Status

## Suspend Token

## Resume Token

## Delete Token

## Get Card Meta

## Exchange token to Card Data (internal)

# Funding

## Introduction

This is an API to card2card service that was inspired by a [Visa Direct](https://developer.visa.com/products/visa_direct/reference#visa_direct__funds_transfer__v1__pullfunds) and [MasterCard Money Send](https://developer.mastercard.com/portal/display/api/MoneySend) API's. It allows to transfer money between cards via card PAN's or Card Tokens.

## List of available aquiers

- TAS
- UPC
- ALFA

## Funding

### Funding with a Card Number

### Funding with a Card Token

### MultiFunding

## Payment

### Payment with a Card Number

### Payment with a Card Token

### MultiPayment

## Reverse

### MultiReverse

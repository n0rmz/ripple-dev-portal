#INTRODUCTION

## Ripple-REST API

The [`ripple-rest`](http://github.com/ripple/ripple-rest) API is RESTful interface built to communicate directly to `rippled`. Our API is designed to have predictable, resource-oriented URLs and uses HTTP response codes to indicate any API errors.

Available API Routes:

+ `GET /api/v1/addresses/:address/next_notification`
+ `GET /api/v1/addresses/:address/next_notification/:prev-hash`
+ `GET /api/v1/addresses/:address/payments/:dst_address/:dst_amount`
+ `POST /api/v1/addresses/:address/payments`
+ `GET /api/v1/addresses/:address/payments/:hash`
+ `GET /api/v1/addresses/:address/txs/:hash`
+ `GET /api/v1/status`
+ `GET /api/v1/server/connected`


## Ripple Concepts

Ripple is an internet protocol for financial transactions. It uses a shared ledger to track balances among parties no matter how physically far apart they may be. This allows each payments to clear in seconds rather than days. It also makes it possible for a person who uses one currency to seamlessly pay a person who uses a different currency. Learn more about it on the [Ripple Wiki](https://ripple.com/wiki/Ripple_Introduction).

### Ripple Address

A Ripple Address is an entry in the Ledger. People typically have one Address that stores their Ripple credits, IOUs and the trust paths granted to and from other accounts. Each address has a private key. Anyone that knows an addresses' private key can authorize transactions from that address.

>Sample Ripple Address: rpvfJ4mR6QQAeogpXEKnuyGBx8mYCSnYZi

Learn more about the [Ripple Address](https://ripple.com/wiki/Account).

### Transaction Types

The Ripple protocol supports multiple types of transactions other than just payments. Transactions are considered to be any changes to the database made on behalf of a Ripple Address. Transactions are first constructed and then submitted to the network. After transaction processing, meta data is associated with the transaction which itemizes the resulting changes to the ledger.

+ `Payment` - Payment transactions is an authorized transfer of balance from one address to another.
+ `Trustline` - Trustline transactions is an authorized grant of trust between two addresses.

##Getting Started

### Setup

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

### Making a call

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

### Errors

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

### API Objects

#### 1. Amount

All currencies on the Ripple Network have issuers, except for XRP. In the case of XRP, the `"issuer"` field may be omitted or set to `""`. Otherwise, the `"issuer"` must be a valid Ripple address of the gateway that issues the currency.

For more information about XRP see [the Ripple Wiki page on XRP](https://ripple.com/wiki/XRP). For more information about using currencies other than XRP on the Ripple Network see [the Ripple Wiki page for gateways](https://ripple.com/wiki/Ripple_for_Gateways).

>Amount Object:

```js
{
  "value": "1.0",
  "currency": "USD",
  "issuer": "r..."
}
```
Or for XRP:
```js
{
  "value": "1.0",
  "currency": "XRP",
  "issuer": ""
}
```
All currencies on the Ripple Network have issuers, except for XRP. In the case of XRP, the `"issuer"` field may be omitted or set to `""`. Otherwise, the `"issuer"` must be a valid Ripple address of the gateway that issues the currency.

Note that the `value` can either be specified as a string or a number. Internally this API uses a BigNumber library to retain higher precision if numbers are inputted as strings.

For more information about XRP see [the Ripple Wiki page on XRP](https://ripple.com/wiki/XRP). For more information about using currencies other than XRP on the Ripple Network see [the Ripple Wiki page for gateways](https://ripple.com/wiki/Ripple_for_Gateways).

#### 2. Payment

The `Payment` object is a simplified version of the standard Ripple transaction format. 

This `Payment` format is intended to be straightforward to create and parse, from strongly or loosely typed programming languages. Once a transaction is processed and validated it also includes information about the final details of the payment.

The following fields are the minimum required to submit a `Payment`:

+ `source_transaction_id` is a required field that is used to prevent duplicate payments and confirm that payments have been validated. If `ripple-rest` is in the process of submitting a payment with a given `source_transaction_id` and the same user submits another payment with the same `source_transaction_id` before they have received a `Notification` about the first one, `ripple-rest` will NOT submit the second payment to `rippled`. The `source_transaction_id` should also be used to confirm that outgoing payments have been validated and written into the Ripple Ledger by looking for the `source_transaction_id` submitted here in `Notification`s of validated payments. See the [Guide](GUIDE.md) for more information 
+ `destination_amount` is an [`Amount` object](#1-amount)

```js
{
  "source_address": "rKXCummUHnenhYudNb9UoJ4mGBR75vFcgz",
  "source_transaction_id": "12345",
  "destination_address": "rNw4ozCG514KEjPs5cDrqEcdsi31Jtfm5r",
  "destination_amount": {
    "value": "0.001",
    "currency": "XRP",
    "issuer": ""
  }
}
```

The full set of fields accepted on `Payment` submission is as follows:

+ `src_tag` is an optional unsigned 32 bit integer (0-4294967294, inclusive) that is generally used if the sender is a hosted wallet at a gateway. This should be the same as the `dst_tag` used to identify the hosted wallet when they are receiving a payment.
+ `dst_tag` is an optional unsigned 32 bit integer (0-4294967294, inclusive) that is generally used if the receiver is a hosted wallet at a gateway
+ `src_slippage` can be specified to give the `src_amount` a cushion and increase its chance of being processed successfully. This is helpful if the payment path changes slightly between the time when a payment options quote is given and when the payment is submitted. The `src_address` will never be charged more than `src_slippage` + the `value` specified in `src_amount`
+ `invoice_id` is an optional 256-bit hash field that can be used to link payments to an invoice or bill
+ `paths` is a "stringified" version of the Ripple PathSet structure. Most users of this API will want to treat this field as opaque. See the [Ripple Wiki](https://ripple.com/wiki/Payment_paths) for more information about Ripple pathfinding
+ `flag_no_direct_ripple` is a boolean that can be set to `true` if `paths` are specified and the sender would like the Ripple Network to disregard any direct paths from the `src_address` to the `dst_address`. This may be used to take advantage of an arbitrage opportunity or by gateways wishing to issue balances from a hot wallet to a user who has mistakenly set a trustline directly to the hot wallet. Most users will not need to use this option.
+ `flag_partial_payment` is a boolean that, if set to true, indicates that this payment should go through even if the whole amount cannot be sent because of a lack of liquidity or funds in the `src_address` account. The vast majority of senders will never need to use this option.

```js
{
    /* User Specified */

    "source_address": "rKXCummUHnenhYudNb9UoJ4mGBR75vFcgz",
    "source_tag": "",
    "source_transaction_id": "12345",
    "source_amount": {
        "value": "0.001",
        "currency": "XRP",
        "issuer": ""
    },
    "source_slippage": "0",
    "destination_address": "rNw4ozCG514KEjPs5cDrqEcdsi31Jtfm5r",
    "destination_tag": "",
    "destination_amount": {
        "value": "0.001",
        "currency": "XRP",
        "issuer": ""
    },

    /* Advanced Options */

    "invoice_id": "",
    "paths": "[]",
    "no_direct_ripple": false,
    "partial_payment": false
}
```


When a payment is confirmed in the Ripple ledger, it will have additional fields added:

+ `tx_result` will be `tesSUCCESS` if the transaction was successfully processed. If it was unsuccessful but a transaction fee was claimed the code will start with `tec`. More information about transaction errors can be found on the [Ripple Wiki](https://ripple.com/wiki/Transaction_errors).
+ `tx_timestamp` is the UNIX timestamp for when the transaction was validated
+ `tx_fee` is the network transaction fee charged for processing the transaction. For more information on fees, see the [Ripple Wiki](https://ripple.com/wiki/Transaction_fees). In the standard Ripple transaction format fees are expressed in drops, or millionths of an XRP, but for clarity the new formats introduced by this API always use the full XRP unit.
+ `tx_src_bals_dec` is an array of [`Amount`](#1-amount) objects representing all of the balance changes of the `src_address` caused by the payment. Note that this includes the `tx_fee`
+ `tx_dst_bals_inc` is an array of [`Amount`](#1-amount) objects representing all of the balance changes of the `dst_address` caused by the payment

```js
{
    /* ... */

    /* Generated After Validation */

    "direction": "outgoing",
    "state": "validated",
    "result": "tesSUCCESS",
    "ledger": 4696959,
    "hash": "55BA3440B1AAFFB64E51F497EFDF2022C90EDB171BBD979F04685904E38A89B7",
    "timestamp": 1391025100000,
    "timestamp_human": "2014-01-29T19:51:40.000Z",
    "fee": "0.000012",
    "source_balance_changes": [{
        "value": "-0.001012",
        "currency": "XRP",
        "issuer": ""
    }],
    "destination_balance_changes": [{
        "value": "0.001",
        "currency": "XRP",
        "issuer": ""
    }]
}
```

#### 3. Notification

Notifications are new type of object not used elsewhere on the Ripple Network but intended to simplify the process of monitoring accounts for new activity.

If there is a new `Notification` for an account it will contain information about the type of transaction that affected the account, as well as a link to the full details of the transaction and a link to get the next notification. 


If there is a new `notification` for an account, it will come in this format:

+ `timestamp` is the UNIX timestamp for when the transaction was validated, or the number of milliseconds since January 1st, 1970 (00:00 UTC)
+ `timestamp_human` is the transaction validation time represented in the format `YYYY-MM-DDTHH:mm:ss.sssZ`. The timezone is always UTC as denoted by the suffix "Z"
+ `transaction_url` is a URL that can be queried to retrieve the full details of the transaction. If it the transaction is a payment it will be returned in the `Payment` object format, otherwise it will be returned in the standard Ripple transaction format
+ `next_notification_url` is a URL that can be queried to get the notification following this one for the given address
+ `source_transaction_id` will be the same as the `source_transaction_id` originally submitted by the sender. Senders should look for the `source_transaction_id`'s of payments they have submitted to `ripple-rest` amongst `Notification`s of validated payments. If the `source_transaction_id` of a particular payment appears in a `Notification` with the `state` listed as `validated`, then that payment has been successfully written into the Ripple Ledger

```js
{
  "address": "rKXCummUHnenhYudNb9UoJ4mGBR75vFcgz",
  "type": "payment",
  "direction": "outgoing",
  "state": "validated",
  "result": "tesSUCCESS",
  "ledger": 4696959,
  "hash": "55BA3440B1AAFFB64E51F497EFDF2022C90EDB171BBD979F04685904E38A89B7",
  "timestamp": 1391025100000,
  "timestamp_human": "2014-01-29T19:51:40.000Z",
  "transaction_url": "http://ripple-rest.herokuapp.com/api/v1/addresses/rKXCummUHnenhYudNb9UoJ4mGBR75vFcgz/payments/55BA3440B1AAFFB64E51F497EFDF2022C90EDB171BBD979F04685904E38A89B7",
  "next_notification_url": "http://ripple-rest.herokuapp.com/api/v1/addresses/rKXCummUHnenhYudNb9UoJ4mGBR75vFcgz/next_notification/55BA3440B1AAFFB64E51F497EFDF2022C90EDB171BBD979F04685904E38A89B7",
  "source_transaction_id": "12345",
}
```

If there are no new notifications, the empty `Notification` object will be returned in this format:

+ `type` will be `none` if there are no new notifications
+ `tx_state` will be `pending` if there are still transactions waiting to clear and `empty` otherwise
+ `next_notification_url` will be provided whether there are new notifications or not so that that field can always be used to query the API for new notifications.

```js
{
  "address": "rKXCummUHnenhYudNb9UoJ4mGBR75vFcgz",
  "type": "none",
  "direction": "",
  "state": "empty",
  "result": "",
  "ledger": "",
  "hash": "",
  "timestamp": "",
  "timestamp_human": "",
  "transaction_url": "",
  "next_notification_url": "http://ripple-rest.herokuapp.com/api/v1/addresses/rKXCummUHnenhYudNb9UoJ4mGBR75vFcgz/next_notification/55BA3440B1AAFFB64E51F497EFDF2022C90EDB171BBD979F04685904E38A89B7",
  "source_transaction_id": "",
}
```

#PAYMENTS

The most common use case for Ripple is to send a payment to another Ripple Address. The following section details the endpoints and calls needed to prepare, submit, and confirm a payment.

## Preparing a Payment

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

## Submitting a Payment

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

## Confirming a Payment

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

#PAYMENT MONITORING

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

## Retrieving a Payment

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

#GENERIC RIPPLE TRANSACTIONS

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

#RIPPLED SERVER STATUS

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

## Check 'rippled' Status

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

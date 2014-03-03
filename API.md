#INTRODUCTION

## Ripple-REST API(BETA)

The `ripple-rest` API makes it easy to access the Ripple system via a RESTful web interface.  In this section, we will cover the concepts you need to understand, and get you started accessing the API and learning how to use it.

While there are different APIs that you can use, for example by accessing the `rippled` server directly via a web socket, this documentation focuses on the `ripple-rest` API as this is the high-level API recommended for working with the Ripple system.

## Ripple Concepts

Ripple is an internet protocol for financial transactions. You can use Ripple to send money anywhere in the world, in any currency, instantly and for free. Learn more about it on the [Ripple Wiki](https://ripple.com/wiki/Ripple_Introduction).

### Ripple Address

In the Ripple world, each account is identified by a Ripple ___address___.  A ripple address is a string that uniquely identifies an account, for example: `rNsJKf3kaxvFvR8RrDi9P3LBk2Zp6VL8mp`

Learn more about the [Ripple Address](https://ripple.com/wiki/Account).

### Transaction Types

The Ripple protocol supports multiple types of transactions other than just payments. Transactions are considered to be any changes to the database made on behalf of a Ripple Address. Transactions are first constructed and then submitted to the network. After transaction processing, meta data is associated with the transaction which itemizes the resulting changes to the ledger.

+ `Payment` - Payment transactions is an authorized transfer of balance from one address to another.
+ `Trustline` - Trustline transactions is an authorized grant of trust between two addresses.

### Sending Payments

Sending a payment involves three steps:

1. You need to create the payment object.  If the payment is to be made in a currency other than XRP, the Ripple system will identify the chain of trust, or ___path___, that connects the source and destination accounts; when creating the payment, the `ripple-rest` API will automatically find the set of possible paths for you.

2. You can modify the payment object if necessary, and then ___submit___ it to the API for processing.

3. Finally, you can check to see if your payment has gone through by looking for the appropriate ___notification___.

You can also use notifications to see when a payment has been received.

While the `ripple-rest` API provides a high-level interface for sending and receiving payments, there are other endpoints within the API that you can use to work with generic ripple transactions, and to check the status of the Ripple server.

##Getting Started

### Setup

Before you can use the `ripple-rest` API, you will need to have two things:

 * An activated Ripple account.  If you don't have a Ripple account, you can use the Ripple web client to create one, as described in the [Client manual](https://ripple.com/wiki/Client_Manual).  Make sure you have a copy of the Ripple address for your account; the address can be found by clicking on the __Receive__ tab in the web client.
 
 * The URL of the server running the `ripple-rest` API that you wish to use.  In this documentation, we will assume that the server is running at [https://ripple-rest.herokuapp.com](https://ripple-rest.herokuapp.com), which is the URL for a test version of the server.  When you follow the examples below, make sure that you replace this with the URL for the server you want to access. Please remember to only use 
 
As a programmer, you will also need to have a suitable HTTP client library that allows you to make secure HTTP (`HTTPS`) requests.  To follow the examples below, you will need to have access to the `curl` command-line tool.


### Exploring the API

Let's start by using `curl` to see if the `ripple-rest` API is currently running.  Type the following into a terminal window:

`curl https://ripple-rest.herokuapp.com/api/v1/status`

After a short delay, the following response should be displayed:

```js
{
       "api_server_status": "online",
       "rippled_server_url": "wss://s_west.ripple.com:443",
       "rippled_server_status": {
         "info": {
           "build_version": "0.21.0-rc2",
           "complete_ledgers": "32570-5254146",
           "hostid": "WEAN",
           "last_close": {
             "converge_time_s": 2.022,
             "proposers": 5
           },
           "load_factor": 1,
           "peers": 52,
           "pubkey_node": "n9LVyJ9GGBwHeeZ1bwPQUKj5P6vyD5tox2ozMPadMDvXx8CrPPmJ",
           "server_state": "full",
           "validated_ledger": {
             "age": 6,
             "base_fee_xrp": 1.0e-5,
             "hash": "ADF8BEFA91F4D355C60AE37E7ED79E91591704D052114F2BBDB6AF892E5E749E",
             "reserve_base_xrp": 20,
             "reserve_inc_xrp": 5,
             "seq": 5254146
           },
           "validation_quorum": 3
         }
       },
       "api_documentation_url": "https://github.com/ripple/ripple-rest"
     }
```
#### Using the API ####

The `ripple-rest` API conforms to the following general behavior for a web interface:

* The HTTP method identifies what you are trying to do.  Generally, HTTP `GET` requests are used to retrieve information, while HTTP `POST` requests are used to make changes or submit information.

* You make HTTP (or HTTPS) requests to the API endpoint, including the desired resources within the URL itself.

* If more complicated information needs to be sent, it will be included as JSON-formatted data within the body of the HTTP POST request.

* Upon completion, the server will return an HTTP status code of 200 (OK), and a `Content-Type` value of `application/json`.  The body of the response will be a JSON-formatted object containing the information returned by the endpoint.

* The returned JSON object will include a `success` field indicating whether the request was successful or not.

* If an error occurred, the returned object will include `error` and `message` fields, where `error` is a short string identifying the error that occurred, and `message` will be a longer human-readable string explaining what went wrong.

### Errors

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

### API Objects

#### 1. Amount

All currencies on the Ripple Network have issuers, except for XRP. In the case of XRP, the `"issuer"` field may be omitted or set to `""`. Otherwise, the `"issuer"` must be a valid Ripple address of the gateway that issues the currency.

For more information about XRP see [the Ripple Wiki page on XRP](https://ripple.com/wiki/XRP). For more information about using currencies other than XRP on the Ripple Network see [the Ripple Wiki page for gateways](https://ripple.com/wiki/Ripple_for_Gateways).

`Amount Object:`

```js
{
  "value": "1.0",
  "currency": "USD",
  "issuer": "r..."
}
```
`Or for XRP:`

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

## Available API Routes:

+ [`GET /api/v1/addresses/:address/next_notification`](#preparing-a-payment)
+ [`GET /api/v1/addresses/:address/next_notification/:prev-hash`](#submitting-a-payment)
+ `GET /api/v1/addresses/:address/payments/:dst_address/:dst_amount`
+ `POST /api/v1/addresses/:address/payments`
+ `GET /api/v1/addresses/:address/payments/:hash`
+ `GET /api/v1/addresses/:address/txs/:hash`
+ `GET /api/v1/status`
+ `GET /api/v1/server/connected`

# PAYMENTS

## Outgoing Payments

`ripple-rest` provides access to `ripple-lib`'s robust transaction submission processes. This means that it will set the fee, manage the transaction sequence numbers, sign the transaction with your secret, and resubmit the transaction up to 10 times if `rippled` reports an initial error that can be solved automatically.

### Preparing a Payment

#### `GET /api/v1/addresses/:address/payments/:dst_address/:dst_amount`

A payment needs to be formatted in the following order with the payment object to be submitted as a valid payment.

```js
{
  "secret": "s...",
  "payment": { /* Payment object */ }
}
```

The payment object itself can be prepared manually for any transactions that are occuring directly with XRP or if there are already established trustlines between the two parties for the currency being transferred. In most cases, a payment object can be created automatically by performing a `GET` on the payments endpoint.

This call generates possible payments for a given set of parameters. This is a wrapper around the [Ripple path-find command](https://ripple.com/wiki/RPC_API#path_find) that returns an array of [`Payment Objects`](#2-payment), which can be submitted directly to [`POST /api/v1/addresses/:address/payments`](#post-apiv1addressesaddresspayments).

This uses the [`Payment` Object format](#2-payment).

The `:dst_amount` parameter uses `+` to separate the `value`, `currency`, and `issuer` fields. For XRP the format is `0.1+XRP` and for other currencies it is `0.1+USD+r...`, where the `r...` is the Ripple address of the currency's issuer.

__NOTE:__ This command may be quite slow. If the command times out, please try it again.

### Submitting a Payment

#### `POST /api/v1/addresses/:address/payments`

Submit a payment in the [`Payment` Object](#2-payment) format.

__DO NOT SUBMIT YOUR SECRET TO AN UNTRUSTED REST API SERVER__ -- this is the key to your account and your money. If you are using the test server provided, only use test accounts to submit payments.

Request JSON:
```js
{
  "secret": "s...",
  "payment": { /* Payment */ }
}
```

Response:

```js
{
    "success": true,
    "confirmation_token": "55BA3440B1AAFFB64E51F497EFDF2022C90EDB171BBD979F04685904E38A89B7"
}
```
Or if there is a problem with the transaction:

```js
{
  "success": false,
  "error": "tecPATH_DRY",
  "message": "Path could not send partial amount. Please ensure that the src_address has sufficient funds (in the src_amount currency, if specified) to execute this transaction."
}
```
More information about transaction errors can be found on the [Ripple Wiki](https://ripple.com/wiki/Transaction_errors). Save the `confirmation_token` to check for transaction confirmation by matching that against new `notification`'s. Payments cannot be cancelled once they are submitted.

### Confirming a Payment

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

#PAYMENT MONITORING

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

## Retrieving a Payment

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

#RIPPLE TRANSACTIONS

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

#RIPPLED SERVER STATUS

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

## Check 'rippled' Status

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

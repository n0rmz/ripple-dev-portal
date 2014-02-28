# Ripple Developer Portal #

## Introduction ##

The `ripple-rest` API makes it easy to access the Ripple system via a RESTful web interface.  In this section, we will cover the concepts you need to understand, and get you started accessing the API and learning how to use it.

While there are different APIs that you can use, for example by accessing the `rippled` server directly via a web socket, this documentation focuses on the `ripple-rest` API as this is the high-level API recommended for working with the Ripple system.

### Ripple Concepts ###

Ripple is a system for making financial transactions.  You can use Ripple to send money anywhere in the world, in any currency, instantly and for free.

In the Ripple world, each account is identified by a Ripple ___address___.  A ripple address is a string that uniquely identifies an account, for example: `rNsJKf3kaxvFvR8RrDi9P3LBk2Zp6VL8mp`

A Ripple ___payment___ can be sent using Ripple's native currency, XRP, directly from one account to another.  Payments can also be sent in other currencies, for example US dollars, Euros, Pounds or Bitcoins, though the process is slightly more complicated.

Payments are made between two accounts, by specifying the ___source___ and ___destination___ address for those accounts.  A payment also involves an ___amount___, which includes both the numeric amount and the currency, for example: `100+XRP`.

When you make a payment in a currency other than XRP, you also need to include the Ripple address of the ___issuer___.  The issuer is the gateway or other entity who holds the foreign-currency funds on your behalf.  For foreign-currency payments, the amount will look something like this: `100+USD+rNsJKf3kaxvFvR8RrDi9P3LBk2Zp6VL8mp`.

Sending a payment involves three steps:

1. You need to create the payment object.  If the payment is to be made in a currency other than XRP, the Ripple system will identify the chain of trust, or ___path___, that connects the source and destination accounts; when creating the payment, the `ripple-rest` API will automatically find the set of possible paths for you.

2. You can modify the payment object if necessary, and then ___submit___ it to the API for processing.

3. Finally, you can check to see if your payment has gone through by looking for the appropriate ___notification___.

You can also use notifications to see when a payment has been received.

While the `ripple-rest` API provides a high-level interface for sending and receiving payments, there are other endpoints within the API that you can use to work with generic ripple transactions, and to check the status of the Ripple server.

### Getting Started ###

Before you can use the `ripple-rest` API, you will need to have two things:

 * An activated Ripple account.  If you don't have a Ripple account, you can use the Ripple web client to create one, as described in the [Client manual](https://ripple.com/wiki/Client_Manual).  Make sure you have a copy of the Ripple address for your account; the address can be found by clicking on the __Receive__ tab in the web client.
 
 * The URL of the server running the `ripple-rest` API that you wish to use.  In this documentation, we will assume that the server is running at [https://ripple-rest.herokuapp.com](https://ripple-rest.herokuapp.com), which is the URL for a test version of the server.  When you follow the examples below, make sure that you replace this with the URL for the server you want to access.
 
As a programmer, you will also need to have a suitable HTTP client library that allows you to make secure HTTP (`HTTPS`) requests.  To follow the examples below, you will need to have access to the `curl` command-line tool.

#### Exploring the API ####

Let's start by using `curl` to see if the `ripple-rest` API is currently running.  Type the following into a terminal window:

     curl https://ripple-rest.herokuapp.com/api/v1/status

After a short delay, the following response should be displayed:

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

<div style="margin-left: 20px; margin-right: 20px; padding: 10px; border: 1px solid gray">
  <i>
    In these examples, we assume that the API is running at
    <span style="font-family:monospace; font-weight:bold">
      https://ripple-rest.herokuapp.com
    </span>
    &mdash; don't forget to replace this with the appropriate URL for the server you want
    to access.
   </i>
</div>
<p/>

As you can see from the `api_server_status` field, the server is currently online.

Now let's try a slightly more complicated example.  We will attempt to create a payment between two dummy Ripple accounts -- obviously, we won't submit this payment, but it does show how a payment can be created.

Type the following into your terminal window:

    curl https://ripple-rest.herokuapp.com/api/v1/addresses/rKXCummUHnenhYudNb9UoJ4mGBR75vFcgz/payments/rNsJKf3kaxvFvR8RrDi9P3LBk2Zp6VL8mp/100+XRP

<div style="margin-left: 20px; margin-right: 20px; padding: 10px; border: 1px solid gray">
  <i>This URL has the following basic structure:</i>
  <p/>
  <div style="font-family:monospace; margin-left: 20px">
    /api/v1/addresses/&lt;src_address&gt;/payments/&lt;dst_address&gt;/&lt;amount&gt;
  </div>
  <div style="height: 10px"></div>
  <i>So in this example we are sending 100 XRP from account</i>
  <code>rKXCummUHnenhYudNb9UoJ4mGBR75vFcgz</code>
  <i>to account</i>
  <code>rNsJKf3kaxvFvR8RrDi9P3LBk2Zp6VL8mp</code><i>.</i>
</div>

You should get the following response:

    {
      "success": true,
      "payments": [
        {
          "src_address": "rKXCummUHnenhYudNb9UoJ4mGBR75vFcgz",
          "src_tag": "",
          "src_amount": {
            "value": "100",
            "currency": "XRP",
            "issuer": ""
          },
          "src_slippage": "1",
          "dst_address": "rNsJKf3kaxvFvR8RrDi9P3LBk2Zp6VL8mp",
          "dst_tag": "",
          "dst_amount": {
            "value": "100",
            "currency": "XRP",
            "issuer": ""
          },
          "dst_slippage": "0",
          "invoice_id": "",
          "paths": "[]",
          "flag_partial_payment": false,
          "flag_no_direct_ripple": false
        }
      ]
    }

As you can see this, creates a single payment object which would allow you to send 100 XRP from one account to another.  Various default values have been set for this payment; you can change these defaults if you wish before submitting the payment.

<div style="margin-left: 20px; margin-right: 20px; padding: 10px; border: 1px solid gray">
  <i>
    Because we are creating a payment in XRP, only one option is included for making the
    payment.  If the payment was in a different currency, multiple payment options may be
    provided, with different fees and paths linking the source and destination accounts.
  </i>
</div>

#### Using the API ####

The `ripple-rest` API conforms to the following general behavior for a web interface:

* The HTTP method identifies what you are trying to do.  Generally, HTTP `GET` requests are used to retrieve information, while HTTP `POST` requests are used to make changes or submit information.

* You make HTTP (or HTTPS) requests to the API endpoint, including the desired resources within the URL itself.

* If more complicated information needs to be sent, it will be included as JSON-formatted data within the body of the HTTP POST request.

* Upon completion, the server will return an HTTP status code of 200 (OK), and a `Content-Type` value of `application/json`.  The body of the response will be a JSON-formatted object containing the information returned by the endpoint.

* The returned JSON object will include a `success` field indicating whether the request was successful or not.

* If an error occurred, the returned object will include `error` and `message` fields, where `error` is a short string identifying the error that occurred, and `message` will be a longer human-readable string explaining what went wrong.

### API Objects ###

The `ripple-rest` API makes use of the following data structures to represent various objects within the Ripple system:

#### [Amounts](id:amount_object) ####

The `Amount` object represents an amount to be transferred, for example:

    {
     "value": "100",
     "currency": "XRP"
    }

The Amount object has the following fields:

 * `value`.  This is the actual amount to be transferred, in the specified currency.  The value can be specified either as a number or as a string; using a string ensures that no rounding error occurs when the JSON data is decoded.
 
 * `currency`.  This is a three-letter code indicating which currency the amount is in.  For XRP, the currency value should be set to <span style="font-family:monospace">XRP</span>.
 
 * `issuer`.  This is the Ripple address of the gateway that issued the currency.  This has no meaning for XRP amounts, and the issuer can be omitted or left blank in this case.  For all other currencies, this field must be supplied.

#### [Payments](id:payment_object) ####

The `Payment` object represents a payment within the Ripple system.  The following is an example of a minimal payment:

    {
      "src_address": "rKXCummUHnenhYudNb9UoJ4mGBR75vFcgz",
      "dst_address": "rNw4ozCG514KEjPs5cDrqEcdsi31Jtfm5r",
      "dst_amount": {
        "value": "100",
        "currency": "XRP"
      }
    }

At a minimum, the following fields are __required__ when you submit a payment:

 * `src_address`.  This is the Ripple address for the source account, as a string.
 
 * `dst_address`.  This is the Ripple address for the destination account, as a string.
 
 * `dst_amount`.  This is an [Amount](#amount_object) object representing the amount that should be deposited into the destination account.

The following __optional__ fields can also be included in a submitted payment:

* `src_tag`.  This value is generally only used when the sender is a hosted wallet at a gateway; it identifies which of the gateway's hosted accounts the funds should come out of.  Note that if this is used, it must be an unsigned 32-bit integer value.

* `dst_tag`.  This value is generally only used when the receiver is a hosted wallet at a gateway; it idenfities which of the gateway's hosted accounts the funds should go into.  If this value is used, it must be a 32-bit integer value.

* `src_amount`.   This is an [Amount](#amount_object) object representing the amount that should be deducted from the source account.

* `src_slippage`.  This lets you add a "cushion" to the source amount, increasing the chance that the payment will go through if the payment path changes between when you create the Payment object and when you submit it.  Note that the source account will never be charged more than <span style="font-family:monospace">src_slippage + src_amount.value</span>

* `invoice_id`.  This is an optional string which can be used to link the payment to an invoice or bill.  It is intended that this field contains a 32-character hexadecimal hash value.

* `paths`.  This is a string containing the path to use between the source and destination accounts.  Note that this is an opaque data structure, and should not be changed by the user.

* `flag_no_direct_ripple`.  Setting this field to <span style="font-family:monospace">true</span> will force the Ripple network to disregard any direct paths between the source and destination accounts.  This is only useful in certain special situations, and most users will never need to set this field.

* `flag_partial_payment`.  Setting this field to <span style="font-family:monospace">true</span> will ensure the payment goes through even if the entire amount is not available because of insufficient liquidity in the source account.  Most users will never need to set this field.

When the payment has been confirmed in the Ripple ledge, the following additional fields will be included in the Payment object:

* `tx_direction`.  The direction of the payment (<span style="font-family:monospace">outgoing</span> or <span style="font-family:monospace">incoming</span>).

* `tx-state`.  This indicates the current state of the payment.  This can have the values <span style="font-family:monospace">confirmed</span>, <span style="font-family:monospace">pending</span> or <span style="font-family:monospace">failed</span>.

* `tx_result`.  The final result of the payment.  If the payment went through successfully, this will have the value <span style="font-family:monospace">tesSUCCESS</span>.  If the payment failed, but a fee was imposed, then the code will start with <span style="font-family:monospace">tec</span>.  More information on transaction errors can be found in the [Ripple Wiki](https://ripple.com/wiki/Transaction_errors).

* `tx_ledger`.  ????

* `tx_hash`.  A string uniquely identifying this transaction within the Ripple ledger.

* `tx_timestamp`.  This is the date and time at which the payment was processed, as an integer number of milliseconds since the 1st of January 1970, in UTC.

* `tx_timestamp_human`.  A human-readable version of the `tx_timestamp` value, as a string.

* `tx_fee`.  This is the network transaction fee charged for processing the payment.

* `tx_src_bals_dec`.  This is an array of [Amount](#amount_object) objects representing all the balance changes caused by this payment to the source account.  This includes the transaction processing fee.  For example:

      "tx_src_bals_dec": [{
          "value": "-0.001012",
          "currency": "XRP",
          "issuer": ""
      }]
     
* `tx_dst_bals_inc`.  This is an array of [Amount](#amount_object) objects representing all the balance changes caused by this payment to the destination account.  For example:

      "tx_dst_bals_inc": [{
          "value": "100",
          "currency": "XRP",
          "issuer": ""
      }]

#### [Notifications](id:notification_object) ####

More to come...

## Addresses ##

More to come...

## Trustlines ##

More to come...

## Outgoing Payments ##

### Preparing a Payment ###

### Submitting a Payment ###

### Confirming a Payment ###

## Incoming Payments ##

## Generic Ripple Transactions ##

## Server Status ##

## Examples ##

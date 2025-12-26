---
title: BTSE API Documentation
language_tabs:
  - json
toc_footers: []
includes: []
search: true
highlight_theme: darkula
code_clipboard: true
headingLevel: 2

---

# Change Log

## Version 1.3 (11th Nov 2025)

* Remove [`quote`](#quote-stream) websocket topic from this section. For more details about this websocket topic, please refer to `WebSocket Streams` under `OTC` session. 

## Version 1.2 (17th May 2023)

* Add [`Ping/Pong`](#ping-pong) for websocket streams

## Version 1.1 (16th March 2022)

* Addition of request parameter [`side`](#quote-stream) to allow return one side quote.


## Version 1.0 (19th November 2021)

* Addition of [`quote`](#quote-stream) websocket topic to subscribe to price streams on the OTC market


# Overview

## Generating API Key

You will need to create an API key on the BTSE platform before you can use authenticated APIs. To create API keys, you can follow the steps below:

* Login with your username / email and password into the BTSE website
* Click on “Account” on the top right hand corner
* Select the API tab
* Click on “New API” button to create an API key and passphrase. (Note: the passphrase will only appear once)
* Use your API key and passphrase to construct a signature.

## Endpoints

### Streaming OTC quote

* Production
  * Websocket
     * `wss://ws.btse.com/ws/otc`
* Testnet
  * Websocket
     * `wss://testws.btse.io/ws/otc`


## Authentication

* API Key (request-api)
  * Parameter Name: `request-api`, in: header. API key is obtained from BTSE platform as a string

* API Key (request-nonce)
  * Parameter Name: `request-nonce`, in: header. Representation of current timestamp in long format

* API Key (request-sign)
  * Parameter Name: `request-sign`, in: header. A composite signature produced based on the following algorithm: Signature=HMAC.Sha384 (secretkey, (urlpath + request-nonce + bodyStr)) (note: bodyStr = '' when no data):

# Workflow

## Streaming OTC

* Fetch market info via `Market Summary` OTC api if needed.
* Please refer to `OTC` section to subscribe `Quote Stream` to get streaming otc quote along with quote ids periodically.
* Please refer to `OTC` section for API to accept the quote
  - If users chooses to accept the quote, quote is sent to BTSE (Quote Accepted)
  - If quote is accepted by BTSE, then transaction is completed (Transaction Completed)
  - If quote rejected by BTSE, then BTSE will respond with an updated quote with reason of the rejection

# Websocket Streams

## Ping/Pong
For all our WebSocket servers, simply send a 'ping' message, and the WebSocket server will respond with a 'pong' message if the WebSocket connection is established and active.
> Request

```
ping
```

> Response

```
pong
```

## Authentication

> Request

```json
{
  "op":"authKeyExpires",
  "args":["APIKey", "nonce", "signature"]
}
```

Authenticate the websocket session to subscribe to authenticated websocket topics. Assume we have values as follows:

* `request-nonce`: 1624985375123
* `request-api`: 4e9536c79f0fdd72bf04f2430982d3f61d9d76c996f0175bbba470d69d59816x
* `secret`: 848db84ac252b6726e5f6e7a711d9c96d9fd77d020151b45839a5b59c37203bx

Our subscription request will be:

```
{
  "op":"authKeyExpires",
  "args":["4e9536c79f0fdd72bf04f2430982d3f61d9d76c996f0175bbba470d69d59816x", "1624985375123", "c410d38c681579adb335885800cff24c66171b7cc8376cfe43da1408c581748156b89bcc5a115bb496413bda481139fb"]
}
```

### Request Parameters

Below details the arguments needed to be sent in.

| Index | Type   | Required | Description                          |
| ---   | ---    | ---      | ---                                  |
| 0     | string | Yes      | First argument is the API key        |
| 1     | long   | Yes      | Nonce which is the current timestamp |
| 2     | string | Yes      | Generated signature                  |

> Generating a signature

```shell
echo -n "/ws/otc1624985375123"  | openssl dgst -sha384 -hmac "848db84ac252b6726e5f6e7a711d9c96d9fd77d020151b45839a5b59c37203bx"
(stdin)= c410d38c681579adb335885800cff24c66171b7cc8376cfe43da1408c581748156b89bcc5a115bb496413bda481139fb
```

## Quote Stream

For more details about this websocket topic, please refer to `WebSocket Streams` under `OTC` session.

</section>

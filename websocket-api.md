**Thanks bitstamp**

# Public WebSocket API for string.exchange (2018-10-25)


## What is WebSocket?

WebSocket is a protocol providing full-duplex communications channels over a single TCP connection. The WebSocket protocol was standardized by the IETF as RFC 6455 in 2011, and the WebSocket API in Web IDL is being standardized by the W3C.

## How to connect?

String Exchange uses [Pusher](https://pusher.com/) for real time websocket streaming. Please refer to the [Pusher documentation](https://pusher.com/docs) and libraries to connect to our websocket stream. We provide two types of channels for the clients to subscribe.

* Public channel
* Private channel

### General Pusher Information

* Pusher APP_KEY: 16e8920e945846c1be20
* Pusher CLUSTER: ap1

### Public channel subscription

One example with *javascript* code sample is shown below

```javascript
// new a pusher instance
var pusher = new Pusher('APP_KEY', { cluster: 'CLUSTER' });
// subscribe the channel
var channel = pusher.subscribe('channelName');
// bind the event
channel.bind('eventName', callback);
```

### Private channel subscription

The subscription to private channels is processed in two steps.

* Get one-time api nonce fro authentication through the api call
* Connect to the private channel

One example with *javascript* code sample is shown below

### One-time API nonce creation by api

#### How to sign a request payload

  * BASE_ENDPOINT: https://api.string.exchange
  * Get your API_KEY and SECRET_KEY from the UI api key page
  * Refer to the document about [the request creation with the signature ](https://github.com/blockchaintech-au/string-exchange-api-docs/blob/master/rest-api.md#endpoint-security-type)
  * If you request for multiple times in 30 seconds, it returns 429 with error.

#### Nonce creation endpoint

```javascript
POST /api/v1/nonce
```

**Parameters:**

Name | Type | Mandatory | Example
------------ | ------------ | ------------ | ------------
timestamp | LONG | YES | 1530077353000 |
signature | STRING | YES | c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71 |

**Response:**

Success response
```javascript
{
  "apiNonce": "diGRIKUg5q8VkuMlUyQflGoGlqKHRR5b31vHjMMF",
  "userUuid": "41d423ad-a0aa-4a2a-997e-12995a6b8fae"
}
```

Failure responses
```javascript
# request too frequently
{
  "errors": [
    {
      "type": "REQUEST_TOO_FREQUENT",
      "message": "please request after 30 seconds"
    }
  ]
}
```

### Connect to the private channel

One example with *javascript* code sample is shown below. You can also refer to the [Pusher authentication](https://pusher.com/docs/authenticating_users)

```javascript
// new a pusher instance
var PUSHER_AUTH_ENDPOINT = "${BASE_ENDPOINT}/api/v1/pusher/authentication"
var pusher = new Pusher(APP_KEY, {
  cluster: CLUSTER,
  authEndpoint: PUSHER_AUTH_ENDPOINT,
  auth: {
    params: {
      apiNonce: API_NONCE, // from the response of nonce creation endpoint
      accessKey: API_KEY
    }
  }
});
// subscribe the channel
var channel = pusher.subscribe('channelName');
// bind the event
channel.bind('eventName', callback);
```

## Live Stream

### Private Stream

#### My Trades

**Request:**

Name  | Example | description |
------------ | ------------ | ------------ 
userUuid | 6e5d55b3-ad1d-43a3-ac5f-6e6935bf81d9 | from the response of nonce creation endpoint
channelName | private-user@{:userUuid}.trade | -
event | new | -

**Response:**

```javascript
{
  "uuid": "1f5d45b3-ad1d-43a3-ac5f-6e6935df81d9", 
  "updatedAt": "2018-09-05T0349:34.587Z", 
  "price": "26000.34", 
  "amount": "1.54", 
  "symbol": "BTC/AUD",
  "maker": true/false,
  "side": "BID/ASK"
}
```

### Public Stream

#### DepthBook

**Request:**

Name  | Example | description
precisionScale | 6 | BTC market is 6, AUD market is 2
------------ | ------------ | ------------ 
channelName | ETH/BTC@depth.6 | -
event | new | -

**Response:**

```javascript
"depthBook": {
  "bids": [
    {
      "price": "1.00",
      "amount": "1.23"
    },
    ...
  ],
  "asks": [
    {
      "price": "1.00",
      "amount": "1.23"
    },
    ...
  ]
}
```

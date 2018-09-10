**Thanks bitstamp**

# Public WebSocket API for string.exchange (2018-09-10)


## What is WebSocket?

WebSocket is a protocol providing full-duplex communications channels over a single TCP connection. The WebSocket protocol was standardized by the IETF as RFC 6455 in 2011, and the WebSocket API in Web IDL is being standardized by the W3C.

## How to connect?

String Exchange uses [Pusher](https://pusher.com/) for real time websocket streaming. Please refer to the [Pusher documentation](https://pusher.com/docs) and libraries to connect to our websocket stream. You can also find an example for each stream below.

For the subscription to private channels, authentication is mandatory. One example with *javascript* code sample is shown below

### One-time API nonce creation

```javascript
POST /api/vi/nonce
```

Note:
* This API requires ID verification first, otherwise it returns 400 with error. For more detail, please refer to the [public restful api documentation](https://github.com/blockchaintech-au/cex-api-docs/blob/master/rest-api.md)
* If you request for multiple times in 30 seconds, it returns 429 with error.

**Parameters:**

Name | Type | Mandatory | Example
------------ | ------------ | ------------ | ------------
timestamp | LONG | YES | 1530077353000 |

**Response:**

Success response
```javascript
{
  "apiNonce": "diGRIKUg5q8VkuMlUyQflGoGlqKHRR5b31vHjMMF",
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

### Pusher Authentication

Note:
* Please refer to the [Pusher authentication](https://pusher.com/docs/authenticating_users)
* Pusher app_key: e5f57e41f4928bf98a7f

```javascript
auth endpoint: /api/v1/pusher/authentication
```

**Parameters:**

Name | Type | Mandatory | Example
------------ | ------------ | ------------ | ------------
apiNonce | STRING | YES | diGRIKUg5q8VkuMlUyQflGoGlqKHRR5b31vHjMMF |
accessKey | STRING | YES | diGRIKUg5q8VkuMlUyQflGoGlqKHRR5b31vHjMMF |

**Example:**

```javascript
<script>
  var pusher = new Pusher('app_key', {
    authEndpoint: '/api/v1/pusher/authentication',
    auth: {
      params: {
        apiNonce,
        accessKey,
      }
    }
  });
</script>
```

## Live Stream

### My Trades (Private)

**Request:**

Name  | Example
------------ | ------------
userUuid | 6e5d55b3-ad1d-43a3-ac5f-6e6935bf81d9 |
channelName | private-user@{:userUuid}.trade |
event | new |

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

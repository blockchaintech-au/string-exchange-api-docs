**Thanks binance**

# Public Rest API for String.exchange
# General API Information
* The base endpoint is: **https://api.string.exchange**
* All endpoints return either a JSON object or array.
* Data is returned in **ascending** order. Oldest first, newest last.
* All time and timestamp related fields are in milliseconds.
* HTTP `400` return code are used for for malformed requests;
  the issue is on the sender's side.
* HTTP `401` return code are used for API key and signature error.\
* HTTP `500` return code is used for server error
* Any endpoint can retun an ERROR; the error payload is as follows:
```javascript
{
  "errors": [
    {
      "type": "INSUFFICIENT_FUND",
      "message": "Not enough fund"
    }
  ]
}
```

* Specific error codes and messages defined in another document.
* For `GET` endpoints, parameters must be sent as a `query string`.
* For `POST`, `PUT`, and `DELETE` endpoints, the parameters may be sent as a
  `query string` or in the `request body` with content type
  `application/x-www-form-urlencoded`. You may mix parameters between both the
  `query string` and `request body` if you wish to do so.
* Parameters may be sent in any order.


# Endpoint security type
* API-keys are passed into the Rest API via the `X-CEX-APIKEY`
  header.
* API-keys and secret-keys **are case sensitive**.
* API-keys can access all secure routes.

# SIGNED (TRADE and USER_DATA) Endpoint security
* `SIGNED` endpoints require an additional parameter, `signature`, to be
  sent in the  `query string` or `request body`.
* Endpoints use `HMAC SHA256` signatures. The `HMAC SHA256 signature` is a keyed `HMAC SHA256` operation.
  Use your `secretKey` as the key and `totalParams` as the value for the HMAC operation.
* The `signature` is **not case sensitive**.
* `totalParams` is defined as the `query string` concatenated with the
  `request body`.

## Timing security
* A `SIGNED` endpoint also requires a parameter, `timestamp`, to be sent which
  should be the millisecond timestamp of when the request was created and sent.

## SIGNED Endpoint Examples for POST /api/v1/order
Here is a step-by-step example of how to send a vaild signed payload from the
Linux command line using `echo`, `openssl`, and `curl`.

Key | Value
------------ | ------------
apiKey | vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A
secretKey | NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j


Parameter | Value
------------ | ------------
symbol | ETH/BTC
side | ASK
type | LIMIT
amount | 1
price | 0.1
timestamp | 1499827319559


### Example 1: As a query string
* **queryString:** symbol=ETH%2FBTC&side=ASK&type=LIMIT&amount=1&price=0.1&timestamp=1499827319559
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "symbol=ETH%2FBTC&side=ASK&type=LIMIT&amount=1&price=0.1&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-CEX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://api.cex24.io/api/v1/order?symbol=ETH%2FBTC&side=ASK&type=LIMIT&amount=1&price=0.1&timestamp=1499827319559&signature=c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71'
    ```

### Example 2: As a request body
* **requestBody:** symbol=ETH/BTC&side=ASK&type=LIMIT&amount=1&price=0.1&timestamp=1499827319559
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "symbol=ETH/BTC&side=ASK&type=LIMIT&amount=1&price=0.1&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-CEX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://api.cex24.io/api/v1/order' -d 'symbol=ETH/BTC&side=ASK&type=LIMIT&amount=1&price=0.1&timestamp=1499827319559&signature=c8db56825ae71d6d79447849e617115f4a920fa2acdcab2b053c4b2838bd6b71'
    ```

### Example 3: Mixed query string and request body
* **queryString:** symbol=ETH%2FBTC&side=ASK&type=LIMIT
* **requestBody:** amount=1&price=0.1&timestamp=1499827319559
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "symbol=ETH%2FBTC&side=ASK&type=LIMITamount=1&price=0.1&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= 0fd168b8ddb4876a0358a8d14d0c9f3da0e9b20c5d52b2a00fcf7d1c602f9a77
    ```


* **curl command:**

    ```
    (HMAC SHA256)
    [linux]$ curl -H "X-CEX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://api.binance.com/api/v1/order?symbol=ETH%2FBTC&side=ASK&type=LIMIT' -d 'amount=1&price=0.1&timestamp=1499827319559&signature=0fd168b8ddb4876a0358a8d14d0c9f3da0e9b20c5d52b2a00fcf7d1c602f9a77'
    ```

Note that the signature is different in example 3.
There is no & between "LIMIT" and "amount=1".

# Public API Endpoints

## Market Data endpoints
### Order book
```
GET /api/v1/depth
```

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES | e.g. ETH/BTC
limit | INT | NO | Default 100; If limit is 10, you will get 10 bids and 10 asks.

**Caution:** setting a big limit could cause request timeout

**Response:**
```javascript
{
  "bids": [
    {
      "price": "4.00000200",
      "amount": "12.00000000"
    }
  ],
  "asks": [
    {
      "price": "4.00000200",
      "amount": "12.00000000"
    }
  ]
}
```

### 24hr ticker price change statistics
```
GET /api/v1/ticker/24hr
```
24 hour price change statistics.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |

* If the symbol is not sent, tickers for all symbols will be returned in an array.
* When all symbols are returned, the number of requests counted against the rate limiter is equal to the number of symbols currently trading on the exchange.

**Response:**
```javascript
[
  {
    "symbol": "BTC/AUD",
    "close": '0.0923',
    "high": '0.0761',
    "low": '0.0931',
    "open": '0.0292',
    "volume": '0.0413',
    "change": '4.35',
    "askPrice": '10000.00',
    "bidPrice": '9999.00',
    "timestamp": "2018-10-24T03:10:12.142Z"
  }
]
```


## Account endpoints
### Create order  (TRADE)
```
POST /api/v1/order  (HMAC SHA256)
```
Send in a new order.

Note that this API requires ID verification first, otherwise it returns 400 with error.

**Parameters:**

Name | Type | Mandatory | Example
------------ | ------------ | ------------ | ------------
orderUuid | STRING | YES | 446c0621-ceb8-4cbb-a224-cc2ae80a134b |
timestamp | LONG | YES | 1530077353000 |

Single order total value ( amount * price ) should be over minimum trading total described below:

Symbol | Minimum order total
------------ | ------------
ETH/BTC | 0.001
BTC/AUD | 10
ETH/AUD | 10

Additional mandatory parameters based on `type`:

Type | Additional mandatory parameters
------------ | ------------
`LIMIT` | `amount`, `price`


**Response:**

Success response
```javascript
{
    "uuid": "50f78c1d-a61c-4cd7-baf4-5eeef27852a5",
    "type": "LIMIT",
    "side": "ASK",
    "price": "1.3",
    "amount": "1.3",
    "filled": "0.0",
    "status": "OPEN",
    "createdAt": "2018-10-24T06:20:20.012Z",
    "updatedAt": "2018-10-24T06:20:20.202Z",
    "symbol": "ETH/BTC"
}
```

Failure responses
```javascript
# below minimum order total
{
    "errors": [
        {
            "type": "BELOW_MIN_ORDER_TOTAL",
            "message": "Order total less than 0.001"
        }
    ]
}

# ID not verified
{
    "errors": [
        {
            "type": "ID_NOT_VERIFIED",
            "message": "KYC is required"
        }
    ]
}
```


### Query order (USER_DATA)
```
GET /api/v1/order (HMAC SHA256)
```
Check an order's status.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
orderUuid | LONG | YES |
timestamp | LONG | YES |


**Response:**
```javascript
{
  "symbol": "BTC/AUD",
  "uuid": "446c0621-ceb8-4cbb-a224-cc2ae80a134b",
  "price": "0.1",
  "amouont": "1.0",
  "filled": "0.0",
  "status": "OPEN",
  "type": "LIMIT",
  "side": "ASK",
  "createdAt": "2018-08-16T04:59:49.336Z",
  "updatedAt": "2018-08-16T04:59:49.336Z",
}
```
**Response Fields:**

Name | Type | Values | Description
------------ | ------------ | ------------ | ------------
status | String | OPEN, CANCEL, PARTIAL_FILLED, FILLED  |

### Cancel order (TRADE)
```
DELETE /api/v1/order  (HMAC SHA256)
```
Cancel an active order.

**Weight:**
1

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
orderUuid | String | YES |
timestamp | LONG | YES |

**Response:**
Status: 202
Body: empty

### Current open orders (USER_DATA)
```
GET /api/v1/openOrders  (HMAC SHA256)
```
Get all open orders.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO | If no symbol passed, return all open orders
limit | INT | NO | default 500, max 1000 
timestamp | LONG | YES |

**Response:**
```javascript
[
    {
        "uuid": "50f78c1d-a61c-4cd7-baf4-5eeef27852a5",
        "price": "1.3",
        "createdAt": "2018-10-24T06:20:20.012Z",
        "updatedAt": "2018-10-24T06:20:20.202Z",
        "amount": "1.3",
        "filled": "0.0",
        "type": "LIMIT",
        "side": "ASK",
        "status": "OPEN",
        "symbol": "ETH/BTC"
    }
]
```

### Account information (USER_DATA)
```
GET /api/v1/account (HMAC SHA256)
```
Get current account information.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
timestamp | LONG | YES |

**Response:**
```javascript
[
    {
        "uuid": "35f354d2-06d6-45a6-aec2-1f1b082fdced",
        "holderUuid": "627dffa7-4433-4fe0-bdc7-27adf8e75585",
        "depositAddress": "0xef87011b6404e453bdf2212fe5bdb7c198f12503",
        "totalBalance": "195.0843",
        "availableBalance": "192.4843",
        "inOrder": "2.6",
        "coin": {
            "uuid": "18bb0c4b-1b7d-4151-a95c-05994bc89d1c",
            "symbol": "ETH",
            "name": "Ethereum",
            "precisionScale": 8,
            "withdrawFee": "0.005",
            "minimumWithdraw": "0.01",
            "expectedConfirmations": 10
        }
    }
]
```

### Account trade list (USER_DATA)
```
GET /api/v1/myTrades  (HMAC SHA256)
```
Get trades for a specific account and symbol.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | 
timestamp | LONG | YES |

**Response:**
```javascript
[
  {
    "uuid": "abc32965-3e8a-4884-9f48-7a6e90badc22",
    "createdAt": "2018-10-10T00:11:33.875Z",
    "price": "8530.32",
    "amount": "0.015",
    "isBuyer": false,
    "isMaker": false,
    "tradingPair": {
      "uuid": "993c9939-5fb1-430d-865f-5429c8114072",
      "symbol": "BTC/AUD",
      "base": {
        "uuid": "1161f80d-84b6-4a2e-b4b2-1ae722932021",
        "symbol": "BTC",
        "precisionScale": 6
      },
      "quote": {
        "uuid": "b128a80e-53d9-48f3-beda-e6076d40de24",
        "symbol": "AUD",
        "precisionScale": 2
      },
      "minimumTradingTotal": "10.0"
    }
  }
]
```

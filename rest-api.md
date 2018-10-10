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
* If a parameter sent in both the `query string` and `request body`, the
  `query string` parameter will be used.

# LIMITS
* The `/api/v1/exchangeInfo` `rateLimits` array contains objects related to the exchange's `REQUESTS` and `ORDER` rate limits.
* A 429 will be returned when either rather limit is violated.
* Each route has a `weight` which determines for the number of requests each endpoint counts for. Heavier endpoints and endpoints that do operations on multiple symbols will have a heavier `weight`.
* When a 429 is recieved, it's your obligation as an API to back off and not spam the API.
* **Repeatedly violating rate limits and/or failing to back off after receiving 429s will result in an automated IP ban (http status 418).**
* IP bans are tracked and **scale in duration** for repeat offenders, **from 2 minutes to 3 days**.

# Endpoint security type
* Each endpoint has a security type that determines the how you will
  interact with it.
* API-keys are passed into the Rest API via the `X-CEX-APIKEY`
  header.
* API-keys and secret-keys **are case sensitive**.
* API-keys can be configured to only access certain types of secure endpoints.
 For example, one API-key could be used for TRADE only, while another API-key
 can access everything except for TRADE routes.
* By default, API-keys can access all secure routes.

Security Type | Description
------------ | ------------
NONE | Endpoint can be accessed freely.
TRADE | Endpoint requires sending a valid API-Key and signature.
USER_DATA | Endpoint requires sending a valid API-Key and signature.
USER_STREAM | Endpoint requires sending a valid API-Key.
MARKET_DATA | Endpoint requires sending a valid API-Key.


* `TRADE` and `USER_DATA` endpoints are `SIGNED` endpoints.

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
* An additional parameter, `recvWindow`, may be sent to specific the number of
  milliseconds after `timestamp` the request is valid for. If `recvWindow`
  is not sent, **it defaults to 5000**.
* The logic is as follows:
  ```javascript
  if (timestamp < (serverTime + 1000) && (serverTime - timestamp) <= recvWindow) {
    // process request
  } else {
    // reject request
  }
  ```

**Serious trading is about timing.** Networks can be unstable and unreliable,
which can lead to requests taking varying amounts of time to reach the
servers. With `recvWindow`, you can specify that the request must be
processed within a certain number of milliseconds or be rejected by the
server.


**Tt recommended to use a small recvWindow of 5000 or less!**


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
There is no & between "GTC" and "quantity=1".

# Public API Endpoints
## Terminology
* `base asset` refers to the asset that is the `quantity` of a symbol.
* `quoate asset` refers to the asset that is the `price` of a symbol.


## ENUM definitions
**Symbol status:**

* PRE_TRADING
* TRADING
* POST_TRADING
* END_OF_DAY
* HALT
* AUCTION_MATCH
* BREAK

**Symbol type:**

* SPOT

**Order status:**

* NEW
* PARTIALLY_FILLED
* FILLED
* CANCELED
* PENDING_CANCEL (currently unused)
* REJECTED
* EXPIRED

**Order types:**

* LIMIT
* MARKET
* STOP_LOSS
* STOP_LOSS_LIMIT
* TAKE_PROFIT
* TAKE_PROFIT_LIMIT
* LIMIT_MAKER

**Order side:**

* BUY
* SELL

**Time in force:**

* GTC
* IOC
* FOK

**Kline/Candlestick chart intervals:**

m -> minutes; h -> hours; d -> days; w -> weeks; M -> months

* 1m
* 3m
* 5m
* 15m
* 30m
* 1h
* 2h
* 4h
* 6h
* 8h
* 12h
* 1d
* 3d
* 1w
* 1M

**Rate limiters (rateLimitType)**

* REQUESTS
* ORDERS

**Rate limit intervals**

* SECOND
* MINUTE
* DAY


## Market Data endpoints
### Order book
```
GET /api/v1/depth
```

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 100; If limit is 10, you will get 10 bids and 10 asks.

**Caution:** setting a big limit could cause request timeout

**Response:**
```javascript
{
  "bids": [
    [
      "4.00000000",     // PRICE
      "431.00000000"   // QTY
    ]
  ],
  "asks": [
    [
      "4.00000200",
      "12.00000000"
    ]
  ]
}
```

### 24hr ticker price change statistics
```
GET /api/v1/ticker/24hr
```
24 hour price change statistics. **Careful** when accessing this with no symbol.

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
    "bidPrice": '9999.00'
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

**Weight:**
1

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
  "orderUuid": "a7b1f89a-660e-4c9c-8dc6-489860c4e82e",
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
orderId | LONG | YES |
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
limit | INT | NO | default 100, max 1000 
timestamp | LONG | YES |

**Response:**
```javascript
[
  {
    "uuid": "",
    "price": "0.1",
    "amount": "1.0",
    "filled": "0.0",
    "status": "Partial Filled",
    "type": "LIMIT",
    "side": "BUY",
    "createdAt": "2018-06-27 16:47:55 +1000",
    "tradingPair"{
      "symbol" : "ETH/BTC"
    }
  }
]
```

### All orders (USER_DATA)
```
GET /api/v1/allOrders (HMAC SHA256)
```
Get all account orders; active, canceled, or filled.

**Weight:**
5

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
orderId | LONG | NO |
limit | INT | NO | Default 1000; max 1000.
recvWindow | LONG | NO |
timestamp | LONG | YES |

* If `orderId` is set, it will get orders >= that `orderId`.
Otherwise most recent orders are returned.

**Response:**
```javascript
[
  {
    "symbol": "LTCBTC",
    "orderId": 1,
    "clientOrderId": "myOrder1",
    "price": "0.1",
    "origQty": "1.0",
    "executedQty": "0.0",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "BUY",
    "stopPrice": "0.0",
    "icebergQty": "0.0",
    "time": 1499827319559,
    "isWorking": true
  }
]
```

### Account information (USER_DATA)
```
GET /api/v1/account (HMAC SHA256)
```
Get current account information.

**Weight:**
5

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
recvWindow | LONG | NO |
timestamp | LONG | YES |

**Response:**
```javascript
{
  "makerCommission": 15,
  "takerCommission": 15,
  "buyerCommission": 0,
  "sellerCommission": 0,
  "canTrade": true,
  "canWithdraw": true,
  "canDeposit": true,
  "updateTime": 123456789,
  "balances": [
    {
      "asset": "BTC",
      "free": "4723846.89208129",
      "locked": "0.00000000"
    },
    {
      "asset": "LTC",
      "free": "4763368.68006011",
      "locked": "0.00000000"
    }
  ]
}
```

### Account trade list (USER_DATA)
```
GET /api/v1/myTrades  (HMAC SHA256)
```
Get trades for a specific account and symbol.

**Weight:**
5

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | YES |
limit | INT | NO | Default 500; max 500.
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

# Filters
Filters define trading rules on a symbol or an exchange.
Filters come in two forms: `symbol filters` and `exchange filters`.

## Symbol filters
### PRICE_FILTER
The `PRICE_FILTER` defines the `price` rules for a symbol. There are 3 parts:

* `minPrice` defines the minimum `price`/`stopPrice` allowed.
* `maxPrice` defines the maximum `price`/`stopPrice` allowed.
* `tickSize` defines the intervals that a `price`/`stopPrice` can be increased/decreased by.

In order to pass the `price filter`, the following must be true for `price`/`stopPrice`:

* `price` >= `minPrice`
* `price` <= `maxPrice`
* (`price`-`minPrice`) % `tickSize` == 0

**/exchangeInfo format:**
```javascript
  {
    "filterType": "PRICE_FILTER",
    "minPrice": "0.00000100",
    "maxPrice": "100000.00000000",
    "tickSize": "0.00000100"
  }
```

### LOT_SIZE
The `LOT_SIZE` filter defines the `quantity` (aka "lots" in auction terms) rules for a symbol. There are 3 parts:

* `minQty` defines the minimum `quantity`/`icebergQty` allowed.
* `maxQty` defines the maximum `quantity`/`icebergQty` allowed.
* `stepSize` defines the intervals that a `quantity`/`icebergQty` can be increased/decreased by.

In order to pass the `lot size`, the following must be true for `quantity`/`icebergQty`:

* `quantity` >= `minQty`
* `quantity` <= `maxQty`
* (`quantity`-`minQty`) % `stepSize` == 0

**/exchangeInfo format:**
```javascript
  {
    "filterType": "LOT_SIZE",
    "minQty": "0.00100000",
    "maxQty": "100000.00000000",
    "stepSize": "0.00100000"
  }
```

### MIN_NOTIONAL
The `MIN_NOTIONAL` filter defines the minimum notional value allowed for an order on a symbol.
An order's notional value is the `price` * `quantity`.

**/exchangeInfo format:**
```javascript
  {
    "filterType": "MIN_NOTIONAL",
    "minNotional": "0.00100000"
  }
```

### MAX_NUM_ORDERS
The `MAX_NUM_ORDERS` filter defines the maximum number of orders an account is allowed to have open on a symbol.
Note that both "algo" orders and normal orders are counted for this filter.

**/exchangeInfo format:**
```javascript
  {
    "filterType": "MAX_NUM_ORDERS",
    "limit": 25
  }
```

### MAX_ALGO_ORDERS
The `MAX_ALGO_ORDERS` filter defines the maximum number of "algo" orders an account is allowed to have open on a symbol.
"Algo" orders are `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, and `TAKE_PROFIT_LIMIT` orders.

**/exchangeInfo format:**
```javascript
  {
    "filterType": "MAX_ALGO_ORDERS",
    "limit": 5
  }
```

## Exchange Filters
### EXCHANGE_MAX_NUM_ORDERS
The `MAX_NUM_ORDERS` filter defines the maximum number of orders an account is allowed to have open on the exchange.
Note that both "algo" orders and normal orders are counted for this filter.

**/exchangeInfo format:**
```javascript
  {
    "filterType": "EXCHANGE_MAX_NUM_ORDERS",
    "limit": 1000
  }
```

### EXCHANGE_MAX_ALGO_ORDERS
The `MAX_ALGO_ORDERS` filter defines the maximum number of "algo" orders an account is allowed to have open on the exchange.
"Algo" orders are `STOP_LOSS`, `STOP_LOSS_LIMIT`, `TAKE_PROFIT`, and `TAKE_PROFIT_LIMIT` orders.

**/exchangeInfo format:**
```javascript
  {
    "filterType": "EXCHANGE_MAX_ALGO_ORDERS",
    "limit": 200
  }
```

# API Libraries
[string-exchange-php](https://github.com/blockchaintech-au/string-exchange-php-sdk)

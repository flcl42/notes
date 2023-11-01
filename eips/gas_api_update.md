# Blob gas price prediction and history methods update

## Motivation 
Execution client provides gas price prediction via eth_getPrice and fee history via xxx. Such services allow to create a transaction with real value for gas fee cap and tip, so it will be executed in adequate time. Shard blob transactipns EIP introduces another kind of gas - blob gas, so users have to pay for it in addition to regular gas fees. It may be usefull to have ability to retrieve potential blob gas price for the same purpose as it is for regular gas. 

Before adding another endpoints or upgrading existing ones let's enumerate cons that woulkd be cool to address.

1. Blob gas price calculation is trivial: to predict blob gas fee for the next blob we just need the values of the current block's header, specifically excessdata gas and blob gas used;
   
   The same is true for the regular gas, the oracle may also be based on non-trivial implementation that produces more efficient prediction, taking into account the transaction may be added to some block far ahead due to price hike.
   
   
3. Is there a real need? Blob transactions will be mostly sent by projects like L2s, not by end users. Execution client's oracle can be not so usefull for such projects.
   
   - Still the update specified below is at least providing standartisation for oracle API that will consider new type of gas. 
   - Making sending blobs more convinient increases neutraility of the blobs update, it's hard to let caviets stay if we know about them.
   - Make blob senders who do it from browsers happier


## New method `eth_getPrices`


The method is similar to `eth_getPrice`, but with prices for multiple dimensions.

Request

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "method": "eth_getPrices"
}
```

Response

```json
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": {
    "gasPrice" : "0x07",
    "blobGasPrice" : "0x07"
  }
}
```

<details>
  <summary>Some funny alternatives</summary>

1. Merged with priority fee

   Request   
   ```json
   {
     "id": 1,
     "jsonrpc": "2.0",
     "method": "eth_getPrices"
   }
   ```   
   Response   
   ```json
   {
     "id": 1,
     "jsonrpc": "2.0",
     "result": {
       "gasPrice" : "0x07",
       "blobGasPrice" : "0x01",
       "priorityGasFee" : "0x02"
     }
   }
   ```

2. Respond with an array

   Request   
   ```json
   {
     "id": 1,
     "jsonrpc": "2.0",
     "method": "eth_getPrices",
   }
   ```   
   Response   
   ```json
   {
     "id": 1,
     "jsonrpc": "2.0",
     "result": ["0x07", "0x01"]
   }
   ```

3. REST style

   Request

   ```
   GET /eth/prices
   ```

   Response

   ```json
   200 OK

   {
     "gasPrice" : "0x07",
     "blobGasPrice" : "0x07",
   }
</details>

## Update for `eth_feeHistory`

Let's add `baseFeePerBlobGas`/`blobGasUsedRatio`, that is `null` pre-Cancun and has `null` values for pre-Cancun blocks

Request

```json
{
    "jsonrpc":"2.0",
    "method":"eth_feeHistory",
    "params":[2, "latest", [25, 75]],
    "id":1
}
```

Response

```json
{
  "id": "1",
  "jsonrpc": "2.0",
  "result": {
    "oldestBlock": 10762137,
    "reward": [
      [
        "0x4a817c7ee",
        "0x4a817c7ee"
      ], [
        "0x773593f0",
        "0x773593f5"
      ], [
        "0x0",
        "0x0"
      ], [
        "0x773593f5",
        "0x773bae75"
      ]
    ],
    "baseFeePerGas": [
      "0x10",
      "0x12",
      "0x10",
    ],
    "baseFeePerBlobGas": [
      "0x01",
      "0x02",
      "0x01",
    ],
    "gasUsedRatio": [
      0.026089875,
      0.406803,
      0,
      0.0866665
    ],
    "blobGasUsedRatio": [
      0.5,
      0.33333,
      0,
      0.0866665
    ]
  }
}
```

Alternatives:
- make a seprate endpoint.
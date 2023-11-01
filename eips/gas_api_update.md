# Blob gas price prediction and history methods update

Stage: Idea

## TLDR;

- Add `eth_getPrices` that returns prices for regular and blob gas
- Update `eth_feeHistory` to include blob related data

## Motivation

Execution client provides gas price prediction via `eth_getPrice` and fee history via `eth_feeHistory`. Such information allows to create a transaction with real value for gas fee cap and tip, so it will be executed in adequate time for adequate price. Shard blob transactions EIP introduces another kind of gas - blob's, so users have to pay for it in addition to regular gas fees when posting blobs. It may be useful to have ability to retrieve potential blob gas price for the same purpose as it is for regular gas. 

Before adding another endpoints or upgrading existing ones let's enumerate cons that would be cool to address.

1. Blob gas price calculation is trivial: to predict blob gas fee for the next blob we just need the values of the current block's header, specifically excess data gas and blob gas used;

   The same is true for the regular gas, the oracle may also be based on non-trivial implementation that produces more efficient prediction, taking into account the transaction may be added to some block far ahead due to price hike.

3. Is there a real need? Blob transactions will be mostly sent by projects like L2s, not by end users. Execution client's oracle can be not so useful for such projects.

   - Making sending blobs more convenient increases neutrality of the blobs update, it's hard to let obstacles stay if we know about them;
   - Make blob senders who do it from browsers happier;
   - The update specified below is at least defining standard for oracle API that will consider new type of gas.


## New method `eth_getPrices`

The method is similar to `eth_getPrice`, but with prices for multiple gas dimensions.

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
    "gas" : "0x07",
    "blobGas" : "0x07"
  }
}
```


Alternatives:

- make a separate endpoint for blob gas;

- <details>
  <summary>some funny alternatives</summary>

  ### 1. Merged with `eth_maxPriorityFeePerGas`

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
        "maxPriorityFee" : "0x02"
      }
    }
    ```

  ### 2. Respond with an array

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

  ### 3. REST style

    Request

    ```
    GET /v1/eth/prices
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

Let's add `baseFeePerBlobGas`/`blobGasUsedRatio`, that has `0` values for pre-Cancun blocks.

Request

```json
{
    "jsonrpc":"2.0",
    "id":1,
    "method":"eth_feeHistory",
    "params":[2, "latest", [25, 75]]
}
```

Response

```json
{
  "id": "1",
  "jsonrpc": "2.0",
  "result": {
    "oldestBlock": 42, // 0x2a?
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
      0.0,
      0.5,
      0.25,
      0.25
    ],
    "blobGasUsedRatio": [
      0.0,
      0.5,
      0.25,
      0.25
    ]
  }
}
```

Alternatives:

- make a separate endpoint.


## Prototype

Cooking

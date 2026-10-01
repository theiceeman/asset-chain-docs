# Bridge API

HTTP API for listing bridge configuration, reading token balances, syncing a wallet’s bridge transfers, recording a send, marking a claim, and requesting relayer signatures.

All routes below are relative to the bridge base URL. Requests and responses use JSON unless noted.

These routes require the [API guard](#api-guard) headers.

---

## API guard

Every documented route is signed. The bridge rejects the request before the handler runs when the signature is missing, stale, or wrong.

To obtain a client, contact AssetChain. You will receive a **client key** 

Do not request a client key from the bridge. AssetChain issues it.

### Headers

| Header | Required | Description |
| --- | --- | --- |
| `x-client-name` | yes | Client name issued by AssetChain. |
| `x-timestamp` | yes | Unix time in seconds or milliseconds. Must be within 60 seconds of the server clock. |
| `x-signature` | yes | Hex-encoded HMAC-SHA256 of the canonical string, using the client key as the secret. |
| `Content-Type` | for bodies | `application/json` when the request has a body. |

### Canonical string

Hash the JSON body with SHA-256 (hex). If the request has no body, hash the empty string.

Join these four lines with a newline (`\n`). The path is the request path only, with no query string.

```text
<HTTP method in uppercase>
<request path>
<x-timestamp value exactly as sent>
<sha256 hex of the body>
```

`x-signature` is `HMAC-SHA256(clientKey, canonicalString)` encoded as hex.

Example for `GET /bridges` with timestamp `1710000000` and no body:

```text
GET
/bridges
1710000000
e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
```

The last line is SHA-256 of an empty string. If you send a JSON body, hash that exact JSON string, not a re-serialized object.

Node.js:

```js
const crypto = require("crypto");

function signRequest({ method, path, timestamp, body, clientKey }) {
  const payload = body ? JSON.stringify(body) : "";
  const bodyHash = crypto.createHash("sha256").update(payload).digest("hex");
  const canonical = [method.toUpperCase(), path, String(timestamp), bodyHash].join("\n");
  return crypto.createHmac("sha256", clientKey).update(canonical).digest("hex");
}
```

Browsers and other web apps (TypeScript). `crypto.subtle` is async, so `signRequest` returns a `Promise<string>`:

```ts
function toHex(buffer: ArrayBuffer): string {
  return [...new Uint8Array(buffer)]
    .map((byte) => byte.toString(16).padStart(2, "0"))
    .join("");
}

async function signRequest({
  method,
  path,
  timestamp,
  body,
  clientKey,
}: {
  method: string;
  path: string;
  timestamp: string | number;
  body?: unknown;
  clientKey: string;
}): Promise<string> {
  const encoder = new TextEncoder();
  const payload = body ? JSON.stringify(body) : "";

  const bodyHash = toHex(
    await crypto.subtle.digest("SHA-256", encoder.encode(payload)),
  );
  const canonical = [method.toUpperCase(), path, String(timestamp), bodyHash].join("\n");

  const key = await crypto.subtle.importKey(
    "raw",
    encoder.encode(clientKey),
    { name: "HMAC", hash: "SHA-256" },
    false,
    ["sign"],
  );
  const signature = await crypto.subtle.sign("HMAC", key, encoder.encode(canonical));

  return toHex(signature);
}
```

Use the same `timestamp` string in `x-timestamp` and in the canonical string. `JSON.stringify` the same object you send as the body. Hash that exact string. The client key is the HMAC secret: a browser can read it, so do not ship it in a public web app.

### Guard responses

Missing headers:

```json
{ "message": "Missing signature headers" }
```

`HTTP 401`

Timestamp is not a number:

```json
{ "message": "Invalid timestamp" }
```

`HTTP 401`

Timestamp is older or newer than 60 seconds:

```json
{ "message": "Request expired" }
```

`HTTP 401`

Signature does not match:

```json
{ "message": "Invalid signature" }
```

`HTTP 401`

---

## Errors from route handlers

Validation failures returned by a route use HTTP 400:

```json
{ "error": "Invalid user address" }
```

Thrown errors are passed through the error handler:

```json
{
  "code": "INTERNAL_ERROR",
  "message": "Transaction not found"
}
```

## GET /tokens

Lists configured tokens.

### Parameters

None.

### Response

`HTTP 200`

```json
{
  "success": true,
  "data": [
    {
      "id": "8b1c0c2e-1d2a-4c3b-9f10-0a1b2c3d4e5f",
      "tokenAddress": "0x036CbD53842c5426634e7929541eC2318f3dCF7e",
      "decimal": 6,
      "symbol": "USDC",
      "name": "USD Coin",
      "chainId": "84532",
      "maxClaimAmount": null,
      "createdAt": "2026-01-01T00:00:00.000Z",
      "updatedAt": "2026-01-01T00:00:00.000Z"
    }
  ]
}
```

| Field | Description |
| --- | --- |
| `decimal` | Token decimals. |
| `chainId` | Chain id string, including Solana ids such as `sol.devnet`. |
| `maxClaimAmount` | Raw max claim amount for this token, or `null` when no limit is set. |


## GET /bridges

Lists bridge deployments, including fees and the linked token.

### Parameters

None.

### Response

`HTTP 200`

```json
{
  "success": true,
  "data": [
    {
      "id": "c3d4e5f6-a7b8-4901-8234-567890abcdef",
      "bridgeAddress": "0xEC91dd5f2048DB18C32e15Ca75a59e1e72E5E267",
      "chainId": "421614",
      "fees": {
        "feeFulfill": 0,
        "feeSend": 0
      },
      "tokenId": "8b1c0c2e-1d2a-4c3b-9f10-0a1b2c3d4e5f",
      "limitPerSend": "1000000000",
      "tokenDecimal": 6,
      "createdAt": "2026-01-01T00:00:00.000Z",
      "updatedAt": "2026-01-01T00:00:00.000Z",
      "token": {
        "id": "8b1c0c2e-1d2a-4c3b-9f10-0a1b2c3d4e5f",
        "tokenAddress": "0x036CbD53842c5426634e7929541eC2318f3dCF7e",
        "decimal": 6,
        "symbol": "USDC",
        "name": "USD Coin",
        "chainId": "421614",
        "maxClaimAmount": null,
        "createdAt": "2026-01-01T00:00:00.000Z",
        "updatedAt": "2026-01-01T00:00:00.000Z"
      }
    }
  ]
}
```

| Field | Description |
| --- | --- |
| `fees.feeSend` | Fee taken on send, in protocol units. |
| `fees.feeFulfill` | Fee taken on fulfill, in protocol units. |
| `limitPerSend` | Raw maximum amount per send. |
| `tokenDecimal` | Decimals used for this bridge’s token on this chain. |

---

## GET /transactions

Returns stored bridge transfers for a wallet. On the first request for that wallet, or when `forceSync=true`, the bridge reads transfers from chain and writes them before responding.

### Query parameters

| Parameter | Required | Description |
| --- | --- | --- |
| `userAddress` | yes | EVM or Solana wallet. |
| `secondaryAddress` | no | Second wallet included in the same result. |
| `chainIds` | no | Comma-separated chain ids to filter, for example `84532,421614`. |
| `fulfilled` | no | `true` or `false`. Omit to return both. |
| `forceSync` | no | `true` re-reads transfers from chain before returning the page. |
| `symbol` | no | Token symbol filter. `XRWA` is stored as `RWA`. |
| `page` | no | Page number. Default `1`. Minimum `1`. |
| `limit` | no | Page size. Default `10`. Minimum `1`, maximum `100`. |

### Response

`HTTP 200`

```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "a1b2c3d4-e5f6-4789-a012-3456789abcde",
        "amount": "1000000",
        "timestamp": "1710000000",
        "fromUser": "0x1111111111111111111111111111111111111111",
        "toUser": "0x2222222222222222222222222222222222222222",
        "fromChain": "84532",
        "toChain": "421614",
        "nonce": 3,
        "symbol": "USDC",
        "fulfillAmount": "1000000",
        "fulfillFromChain": "84532",
        "fulfillNonce": 3,
        "fulfillFromUser": "0x1111111111111111111111111111111111111111",
        "fulfillToUser": "0x2222222222222222222222222222222222222222",
        "txBlock": "12345678",
        "confirmations": 12,
        "fulfilled": false,
        "index": 3,
        "userAddress": "0x1111111111111111111111111111111111111111",
        "chainId": "84532",
        "transactionHash": "0xabc",
        "claimTransactionHash": null,
        "transactionDate": "2026-03-01T12:00:00.000Z",
        "claimApproved": false,
        "approvedByAdminId": null,
        "approvedAt": null,
        "bridgeInfoId": "c3d4e5f6-a7b8-4901-8234-567890abcdef",
        "createdAt": "2026-03-01T12:00:05.000Z",
        "updatedAt": "2026-03-01T12:00:05.000Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "totalCount": 1,
      "totalPages": 1,
      "hasNextPage": false,
      "hasPreviousPage": false,
      "nextPage": null,
      "previousPage": null
    }
  }
}
```

`amount`, `fulfillAmount`, `timestamp`, and `txBlock` are strings. `claimApproved` is `true` after an admin approves a transfer that is above the token’s max claim amount.

Invalid query:

```json
{ "error": "Invalid user address" }
```

`HTTP 400`

---

## POST /transactions

Records a transfer that already exists on the source chain. The bridge reads the on-chain send and stores it. If that index is already stored for the same wallet and bridge, the existing row is returned.

### Body

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `bridgeId` | string | yes | Bridge id from `GET /bridges`. |
| `index` | string | yes | Source-chain transaction index for this user. |
| `userAddress` | string | yes | EVM or Solana wallet that sent the transfer. |
| `transactionHash` | string | yes | Source-chain transaction hash. |

### Response

`HTTP 201`

```json
{
  "success": true,
  "data": {
    "success": true,
    "message": "Transaction added successfully",
    "data": {
      "id": "a1b2c3d4-e5f6-4789-a012-3456789abcde",
      "amount": "1000000",
      "timestamp": "1710000000",
      "fromUser": "0x1111111111111111111111111111111111111111",
      "toUser": "0x2222222222222222222222222222222222222222",
      "fromChain": "84532",
      "toChain": "421614",
      "nonce": 3,
      "symbol": "USDC",
      "fulfillAmount": "1000000",
      "fulfillFromChain": "84532",
      "fulfillNonce": 3,
      "fulfillFromUser": "0x1111111111111111111111111111111111111111",
      "fulfillToUser": "0x2222222222222222222222222222222222222222",
      "txBlock": "12345678",
      "confirmations": 12,
      "fulfilled": false,
      "index": 3,
      "userAddress": "0x1111111111111111111111111111111111111111",
      "chainId": "84532",
      "transactionHash": "0xabc",
      "bridgeInfoId": "c3d4e5f6-a7b8-4901-8234-567890abcdef",
      "transactionDate": "2026-03-01T12:00:00.000Z"
    }
  }
}
```

If the row already exists, the HTTP status is still 201 and `data.message` is `"Transaction already exists"`.

Missing fields:

```json
{ "error": "Missing required fields: bridgeId, index, userAddress" }
```

`HTTP 400`

Unknown bridge (`HTTP 400`):

```json
{
  "code": "INTERNAL_ERROR",
  "message": "Bridge not found"
}
```

The on-chain send does not exist (`HTTP 400`):

```json
{
  "code": "INTERNAL_ERROR",
  "message": "Transaction not found"
}
```

---

## PUT /transactions

Checks whether a stored transfer has been fulfilled on the destination chain. When the destination confirms fulfillment, the row is marked fulfilled and the claim hash is saved.

### Body

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `transactionId` | string | yes | Transaction id from `GET /transactions`. |
| `claimTransactionHash` | string | yes | Destination-chain claim transaction hash. |
| `toBridgeId` | string | yes | Destination bridge id from `GET /bridges`. |

### Response

`HTTP 200`

```json
{
  "success": true,
  "data": {
    "success": true,
    "message": "Transaction marked as claimed",
    "data": {
      "id": "a1b2c3d4-e5f6-4789-a012-3456789abcde",
      "fulfilled": true,
      "claimTransactionHash": "0xdef",
      "amount": "1000000",
      "fromChain": "84532",
      "toChain": "421614",
      "index": 3,
      "userAddress": "0x1111111111111111111111111111111111111111",
      "bridgeInfoId": "c3d4e5f6-a7b8-4901-8234-567890abcdef"
    }
  }
}
```

`fulfilled` stays `false` when the destination chain does not yet show the claim. If the row was already fulfilled, `data.message` is `"Transaction already fulfilled"`.

Missing fields:

```json
{
  "error": "Missing required fields: transactionId, claimTransactionHash, toBridgeId"
}
```

`HTTP 400`

Unknown transaction or bridge (`HTTP 400`):

```json
{
  "code": "INTERNAL_ERROR",
  "message": "Transaction not found"
}
```

```json
{
  "code": "INTERNAL_ERROR",
  "message": "Bridge not found"
}
```

---

## GET /blockchain/sign

Returns the relayer signatures required to fulfill a transfer on the destination chain.

### Query parameters

| Parameter | Required | Description |
| --- | --- | --- |
| `bridgeId` | yes | Source bridge id from `GET /bridges`. |
| `index` | yes | Source-chain transaction index for `fromUser`. |
| `fromUser` | yes | EVM or Solana address that sent the transfer. |

### Response

`HTTP 200`

`signature` contains one signature string per destination relayer, in relayer index order.

```json
{
  "signature": [
    "0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "0xbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
  ]
}
```

Validation failure:

```json
{ "error": "Invalid user address" }
```

`HTTP 400`

Common thrown errors (`HTTP 400`, `code` `INTERNAL_ERROR`):

```json
{ "code": "INTERNAL_ERROR", "message": "Transaction not found" }
```

```json
{ "code": "INTERNAL_ERROR", "message": "Not supported" }
```

```json
{ "code": "INTERNAL_ERROR", "message": "Bridge not found" }
```

```json
{ "code": "INTERNAL_ERROR", "message": "Transaction already fulfilled" }
```

```json
{ "code": "INTERNAL_ERROR", "message": "Claim requires admin approval" }
```

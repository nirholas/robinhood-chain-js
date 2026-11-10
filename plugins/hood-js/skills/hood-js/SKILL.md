---
name: hood-js
description: Write JavaScript or TypeScript against Robinhood Chain (chain ID 4663) with the hood-js package: Stock Token prices, multiplier-correct portfolios, trending memecoins, launchpad streams, quotes and swaps. Use when a task needs Robinhood Chain data or swaps from Node or the browser.
---

# hood-js

`hood-js` is a promise-first wrapper over the `hoodchain` SDK. It runs in Node 20+ and in browsers, uses the public RPC by default, and needs no configuration for reads.

## Install

```bash
npm install hood-js viem
```

`viem` (^2.55.0) is a peer dependency. The package default export is the `hood` facade; `HoodError` is a named export.

```js
import hood, { HoodError } from 'hood-js'
```

## API

| Call | Returns |
| --- | --- |
| `hood.price('TSLA')` | `{ symbol, usd, updatedAt }`; `usd` is `null` when the token has no live feed |
| `hood.prices(['AAPL', 'TSLA'])` | array of the same shape, batched |
| `hood.portfolio('0xAddress')` | `{ owner, totalUsd, positions, unpriced }`, multiplier-corrected |
| `hood.coins({ limit, window, timeoutMs })` | trending memecoins; `window` is `'5m'`, `'1h'`, `'6h'` or `'24h'` (default 24h, limit 20) |
| `hood.launches({ launchpad, lookbackBlocks })` | recent launches as an array |
| `hood.launches({ live: true, launchpad, pollingInterval, onError }, cb)` | live stream; returns a stop function |
| `hood.quote({ sell, buy, amount })` | a quote with `sell`, `buy`, `sellAmount`, `buyAmount`, `rate`, `route` |
| `hood.swap({ ...quote, wallet, slippageBps, recipient })` | `{ hash, url, sold, bought, amountOut, minReceived, status }` |
| `hood.config({ rpcUrl, alchemyKey, acknowledgeEligibility, geckoNetwork })` | chainable; sets options |
| `hood.testnet()` / `hood.mainnet()` | chainable; switch to chain 46630 or 4663 |

`launchpad` is `'noxa'` or `'odyssey'` (both by default). `wallet` is a `0x` private key or an EIP-1193 provider.

## Worked example

```js
import hood from 'hood-js'

const { usd } = await hood.price('AAPL')
const { totalUsd, positions } = await hood.portfolio('0x0000000000000000000000000000000000000001')
const trending = await hood.coins({ limit: 5, window: '1h' })

const stop = hood.launches({ live: true }, (l) => console.log(l.launchpad, l.token, l.url))
setTimeout(stop, 60_000)
```

## Swaps

`hood.swap` sends a real transaction and approvals are handled automatically. Quote first, show the user the sell token, buy token, amount, minimum received and network, and wait for an explicit yes before calling `swap`. Develop against testnet with `hood.testnet()`; the repo's example reads the key from `ROBINHOOD_CHAIN_PRIVATE_KEY`. Never hardcode a key.

Buying a Stock Token throws until `hood.config({ acknowledgeEligibility: true })` has been called; reading and selling are never gated. A quote made on one network is rejected after switching networks, so re-quote.

## Errors

Failures are thrown as `HoodError` with a friendly message. Catch it at the call site and surface the message.

## Run the examples from a clone

```bash
git clone https://github.com/nirholas/robinhood-chain-js
cd robinhood-chain-js && npm install && npm run build
node examples/price.mjs AAPL TSLA
node examples/coins.mjs
node examples/portfolio.mjs 0xADDR
```

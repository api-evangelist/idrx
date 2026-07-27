---
name: Mint IDRX with hosted checkout
description: Create an IDRX (or USDT) mint request paid with fiat IDR, redirect the user to the hosted payment page, and confirm settlement reliably.
api: openapi/idrx-openapi.yml
operations: [createMintRequest, getTransactionHistory]
---

# Mint IDRX

Mint IDRX (or USDT) to a wallet on a supported chain, paid with fiat IDR via a hosted checkout.

## Auth (every request)
Send four headers: `idrx-api-key`, `idrx-api-sig` (HMAC-SHA256 signature), `idrx-api-ts` (Unix ms), and a custom `User-Agent` like `my-app/1.0`. A generic User-Agent is blocked with HTTP 403. Signature = `HMAC-SHA256(secret, METHOD + ":" + PATH + ":" + SHA256(body) + ":" + timestamp)`. See `conventions/idrx-conventions.yml`.

## Steps
1. Call `createMintRequest` (`POST /api/transaction/mint-request`) with `toBeMinted`, `destinationWalletAddress`, `networkChainId`, `returnUrl`, and optionally `requestType` (`idrx` default or `usdt`). Min 20,000 IDR / 2 USD; max 1,000,000,000 IDR / 5,555 USD.
2. Redirect the user to `data.paymentUrl`. The returned `data.amount` is the after-fees amount the user actually pays.
3. Do NOT trust the `returnUrl` redirect to confirm payment. Confirm via the mint callback OR by polling `getTransactionHistory` (`GET /api/transaction/user-transaction-history`) filtered on your `merchantOrderId`.
4. Success state is `paymentStatus: PAID` and `userMintStatus: MINTED`. Terminal states: `MINTED`, `REFUND`, `REJECTED`, or `paymentStatus: EXPIRED`. Stop on a terminal state.

## Rules
- Callbacks are single-delivery, unsigned, not retried — always re-verify via `getTransactionHistory` before crediting users.
- On `5xx`, do NOT create a new order; recover the original payload and reconcile.
- No idempotency key exists; dedupe on your own `merchantOrderId`. See `errors/idrx-problem-types.yml`.

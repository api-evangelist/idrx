---
name: Redeem IDRX to a bank account
description: Redeem IDRX back to fiat IDR to a registered bank account, referencing the on-chain burn transaction, and track it to settlement.
api: openapi/idrx-openapi.yml
operations: [getBankMethods, createRedeemRequest, getTransactionHistory]
---

# Redeem IDRX to IDR

Redeem IDRX for fiat IDR paid out to a bank account.

## Auth
Same as all IDRX calls: `idrx-api-key`, `idrx-api-sig`, `idrx-api-ts`, and a custom `User-Agent`. See `conventions/idrx-conventions.yml`.

## Steps
1. Look up the recipient bank with `getBankMethods` (`GET /api/transaction/method`) to get the correct `bankCode`, `bankName`, and `maxAmountTransfer`.
2. Burn the IDRX on-chain and capture the burn `txHash`.
3. Call `createRedeemRequest` (`POST /api/transaction/redeem-request`) with `txHash`, `networkChainId`, `amountTransfer`, `bankAccount`, `bankCode`, `bankName`, `bankAccountName`, and `walletAddress`. If the bank account holder name differs from the IDRX account, include `notes` explaining the purpose.
4. The response `data.burnStatus` starts at `REQUESTED`. Poll `getTransactionHistory` to track it to a terminal state. Funds are credited within max 24 hours.

## Rules
- Additional redeem fees apply (see `getAdditionalFees` with `feeType=REDEEM`): Rp5,000 for <= Rp250M (real-time); Rp35,000 for > Rp250M and < Rp1B (office hours only, Mon-Fri 08:00-15:00 WIB).
- IDRX is not responsible for redeem errors from an incorrect bank/e-wallet account number — validate before submitting.

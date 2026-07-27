---
name: Onboard a user and register a bank account
description: Onboard a new user under an organization account and register a bank account with its deposit wallet address.
api: openapi/idrx-openapi.yml
operations: [onboardUser, addBankAccount, getBankAccounts, getMembers]
---

# Onboard a user

Organizations onboard users, who can then mint and redeem via the org's API keys.

## Auth
Same as all IDRX calls: `idrx-api-key`, `idrx-api-sig`, `idrx-api-ts`, and a custom `User-Agent`. See `conventions/idrx-conventions.yml`.

## Steps
1. Call `onboardUser` (`POST /api/auth/onboarding`) as `multipart/form-data` with `email`, `fullname`, `address`, `idNumber`, and `idFile` (a jpeg/png/jpg/webp image, 256x256 to 4096x4096). Accounts onboarded this way skip part of KYC. The response returns the new `apiKey` and `apiSecret` — store the secret securely.
2. Register a payout bank with `addBankAccount` (`POST /api/auth/add-bank-account`) using `bankAccountNumber` and a `bankCode` from `getBankMethods`. This also creates a `DepositWalletAddress` — IDRX sent there is redeemed to the bank account.
3. List registered accounts with `getBankAccounts` (`GET /api/auth/get-bank-accounts`) and members with `getMembers` (`GET /api/auth/members`).

## Rules
- Treat `apiSecret` as a credential; it is shown once at onboarding.
- Errors use the `{ statusCode, message, data }` envelope; see `errors/idrx-problem-types.yml`.

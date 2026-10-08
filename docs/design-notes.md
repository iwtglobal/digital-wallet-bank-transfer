# Design Notes — Wallet-to-Bank Transfers

Practical notes for engineers and operations teams building **wallet-to-bank payouts** on a digital wallet platform.

## 1. Transfer states

| State | Meaning | Customer sees |
|-------|---------|---------------|
| `INITIATED` | Request received, checks not yet passed | Nothing yet |
| `PENDING` | Wallet debited, amount in suspense, sent to bank | "Processing" with reference |
| `UNKNOWN` | Bank did not answer in time | "Processing" (not "failed") |
| `COMPLETED` | Bank confirmed credit | Success with bank reference |
| `FAILED` | Bank rejected; funds back in wallet | Failure reason, refund confirmation |
| `RETURNED` | Bank accepted, then returned later (e.g. closed account) | Refund notice |

Rules of thumb:
- Only `FAILED` and `RETURNED` move money back to the wallet.
- `UNKNOWN` always goes through a status enquiry before any decision.
- A late success on a transfer already reversed must raise an alert, not post silently.

## 2. Ledger postings

| Event | Debit | Credit |
|-------|-------|--------|
| Transfer submitted | Customer wallet | Payout suspense |
| Fee charged | Customer wallet | Fee income |
| Bank confirms | Payout suspense | Bank settlement account |
| Bank rejects | Payout suspense | Customer wallet |
| Bank returns later | Bank settlement account | Customer wallet |

The payout suspense balance should equal the sum of transfers in `PENDING` and `UNKNOWN`. If it does not, something posted out of order.

## 3. Daily reconciliation checklist

- [ ] Match every `COMPLETED` payout to a bank statement line by reference
- [ ] List bank lines with no matching payout (possible duplicates or manual transfers)
- [ ] Age the suspense balance: anything older than the rail's normal window gets investigated
- [ ] Confirm returns posted back to the right wallet
- [ ] Confirm fee income equals fees charged on completed payouts
- [ ] Record and sign off breaks with a named owner

## 4. Risk controls worth having from day one

- Limits by KYC tier and by beneficiary age (new beneficiaries get lower limits)
- Velocity checks: many payouts to many new accounts in a short time
- Name check against the beneficiary account where the scheme supports it
- Alerts on payouts that follow straight after a SIM change, PIN reset, or device change
- Maker-checker approval for manual reversals and suspense adjustments

## Related

See the main [README](../README.md). Product context: [wallet cash-in, cash-out, transfers and reversals](https://evdsystem.com/evd-software/wallet-cash-in-out-transfers-and-reversals/).

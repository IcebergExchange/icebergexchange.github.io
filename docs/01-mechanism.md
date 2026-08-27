# 01 — Mechanism Specification

## Object model
- **Auction(token)**: recurring per-token market. States: `Committing → Revealing → Cleared|Failed → Settled`.
- **Order**: `{ commitment: hash(side, size, limit_px, salt, owner), bond, escrow }`. Escrow = tokens (sell) or quote (buy), held by program PDA. Bond = flat SOL amount, forfeited on no-reveal.
- **Batch**: the set of revealed orders that clear together at ONE price.

## Phase timeline (V1 params)
| Phase | Duration | What happens |
|---|---|---|
| Committing | until T-10m | sealed commits accepted; escrow + bond locked |
| Revealing | 10 min | owners (or their delegated revealer bot) submit plaintext orders; program verifies `hash == commitment` |
| Clearing | 1 tx | keeper submits clearing solution; program VERIFIES it (see 02) |
| Settlement | same tx / crank | fills paid at uniform price; unfilled escrow returned; no-reveal bonds slashed |

## Clearing rule (uniform price, verifiable)
1. Sort revealed buys by limit desc, sells by limit asc.
2. Clearing price `p*` = price maximizing executable volume; ties → midpoint of the crossing interval.
3. **Band check**: `p* ∈ [ref × (1 − band_bps), ref × (1 + band_bps)]` where `ref` = on-chain TWAP reference (03). Outside band ⇒ batch **Fails** (all escrow returned, bonds returned — a failed batch is never a penalty).
4. Pro-rata on the marginal side at `p*`.
5. Everyone — both sides — settles at exactly `p*`.

## Information properties
- Pre-reveal: nothing about size/side/price is on-chain in plaintext. Commit txs are visible (participation is public; CONTENTS are not) — acceptable for V1, documented honestly.
- Post-settlement: fills are public (on-chain transfers). **Unfilled orders are never distinguishable from small fills or non-participation** — the program returns escrow via the same instruction shape regardless.
- No order book ever exists. There is no sequence, therefore no front-running position.

## Bonds & griefing economics
- Bond ≥ expected value of information gained by fishing (start: 0.25 SOL flat, tune with data).
- No-reveal ⇒ bond → insurance fund (NOT to counterparties — avoids incentive to grief reveals).
- Repeated no-reveals ⇒ wallet-lineage denylist (off-chain policy, on-chain allowlist gate in pilot).

## Explicit non-goals (V1)
- No cross-token netting. No continuous trading. No margin. No leverage. No composability
  guarantees for third-party programs mid-auction.

# 02 — Program Design (Anchor)

## Accounts
- `Venue` (global): authority (multisig), keeper pubkey, fee_bps, insurance_vault, paused.
- `Market` (per token): mint, quote_mint (wSOL), ref_config (TWAP source PDA), band_bps,
  min_order, max_batch_orders (V1: 64), schedule (cadence + phase durations), state, epoch.
- `OrderSlot` (PDA per order, seeded [market, epoch, owner, nonce]): commitment, bond_lamports,
  escrow vault refs, revealed?, filled_amount.
- Vaults: token escrow ATA per order (or pooled per-market with internal accounting — V1 uses
  PER-ORDER vaults: dumber, safer, rent paid by owner, zero commingling).

## Instructions
1. `init_market` (authority) — pins mint, ref, band, schedule.
2. `commit(order_hash, side_hint_none)` — transfers escrow + bond. NOTE: escrow ASSET reveals side
   (token escrow = sell). V1 accepts this (side leaks, size/price do not). V2 option: quote-side
   wrapping to hide side.
3. `reveal(size, limit_px, salt)` — verifies hash, marks revealed. Permissionless crank window.
4. `clear(solution)` — keeper-only. Program RE-VERIFIES the solution in-program:
   - every included order is revealed + within its limit at p*
   - p* inside TWAP band
   - executable volume of solution == max (verified by checking the two candidate neighbor prices
     don't beat it — O(1) check with sorted inputs supplied as remaining accounts)
   - pro-rata math exact, conservation of funds exact
   Keeper computes; program checks. A bad solution cannot settle — keeper compromise ⇒ liveness
   risk only, never price/funds risk. (Authority may propose; the chain enforces.)
5. `settle_order` — crank, pays fills at p*, returns remainder, closes vaults.
6. `slash_unrevealed` — after reveal window, bond → insurance fund, escrow RETURNED (funds are
   never taken for failing to reveal; only the bond).
7. `cancel_market_epoch` (authority) — nuclear: return everything, no clearing. Kill switch.

## Compute strategy
- V1 hard cap **64 orders/batch** → clearing verification fits one tx with sorted remaining
  accounts. Overflow orders roll to next epoch automatically (commit stays live ≤ N epochs).
- If demand outgrows 64: split verification across a `ClearingState` account built by multiple
  `verify_chunk` txs (V2 design, sketched not built).

## Oracle / reference (reuses proven pattern)
- `ref` = TWAP over the token's primary pool, min-samples + min-history + max-age gates,
  cross-checked off-chain by keeper against second source before proposing (keeper refuses to
  propose when sources diverge > threshold; chain still enforces the band regardless).

## Trust model
- Upgrade authority: Squads multisig + 48h timelock from day one (deploy-day, not "later").
- Keeper key: hot, liveness-only power.
- Authority (multisig): market listing, pause, band params — 48h-visible.
- NO instruction moves user funds anywhere except: fill at verified p*, return-to-owner, bond→insurance.

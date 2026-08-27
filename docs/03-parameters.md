# 03 — Parameter Table (V1 pilot)

| Param | Value | Rationale |
|---|---|---|
| batch cadence | 24h fixed (per market, staggered) | thin flow → concentrate it; predictability recruits both sides |
| commit window | until T-10m | max time to gather; late enough to price off fresh ref |
| reveal window | 10 min | short enough to bound drift, long enough for bot reveal redundancy |
| band_bps | 1500 (±15%) around TWAP | wide enough for real size discounts, blocks fake-price clears |
| TWAP window / min-history / max-age | 1800s / 300s / 120s | battle-tested TWAP gate semantics (window / min-history / freshness as three separate constants) |
| max_batch_orders | 64 | single-tx verify headroom |
| min order | 0.5% of pool depth (per-market) | below that, just use the pool — venue is for SIZE |
| max order | 25% of committed opposite interest rolls; hard cap 10% supply | phased exit for mega-positions |
| fee | 75 bps each side on FILLED volume only | no fill, no fee, ever |
| bond | 0.25 SOL flat | > fishing value, < annoyance threshold; revisit with data |
| epochs an unfilled commit persists | 3 (auto-roll, silent) | patient flow without re-signing |
| pilot access | allowlisted wallets both sides | curated cohort until audit + data |

Every param lives in `Market` and changes go through the timelocked multisig — visible 48h before effect.

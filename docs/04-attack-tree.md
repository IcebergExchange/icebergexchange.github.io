# 04 — Attack Tree & Mitigations

1. **Probe & bail** (commit to sniff, never reveal) → bonds forfeit; contents were never visible
   anyway — only participation. Residual: participation-count signaling → mitigate with keeper
   dummy commits (bonded, self-slashing, revealed as zero-size; V1.1).
2. **Ref manipulation** (pump spot pre-batch to drag band) → long TWAP + min-history + batch size
   capped vs pool depth (moving ref costs real money through the pool; band width < manipulation
   profit for capped sizes) + keeper cross-source refusal.
3. **Keeper compromise** → can: stall, propose garbage (rejected on-chain), censor orders from the
   solution. Cannot: move funds, set price outside band, fake reveals. Censorship mitigations:
   solution must include ALL revealed crossing orders (program checks completeness against
   revealed set) — censorship = invalid solution.
4. **Reveal-window sniping** (see reveals land, trade the pool against them) → reveals carry
   size/price but batch still clears banded + at uniform price; sniper can move REF only within
   band economics of (2). Residual accepted + documented; V2: encrypted mempool submission or
   batched reveal via relayer.
5. **Wash/self-cross** (fake volume for token PR) → fees make it cost 150bps round trip; uniform
   price makes it pointless vs pool wash which is cheaper. Not our problem to subsidize; monitored.
6. **Toxic flow** (insider exits pre-news) → disclosed-by-design: buyers knowingly price whale
   exits at a discount; per-market telemetry (holder concentration, dev wallet state) surfaced
   pre-bid. Venue never claims to filter intent.
7. **Rent/dust griefing** (thousands of micro-commits) → min order + per-order rent paid by
   committer + allowlist (pilot).
8. **Program bugs** → the real risk, treated with the only discipline that works: full unit suite + adversarial
   matrix harness on local validator + external audit BEFORE mainnet size + caps ramped by phase
   + kill switch + insurance fund from fees.

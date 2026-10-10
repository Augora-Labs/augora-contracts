# Historical Contract Issue Reports

These files are issue reports from earlier revisions. They may name retired crate and reward-asset identifiers, refer to line numbers that have moved, or describe behavior that has since changed. Treat them as audit history, not as statements about the current contract implementation. Check the active source, tests, and current invariant matrix before relying on a finding.

The reports (02–41) were produced during a security review of an earlier, single-contract revision of the platform. The workspace has since been restructured into separate crates (`prediction_market`, `stellar_trade_token`, `referral_registry`, `leaderboard`) and the reward asset has been replaced by a token contract with minters, allowance and an optional supply cap.

## Index & triage status

| Report | One-line topic | Status |
|---|---|---|
| [issue-02](issue-02.md) | Payout rounding traps dust in the contract; sum of payouts never equals the pool | unverified |
| [issue-03](issue-03.md) | `resolve_market` can sweep the entire pool to fees by resolving to the empty side | unverified |
| [issue-04](issue-04.md) | `AccumulatedFees` is a global fungible pool with no per-market provenance | unverified |
| [issue-05](issue-05.md) | `upgrade()` allows arbitrary WASM replacement with no timelock, delay or multi-sig | unverified — appears still applicable, no timelock on `upgrade` |
| [issue-06](issue-06.md) | `set_config` can re-point the market to malicious contracts with no validation | addressed — config changes are staged, gated by `CONFIG_DELAY_SECS` + `GovernorThreshold`, and cancellable |
| [issue-07](issue-07.md) | No event emission anywhere | addressed — `#[contractevent]`-based event system across the workspace |
| [issue-08](issue-08.md) | `get_market_bettors` iterates an unbounded bettor index (gas DoS) | unverified |
| [issue-09](issue-09.md) | Persistent storage TTLs can expire before claims/refunds (funds locked) | unverified |
| [issue-10](issue-10.md) | `create_market` allows `duration_secs = 0` with no minimum | addressed — `MIN_MARKET_DURATION_SECS` enforced |
| [issue-11](issue-11.md) | `check_rate` underflows on ledger timestamp regression | unverified |
| [issue-12](issue-12.md) | `withdraw_fees` lets a fee recipient drain the entire accumulator uncapped | unverified |
| [issue-13](issue-13.md) | `cancel_refund` zeroes `gross` but leaves `net` intact | unverified |
| [issue-14](issue-14.md) | `OppositeSideBet` permanently locks a user to one side | addressed — the variant is no longer in active use |
| [issue-15](issue-15.md) | `MIN_BET` checked on gross while staked amount is net | unverified |
| [issue-16](issue-16.md) | `get_top_players` uses O(n²) selection sort with full Vec rebuilds | unverified |
| [issue-17](issue-17.md) | `get_rank` returns 0 for any player outside the top-50 | unverified |
| [issue-18](issue-18.md) | `reward`/`add_pts` callable by the market with divergent semantics | unverified |
| [issue-19](issue-19.md) | `reward_bonus`/`add_bonus_pts` don't update `won_bets`/`lost_bets` | unverified |
| [issue-20](issue-20.md) | No way to remove or ban a player from the top list | unverified |
| [issue-21](issue-21.md) | `MinPoints`/`MinSlot` in instance storage with no TTL extension | unverified |
| [issue-22](issue-22.md) | `TopPlayerSlot` reverse lookups not cleaned up on eviction | unverified |
| [issue-23](issue-23.md) | `get_top_players` pagination re-scans the entire list per page | unverified |
| [issue-24](issue-24.md) | Points monotonically increase with no decay or penalty | unverified |
| [issue-25](issue-25.md) | `upsert_top` append path uses `<` instead of `<=` for min tracking | unverified |
| [issue-26](issue-26.md) | `register_referral` doesn't validate the referrer is registered | addressed — `ReferralError::InvalidReferrer` |
| [issue-27](issue-27.md) | `credit` transfers XLM to an arbitrary referrer (reentrancy vector) | unverified |
| [issue-28](issue-28.md) | `ReferralCount`/`ReferralEarnings` TTLs never extended | unverified |
| [issue-29](issue-29.md) | No referral chain depth limit (sybil amplification) | addressed — `MAX_REFERRAL_DEPTH` + `ReferralError::DepthLimitExceeded` |
| [issue-30](issue-30.md) | `register_referral` has no `display_name` length limit | unverified |
| [issue-31](issue-31.md) | `credit` refunds the referral fee to the caller when there's no referrer | unverified |
| [issue-32](issue-32.md) | No cap on referral earnings | unverified |
| [issue-33](issue-33.md) | Legacy key migration creates inconsistent pre/post-upgrade state | unverified |
| [issue-34](issue-34.md) | Legacy reward asset token has no supply cap | addressed — supply cap + boundary tests in `stellar_trade_token` |
| [issue-35](issue-35.md) | `set_minter`/`remove_minter` lack idempotency or authorization | addressed — idempotent minter tests |
| [issue-36](issue-36.md) | `transfer`/`burn` don't extend TTL on balance keys | addressed — `extend_ttl` on transfer paths in `stellar_trade_token` |
| [issue-37](issue-37.md) | Legacy reward asset token has no approve/allowance mechanism | addressed — `approve`/`allowance`/`transfer_from` implemented |
| [issue-38](issue-38.md) | No pause/circuit-breaker mechanism anywhere | addressed — `pause`/`unpause` in `prediction_market` + `ContractPaused` in `referral_registry` |
| [issue-39](issue-39.md) | No upgrade coordination between contracts (ABI drift) | unverified |
| [issue-40](issue-40.md) | Trust model has no revocation | unverified |
| [issue-41](issue-41.md) | Cross-contract call chains have no gas/budget management | unverified |

**Status key**

- **addressed** — a current code path or test in the workspace was located that covers the report.
- **unverified** — not re-confirmed against the current source in this pass; treat as historical per the note above. "Addressed" here is not an audit opinion; it only means the described hazard was not located in the current source.
# Stellar Trade Contract Invariants

This matrix records cross-contract rules that must remain true when market fees, reward values, referral behavior, or storage lifetimes change. The test names below are implemented in `cross_contract_invariants/src/lib.rs` unless otherwise noted.

## Current behavior

| Area | Contract rule | Verification |
| --- | --- | --- |
| Bet fees | 98% of gross stake is credited to the selected side; total fees equal gross minus net. Platform and referral portions are accounted for separately. | `test_inv_cross_fee_conservation`, `test_inv_cross_accumulator_consistency` |
| Pool payout | At resolution, each winning account receives its proportional share. Integer remainder is tracked as market fees; payouts cannot exceed the pool. | `test_inv_cross_fee_conservation`, `prediction_market::tests::test_many_winners_payouts_exact_and_dust_swept` |
| Cancellation | Cancellation refunds eligible gross stakes and reclaims only the cancelled market's fees. | `test_inv_cross_multi_market_isolation`, `prediction_market::tests::test_inv_cancel_then_withdraw_no_overdrain` |
| Empty winning side | Net principal is claimable; fee funds remain protected through the dispute period. | `test_inv_cross_dispute_window_ttl`, `prediction_market::tests::test_empty_side_resolution_pool_to_fees` |
| Fee withdrawals | Each withdrawal is capped; one market's accounting cannot spend another market's fees. | `test_inv_cross_withdraw_cap`, `test_inv_cross_multi_market_isolation` |
| Referral fees | A registered referrer can receive its allocated fee. Missing referrals route the allocation back according to contract rules. Referral depth is bounded. | `test_inv_cross_referral_fee_payment`, `test_inv_cross_referral_depth_isolation`, `test_inv_cross_late_referrer_registration`, `prediction_market::tests::test_inv_referral_depth_does_not_affect_accumulator` |
| Rewards and points | Settled outcomes update leaderboard records and mint rewards through authorized contract calls. | `test_inv_cross_reward_asset_minting_bounded`, `test_inv_cross_leaderboard_points` |
| Supply cap | AUGORA-style supply cap is applied on every reward path — win (`reward`), bonus (`reward_bonus`) and deferred (`claim_pending_rewards`) — and a cap on one path never inflates another. | `leaderboard::tests::test_reward_enforces_supply_cap`, `leaderboard::tests::test_reward_bonus_enforces_supply_cap`, `leaderboard::tests::test_claim_pending_preserves_reward_on_cap_exceeded`, `leaderboard::tests::test_claim_pending_succeeds_after_cap_raised` |
| Storage TTL refresh | `Bet`, `Payout` and `BettorAt` entries are re-bumped on every relevant read/write path (claim, cancel-refund, refresh, rebump) so live entries cannot archive while the market is alive. | `prediction_market::tests::test_claim_rebumps_ttl_entries`, `prediction_market::tests::test_cancel_refund_rebumps_ttl_entries`, `prediction_market::tests::test_refresh_market_ttl_rebumps_bet_and_market`, `test_inv_cross_dispute_window_ttl`, `leaderboard::ttl_tests::test_refresh_player_ttl_rebumps_stats`, `leaderboard::ttl_tests::test_min_points_and_min_slot_survive_ttl_refresh_cycle` |
| Governance threshold | A `set_config` proposal only takes effect after `CONFIG_DELAY_SECS` AND `GovernorThreshold` approvals; it can be cancelled during the window and re-applied after. Live WASM hashes are read on-chain, not caller-supplied. | `prediction_market::tests::test_set_config_is_timelocked`, `prediction_market::tests::test_set_config_multisig_requires_threshold`, `prediction_market::tests::test_set_config_multisig_execute_with_second_approval`, `prediction_market::tests::test_cancel_set_config_during_dispute_window`, `prediction_market::tests::test_set_config_rejects_arbitrary_address` |
| Fee provenance & reserve split | Open-market reserves and proven fees are accounted separately: open-market balances are reserved from withdrawal, per-market fee provenance is isolated, and the on-ledger accumulator matches the sum of ledger entries. | `prediction_market::tests::test_issue163_open_market_fees_are_reserved_from_withdrawal`, `prediction_market::tests::test_per_market_fee_provenance_is_isolated`, `prediction_market::tests::test_inv_accumulator_equals_ledger_sum`, `test_inv_cross_accumulator_consistency`, `test_inv_cross_withdraw_cap` |
| Emergency pause | Paused contract paths reject actions as configured. | `test_inv_cross_emergency_pause_circuit_breaker` |

## Review when changing constants

- Fee rates or withdrawal caps: test fee decomposition, cancellation, and interleaved withdrawals — including the open-market reserve vs. proven-fee split.
- Market durations, dispute windows, or storage TTLs: test create, bet, resolve, dispute, claim, and later reads, and re-run the TTL refresh suite.
- Reward amounts or supply caps: test total supply and **all** reward entry points (win, bonus, deferred claim).
- Governance threshold or config delay: test staged proposals, threshold approvals, cancellation within the window, and re-application.
- Referral depth or fee allocation: test deep chains, late registration, and fee-ledger isolation.

Run the complete suite from this directory with `cargo test --workspace --all-features`. Keep the contract behavior, these invariants, and the user-facing description in sync.
## Non-breaking but important changes

- Snapshot failures between fuzz runs [are now persisted](https://github.com/foundry-rs/foundry/pull/4974), so workarounds are not needed anymore when testing fuzz runs that use snapshots.
- Sensitive broadcast logs are now logged into the `.gitignore`d `cache` directory, instead of being included in the broadcast files that are intended to be committed.
- The base fee calculation on Anvil is now entirely skipped if the fee is set to 0.
- `vm.startPrank` now overwrites the existing prank instead of just erroring. However, the already set prank must be used first.
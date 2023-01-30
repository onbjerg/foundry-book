## Testing

Configuration related to the behavior of `forge test`.

**Sections**

- [General](#general)
- [Fuzz](#fuzz)
- [Invariant](#invariant)

### General

##### `verbosity`

- Type: integer
- Default: 0
- Environment: `FOUNDRY_VERBOSITY` or `DAPP_VERBOSITY`

The verbosity level to use during tests.

- **Level 2 (`-vv`)**: Logs emitted during tests are also displayed.
- **Level 3 (`-vvv`)**: Stack traces for failing tests are also displayed.
- **Level 4 (`-vvvv`)**: Stack traces for all tests are displayed, and setup traces for failing tests are displayed.
- **Level 5 (`-vvvvv`)**: Stack traces and setup traces are always displayed.

##### `ffi`

- Type: boolean
- Default: false
- Environment: `FOUNDRY_FFI` or `DAPP_FFI`

Whether or not to enable the `ffi` cheatcode.

**Warning:** Enabling this cheatcode has security implications for your project, as it allows tests to execute arbitrary programs on your computer.

##### `sender`

- Type: string (address)
- Default: 0x1804c8AB1F12E6bbf3894d4083f33e07309d1f38
- Environment: `FOUNDRY_SENDER` or `DAPP_SENDER`

The value of `msg.sender` in tests.

##### `tx_origin`

- Type: string (address)
- Default: 0x1804c8AB1F12E6bbf3894d4083f33e07309d1f38
- Environment: `FOUNDRY_TX_ORIGIN` or `DAPP_TX_ORIGIN`

The value of `tx.origin` in tests.

##### `initial_balance`

- Type: string (hexadecimal)
- Default: 0xffffffffffffffffffffffff
- Environment: `FOUNDRY_INITIAL_BALANCE` or `DAPP_INITIAL_BALANCE`

The initial balance of the test contracts in wei, written in hexadecimal.

##### `block_number`

- Type: integer
- Default: 1
- Environment: `FOUNDRY_BLOCK_NUMBER` or `DAPP_BLOCK_NUMBER`

The value of `block.number` in tests.

##### `chain_id`

- Type: integer
- Default: 31337
- Environment: `FOUNDRY_CHAIN_ID` or `DAPP_CHAIN_ID`

The value of the `chainid` opcode in tests.

##### `gas_limit`

- Type: integer or string
- Default: 9223372036854775807
- Environment: `FOUNDRY_GAS_LIMIT` or `DAPP_GAS_LIMIT`

The gas limit for each test case.

> ℹ️ **Note**
>
> Due to a limitation in a dependency of Forge, you **cannot raise the gas limit** beyond the default without changing the value to a string.
>
> In order to use higher gas limits use a string:

```toml
gas_limit = "18446744073709551615" # u64::MAX
```

##### `gas_price`

- Type: integer
- Default: 0
- Environment: `FOUNDRY_GAS_PRICE` or `DAPP_GAS_PRICE`

The price of gas (in wei) in tests.

##### `block_base_fee_per_gas`

- Type: integer
- Default: 0
- Environment: `FOUNDRY_BLOCK_BASE_FEE_PER_GAS` or `DAPP_BLOCK_BASE_FEE_PER_GAS`

The base fee per gas (in wei) in tests.

##### `block_coinbase`

- Type: string (address)
- Default: 0x0000000000000000000000000000000000000000
- Environment: `FOUNDRY_BLOCK_COINBASE` or `DAPP_BLOCK_COINBASE`

The value of `block.coinbase` in tests.

##### `block_timestamp`

- Type: integer
- Default: 1
- Environment: `FOUNDRY_BLOCK_TIMESTAMP` or `DAPP_BLOCK_TIMESTAMP`

The value of `block.timestamp` in tests.

##### `block_difficulty`

- Type: integer
- Default: 0
- Environment: `FOUNDRY_BLOCK_DIFFICULTY` or `DAPP_BLOCK_DIFFICULTY`

The value of `block.difficulty` in tests.

##### `gas_reports`

- Type: array of strings (contract names)
- Default: ["*"]
- Environment: `FOUNDRY_GAS_REPORTS` or `DAPP_GAS_REPORTS`

The contracts to print gas reports for.

##### `no_storage_caching`

- Type: boolean
- Default: false
- Environment: `FOUNDRY_NO_STORAGE_CACHING` or `DAPP_NO_STORAGE_CACHING`

If set to `true`, then block data from RPC endpoints in tests will not be cached. Otherwise, the data is cached to `$HOME/.foundry/cache/<chain id>/<block number>`.

##### `[rpc_storage_caching]`

The `[rpc_storage_caching]` block determines what RPC endpoints are cached.

###### `rpc_storage_caching.chains`

- Type: string or array of strings (chain names)
- Default: all
- Environment: N/A

Determines what chains are cached. By default, all chains are cached.

Valid values are:

- "all"
- A list of chain names, e.g. `["optimism", "mainnet"]`

###### `rpc_storage_caching.endpoints`

- Type: string or array of regex patterns (to match URLs)
- Default: remote
- Environment: N/A

Determines what RPC endpoints are cached. By default, only remote endpoints are cached.

Valid values are:

- all
- remote (default)
- A list of regex patterns, e.g. `["localhost"]`

##### `eth_rpc_url`

- Type: string
- Default: none
- Environment: `FOUNDRY_ETH_RPC_URL` or `DAPP_ETH_RPC_URL`

The url of the rpc server that should be used for any rpc calls.

##### `etherscan_api_key`

- Type: string
- Default: none
- Environment: `FOUNDRY_ETHERSCAN_API_KEY` or `DAPP_ETHERSCAN_API_KEY`

The etherscan API key for RPC calls.

#### Pattern Configurations

Below pattern configurations use globs to match which tests to run. The supported glob patterns can be found here: https://docs.rs/glob/latest/glob/struct.Pattern.html  
Here are some examples of patterns that can be used:

- Case sensitive: `*[Ff]oo*`
- Either or: `*Set*,*Update*`
- Both in order: `*Foo*Bar*`
- Include while exclude in front: `*[!Foo]Bar*`

##### `test_pattern`

- Type: glob
- Default: none
- Environment: `FOUNDRY_TEST_PATTERN` or `DAPP_TEST_PATTERN`

Only run test methods matching glob.
Equivalent to `forge test --match-test <TEST_PATTERN>`

##### `test_pattern_inverse`

- Type: glob
- Default: none
- Environment: `FOUNDRY_TEST_PATTERN_INVERSE` or `DAPP_TEST_PATTERN_INVERSE`

Only run test methods not matching glob.
Equivalent to `forge test --no-match-test <TEST_PATTERN_INVERSE>`

##### `contract_pattern`

- Type: glob
- Default: none
- Environment: `FOUNDRY_CONTRACT_PATTERN` or `DAPP_CONTRACT_PATTERN`

Only run test methods in contracts matching glob.
Equivalent to `forge test --match-contract <CONTRACT_PATTERN>`

##### `contract_pattern_inverse`

- Type: glob
- Default: none
- Environment: `FOUNDRY_CONTRACT_PATTERN_INVERSE` or `DAPP_CONTRACT_PATTERN_INVERSE`

Only run test methods in contracts not matching glob.
Equivalent to `forge test --no-match-contract <CONTRACT_PATTERN_INVERSE>`

##### `path_pattern`

- Type: glob
- Default: none
- Environment: `FOUNDRY_PATH_PATTERN` or `DAPP_PATH_PATTERN`

Only runs test methods on files matching the path.

##### `path_pattern_inverse`

- Type: glob
- Default: none
- Environment: `FOUNDRY_PATH_PATTERN_INVERSE` or `DAPP_PATH_PATTERN_INVERSE`

Only runs test methods on files not matching the path.

##### `block_gas_limit`

- Type: integer
- Default: none
- Environment: `FOUNDRY_BLOCK_GAS_LIMIT` or `DAPP_BLOCK_GAS_LIMIT`

The block.gaslimit value during EVM execution.

##### `memory_limit`

- Type: integer
- Default: 33554432
- Environment: `FOUNDRY_MEMORY_LIMIT` or `DAPP_MEMORY_LIMIT`

The memory limit of the EVM in bytes.

##### `names`

- Type: boolean
- Default: false
- Environment: `FOUNDRY_NAMES` or `DAPP_NAMES`

Print compiled contract names.

##### `sizes`

- Type: boolean
- Default: false
- Environment: `FOUNDRY_SIZES` or `DAPP_SIZES`

Print compiled contract sizes.

##### `rpc_endpoints`

- Type: table of RPC endpoints
- Default: none
- Environment: none

This section lives outside of profiles and defines a table of RPC endpoints, where the key specifies the RPC endpoints's name and the value is the RPC endpoint itself.

The value can either be a valid RPC endpoint or a reference to an environment variable (wrapped with in `${}`).

These RPC endpoints can be used in tests and Solidity scripts (see [`vm.rpc`](../../cheatcodes/rpc.md)).

The following example defines an endpoint named `optimism` and an endpoint named `mainnet` that references an environment variable `RPC_MAINNET`:

```toml
[rpc_endpoints]
optimism = "https://optimism.alchemyapi.io/v2/..."
mainnet = "${RPC_MAINNET}"
```

### Fuzz

Configuration values for `[fuzz]` section.

##### `runs`

- Type: integer
- Default: 256
- Environment: `FOUNDRY_FUZZ_RUNS` or `DAPP_FUZZ_RUNS`

The amount of fuzz runs to perform for each fuzz test case. Higher values gives more confidence in results at the cost of testing speed.

##### `max_test_rejects`

- Type: integer
- Default: 65536
- Environment: `FOUNDRY_FUZZ_MAX_TEST_REJECTS`

The maximum number of combined inputs that may be rejected before the test as a whole aborts.
"Global" filters apply to the whole test case. If the test case is rejected, the whole thing is regenerated.

##### `seed`

- Type: string (hexadecimal)
- Default: none
- Environment: `FOUNDRY_FUZZ_SEED`

Optional seed for the fuzzing RNG algorithm.

##### `dictionary_weight`

- Type: integer (between 0 and 100)
- Default: 40
- Environment: `FOUNDRY_FUZZ_DICTIONARY_WEIGHT`

The weight of the dictionary.

##### `include_storage`

- Type: boolean
- Default: true
- Environment: `FOUNDRY_FUZZ_INCLUDE_STORAGE`

The flag indicating whether to include values from storage.

##### `include_push_bytes`

- Type: boolean
- Default: true
- Environment: `FOUNDRY_FUZZ_INCLUDE_PUSH_BYTES`

The flag indicating whether to include push bytes values.

### Invariant

Configuration values for `[invariant]` section.

> ℹ️ **Note**
>
> Configuration for `[invariant]` section has the fallback logic
> for common config entries (`runs`, `seed`, `dictionary_weight` etc).
>
> - If the entries are not set in either section, then the defaults will be used.
> - If the entries are set in the `[fuzz]` section, but are not set in the `[invariant]`
>   section, these values will automatically be set to the values specified in
>   the `[fuzz]` section.
> - For any profile other than `default`:
>   - If the common entries are set in the `[invariant]` (same
>     as `[profile.default.invariant]`) section, then the values from
>     `[invariant]` section will be used.
>   - If the common entries are not set in the `[invariant]` section,
>     but are set in the `[fuzz]` (same as `[profile.default.fuzz]`) section,
>     then the values from the `[fuzz]` section will be used.
>   - If it's none of the cases described above, then the defaults
>     will be used.

##### `runs`

- Type: integer
- Default: 256
- Environment: `FOUNDRY_INVARIANT_RUNS`

The number of runs that must execute for each invariant test group. See also [fuzz.runs](#runs)

##### `depth`

- Type: integer
- Default: 256
- Environment: `FOUNDRY_INVARIANT_DEPTH`

The number of calls executed to attempt to break invariants in one run.

##### `fail_on_revert`

- Type: boolean
- Default: false
- Environment: `FOUNDRY_INVARIANT_FAIL_ON_REVERT`

Fails the invariant fuzzing if a revert occurs.

##### `call_override`

- Type: boolean
- Default: false
- Environment: `FOUNDRY_INVARIANT_CALL_OVERRIDE`

Allows overriding an unsafe external call when running invariant tests. eg. reentrancy checks.

##### `dictionary_weight`

- Type: integer (between 0 and 100)
- Default: 80
- Environment: `FOUNDRY_INVARIANT_DICTIONARY_WEIGHT`

The weight of the dictionary. See also [fuzz.dictionary_weight](#dictionary_weight)

##### `include_storage`

- Type: boolean
- Default: true
- Environment: `FOUNDRY_FUZZ_INCLUDE_STORAGE`

The flag indicating whether to include values from storage. See also [fuzz.include_storage](#include_storage)

##### `include_push_bytes`

- Type: boolean
- Default: true
- Environment: `FOUNDRY_FUZZ_INCLUDE_PUSH_BYTES`

The flag indicating whether to include push bytes values. See also [fuzz.include_push_bytes](#include_push_bytes)

# Invariant Testing
## Overview

Invariant testing allows for a set of invariant expressions to be tested against randomized sequences of pre-defined function calls from pre-defined contracts. After each function call is performed, all defined invariants are asserted.

Invariant testing is a powerful tool to expose incorrect logic in protocols. Due to the fact that function call sequences are randomized and have fuzzed inputs, invariant testing can expose false assumptions and incorrect logic in edge cases and highly complex protocol states.

Invariant testing campaigns have two dimensions, `runs` and `depth`:
- `runs`: Number of times that a sequence of function calls is generated and run.
- `depth`: Number of function calls made in a given `run`. All defined invariants are asserted after each function call is made. If a function call reverts, the `depth` counter still increments.

These and other invariant configuration aspects are explained [`here`](#configuring-invariant-test-execution).

Similar to how standard tests are run in Foundry by prefixing a function name with `test`, invariant tests are denoted by prefixing the function name with `invariant` (e.g., `function invariant_A()`).

### Configuring invariant test execution

Invariant tests execution is governed by parameters that can be controlled by users via Forge configuration primitives. Configs can be applied globally or on a per-test basis. For details on this topic please refer to
 📚 [`Global config`](../reference/config/testing.md) and 📚 [`In-line config`](../reference/config/inline-test-config.md).

## Defining Invariants

Invariants are conditions expressions that should always hold true over the course of a fuzzing campaign. A good invariant testing suite should have as many invariants as possible, and can have different testing suites for different protocol states.

Examples of invariants are:
- *"The xy=k formula always holds"* for Uniswap
- *"The sum of all user balances is equal to the total supply"* for an ERC-20 token.

There are different ways to assert invariants, as outlined in the table below:

<table>
<tr><th>Type</th><th>Explanation</th><th>Example</th></tr>

<tr>

<td>Direct assertions</td>
<td>Query a protocol smart contract and assert values are as expected.</td>
<td>

```solidity
assertGe(
    token.totalAssets(),
    token.totalSupply()
)
```
</td>

</tr>

<tr>

<td>Ghost variable assertions</td>
<td>Query a protocol smart contract and compare it against a value that has been persisted in the test environment (ghost variable).</td>
<td>

```solidity
assertEq(
    token.totalSupply(),
    sumBalanceOf
)
```
</td>

</tr>

<tr>

<td>Deoptimizing (Naive implementation assertions)</td>
<td>Query a protocol smart contract and compare it against a naive and typically highly gas-inefficient implementation of the same desired logic.</td>
<td>

```solidity
assertEq(
    pool.outstandingInterest(),
    test.naiveInterest()
)
```
</td>

</tr>
</table>

### Conditional Invariants

Invariants must hold over the course of a given fuzzing campaign, but that doesn't mean they must hold true in every situation. There is the possibility for certain invariants to be introduced/removed in a given scenario (e.g., during a liquidation).

It is not recommended to introduce conditional logic into invariant assertions because they have the possibility of introducing false positives because of an incorrect code path. For example:

```solidity
function invariant_example() external {
    if (protocolCondition) return;

    assertEq(val1, val2);
}
```

In this situation, if `protocolCondition == true`, the invariant is not asserted at all. Sometimes this can be desired behavior, but it can cause issues if the `protocolCondition` is true for the whole fuzzing campaign unexpectedly, or there is a logic error in the condition itself. For this reason its better to try and define an alternative invariant for that condition as well, for example:

```solidity
function invariant_example() external {
    if (protocolCondition) {
        assertLe(val1, val2);
        return;
    };

    assertEq(val1, val2);
}
```

Another approach to handle different invariants across protocol states is to utilize dedicated invariant testing contracts for different scenarios. These scenarios can be bootstrapped using the `setUp` function, but it is more powerful to leverage *invariant targets* to govern the fuzzer to behave in a way that will only yield certain results (e.g., avoid liquidations).

## Invariant Targets

**Target Contracts**: The set of contracts that will be called over the course of a given invariant test fuzzing campaign. This set of contracts defaults to all contracts that were deployed in the `setUp` function, but can be customized to allow for more advanced invariant testing.

**Target Senders**: The invariant test fuzzer picks values for `msg.sender` at random when performing fuzz campaigns to simulate multiple actors in a system by default. If desired, the set of senders can be customized in the `setUp` function.

**Target Selectors**: The set of function selectors that are used by the fuzzer for invariant testing. These can be used to use a subset of functions within a given target contract.

**Target Artifacts**: The desired ABI to be used for a given contract. These can be used for proxy contract configurations.

**Target Artifact Selectors**: The desired subset of function selectors to be used within a given ABI to be used for a given contract. These can be used for proxy contract configurations.

Priorities for the invariant fuzzer in the cases of target clashes are:

`targetSelectors | targetArtifactSelectors > excludeContracts | excludeArtifacts > targetContracts | targetArtifacts`

### Function Call Probability Distribution

Functions from these contracts will be called at random with fuzzed inputs. The probability of a function being called is broken down by contract and then by function.

For example:

```text
targetContract1: 50%
├─ function1: 50% (25%)
└─ function2: 50% (25%)

targetContract2: 50%
├─ function1: 25% (12.5%)
├─ function2: 25% (12.5%)
├─ function3: 25% (12.5%)
└─ function4: 25% (12.5%)
```

This is something to be mindful of when designing target contracts, as target contracts with less functions will have each function called more often due to this probability distribution.

### Invariant Test Helper Functions
Invariant test helper functions are included in [`forge-std`](https://github.com/foundry-rs/forge-std/blob/master/src/StdInvariant.sol) to allow for configurable invariant test setup. The helper functions are outlined below:

| Function | Description |
|-|-|
| `excludeContract(address newExcludedContract_)` | Adds a given address to the `_excludedContracts` array. This set of contracts is explicitly excluded from the target contracts.|
| `excludeSender(address newExcludedSender_)` | Adds a given address to the `_excludedSenders` array. This set of addresses is explicitly excluded from the target senders. |
| `excludeArtifact(string memory newExcludedArtifact_)` | Adds a given string to the `_excludedArtifacts` array. This set of strings is explicitly excluded from the target artifacts. |
| `targetArtifact(string memory newTargetedArtifact_)` | Adds a given string to the `_targetedArtifacts` array. This set of strings is used for the target artifacts.  |
| `targetArtifactSelector(FuzzSelector memory newTargetedArtifactSelector_)` | Adds a given `FuzzSelector` to the to `_targetedArtifactSelectors` array. This set of `FuzzSelector`s is used for the target artifact selectors. |
| `targetContract(address newTargetedContract_)` | Adds a given address to the to `_targetedContracts` array. This set of addresses is used for the target contracts. This array overwrites the set of contracts that was deployed during the `setUp`. |
| `targetSelector(FuzzSelector memory newTargetedSelector_)` | Adds a given `FuzzSelector` to the to `_targetedSelectors` array. This set of `FuzzSelector`s is used for the target contract selectors. |
| `targetSender(address newTargetedSender_)` | Adds a given address to the to `_targetedSenders` array. This set of addresses is used for the target senders. |


### Target Contract Setup

Target contracts can be set up using the following three methods:
1. Contracts that are manually added to the `targetContracts` array are added to the set of target contracts.
2. Contracts that are deployed in the `setUp` function are automatically added to the set of target contracts (only works if no contracts have been manually added using option 1).
3. Contracts that are deployed in the `setUp` can be **removed** from the target contracts if they are added to the `excludeContracts` array.


## Open Testing

The default configuration for target contracts is set to all contracts that are deployed during the setup. For smaller modules and more arithmetic contracts, this works well. For example:

```solidity
contract ExampleContract1 {

    uint256 public val1;
    uint256 public val2;
    uint256 public val3;

    function addToA(uint256 amount) external {
        val1 += amount;
        val3 += amount;
    }

    function addToB(uint256 amount) external {
        val2 += amount;
        val3 += amount;
    }

}
```

This contract could be deployed and tested using the default target contract pattern:

```solidity
contract InvariantExample1 is Test {

    ExampleContract1 foo;

    function setUp() external {
        foo = new ExampleContract1();
    }

    function invariant_A() external {
        assertEq(foo.val1() + foo.val2(), foo.val3());
    }

    function invariant_B() external {
        assertGe(foo.val1() + foo.val2(), foo.val1());
    }

}
```

This setup will call `foo.addToA()` and `foo.addToB()` with a 50%-50% probability distribution with fuzzed inputs. Inevitably, the inputs will start to cause overflows and the function calls will start reverting. Since the default configuration in invariant testing is `fail_on_revert = false`, this will not cause the tests to fail. The invariants will hold throughout the rest of the fuzzing campaign and the result is that the test will pass. The output will look something like this:

```text
[PASS] invariant_A() (runs: 50, calls: 10000, reverts: 5533)
[PASS] invariant_B() (runs: 50, calls: 10000, reverts: 5533)
```

## Handler-Based Testing

For more complex and integrated protocols, more sophisticated target contract usage is required to achieve the desired results. To illustrate how Handlers can be leveraged, the following contract will be used (an ERC-4626 based contract that accepts deposits of another ERC-20 token):

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.17;

interface IERC20Like {

    function balanceOf(address owner_) external view returns (uint256 balance_);

    function transferFrom(
        address owner_,
        address recipient_,
        uint256 amount_
    ) external returns (bool success_);

}

contract Basic4626Deposit {

    /**********************************************************************************************/
    /*** Storage                                                                                ***/
    /**********************************************************************************************/

    address public immutable asset;

    string public name;
    string public symbol;

    uint8 public immutable decimals;

    uint256 public totalSupply;

    mapping(address => uint256) public balanceOf;

    /**********************************************************************************************/
    /*** Constructor                                                                            ***/
    /**********************************************************************************************/

    constructor(address asset_, string memory name_, string memory symbol_, uint8 decimals_) {
        asset    = asset_;
        name     = name_;
        symbol   = symbol_;
        decimals = decimals_;
    }

    /**********************************************************************************************/
    /*** External Functions                                                                     ***/
    /**********************************************************************************************/

    function deposit(uint256 assets_, address receiver_) external returns (uint256 shares_) {
        shares_ = convertToShares(assets_);

        require(receiver_ != address(0), "ZERO_RECEIVER");
        require(shares_   != uint256(0), "ZERO_SHARES");
        require(assets_   != uint256(0), "ZERO_ASSETS");

        totalSupply += shares_;

        // Cannot overflow because totalSupply would first overflow in the statement above.
        unchecked { balanceOf[receiver_] += shares_; }

        require(
            IERC20Like(asset).transferFrom(msg.sender, address(this), assets_),
            "TRANSFER_FROM"
        );
    }

    function transfer(address recipient_, uint256 amount_) external returns (bool success_) {
        balanceOf[msg.sender] -= amount_;

        // Cannot overflow because minting prevents overflow of totalSupply,
        // and sum of user balances == totalSupply.
        unchecked { balanceOf[recipient_] += amount_; }

        return true;
    }

    /**********************************************************************************************/
    /*** Public View Functions                                                                  ***/
    /**********************************************************************************************/

    function convertToShares(uint256 assets_) public view returns (uint256 shares_) {
        uint256 supply_ = totalSupply;  // Cache to stack.

        shares_ = supply_ == 0 ? assets_ : (assets_ * supply_) / totalAssets();
    }

    function totalAssets() public view returns (uint256 assets_) {
        assets_ = IERC20Like(asset).balanceOf(address(this));
    }

}

```

### Handler Functions

This contract's `deposit` function requires that the caller has a non-zero balance of the ERC-20 `asset`. In the Open invariant testing approach, `deposit()` and `transfer()` would be called with a 50-50% distribution, but they would revert on every call. This would cause the invariant tests to "pass", but in reality no state was manipulated in the desired contract at all. This is where target contracts can be leveraged. When a contract requires some additional logic in order to function properly, it can be added in a dedicated contract called a `Handler`.

```solidity
function deposit(uint256 assets) public virtual {
    asset.mint(address(this), assets);

    asset.approve(address(token), assets);

    uint256 shares = token.deposit(assets, address(this));
}
```

This contract will provide the necessary setup before a function call is made in order to ensure it is successful.

Building on this concept, Handlers can be used to develop more sophisticated invariant tests. With Open invariant testing, the tests run as shown in the diagram below, with random sequences of function calls being made to the protocol contracts directly with fuzzed parameters. This will cause reverts for more complex systems as outlined above.

![Blank diagram](https://user-images.githubusercontent.com/44272939/214752968-5f0e7653-d52e-43e6-b453-cac935f5d97d.svg)

By manually adding all Handler contracts to the `targetContracts` array, all function calls made to protocol contracts can be made in a way that is governed by the Handler to ensure successful calls. This is outlined in the diagram below.

![Invariant Diagrams - Page 2](https://user-images.githubusercontent.com/44272939/216420091-8a5c2bcc-d586-458f-be1e-a9ea0ef5961f.svg)

With this layer between the fuzzer and the protocol, more powerful testing can be achieved.

### Handler Ghost Variables

Within Handlers, "ghost variables" can be tracked across multiple function calls to add additional information for invariant tests. A good example of this is summing all of the `shares` that each LP owns after depositing into the ERC-4626 token as shown above, and using that in the invariant (`totalSupply == sumBalanceOf`).

```solidity
function deposit(uint256 assets) public virtual {
    asset.mint(address(this), assets);

    asset.approve(address(token), assets);

    uint256 shares = token.deposit(assets, address(this));

    sumBalanceOf += shares;
}
```

### Function-Level Assertions

Another benefit is the ability to perform assertions on function calls as they are happening. An example is asserting the ERC-20 balance of the LP has decremented by `assets` during the `deposit` function call, as well as their LP token balance incrementing by `shares`. In this way, handler functions are similar to fuzz tests because they can take in fuzzed inputs, perform state changes, and assert before/after state.

```solidity
function deposit(uint256 assets) public virtual {
    asset.mint(address(this), assets);

    asset.approve(address(token), assets);

    uint256 beforeBalance = asset.balanceOf(address(this));

    uint256 shares = token.deposit(assets, address(this));

    assertEq(asset.balanceOf(address(this)), beforeBalance - assets);

    sumBalanceOf += shares;
}
```

### Bounded/Unbounded Functions

In addition, with Handlers, input parameters can be bounded to reasonable expected values such that `fail_on_revert` in `foundry.toml` can be set to `true`. This can be accomplished using the `bound()` helper function from `forge-std`. This ensures that every function call that is being made by the fuzzer must be successful against the protocol in order to get tests to pass. This is very useful for visibility and confidence that the protocol is being tested in the desired way.

```solidity
function deposit(uint256 assets) external {
    assets = bound(assets, 0, 1e30);

    asset.mint(address(this), assets);

    asset.approve(address(token), assets);

    uint256 beforeBalance = asset.balanceOf(address(this));

    uint256 shares = token.deposit(assets, address(this));

    assertEq(asset.balanceOf(address(this)), beforeBalance - assets);

    sumBalanceOf += shares;
}
```

This can also be accomplished by inheriting non-bounded functions from dedicated "unbounded" Handler contracts that can be used for `fail_on_revert = false` testing. This type of testing is also useful since it can expose issues in assumptions made with `bound` function usage.

```solidity
// Unbounded
function deposit(uint256 assets) public virtual {
    asset.mint(address(this), assets);

    asset.approve(address(token), assets);

    uint256 beforeBalance = asset.balanceOf(address(this));

    uint256 shares = token.deposit(assets, address(this));

    assertEq(asset.balanceOf(address(this)), beforeBalance - assets);

    sumBalanceOf += shares;
}
```

```solidity
// Bounded
function deposit(uint256 assets) external {
    assets = bound(assets, 0, 1e30);

    super.deposit(assets);
}
```

### Actor Management

In the function calls above, it can be seen that `address(this)` is the sole depositor in the ERC-4626 contract, which is not a realistic representation of its intended use. By leveraging the `prank` cheatcodes in `forge-std`, each Handler can manage a set of actors and use them to perform the same function call from different `msg.sender` addresses. This can be accomplished using the following modifier:

```solidity
address[] public actors;

address internal currentActor;

modifier useActor(uint256 actorIndexSeed) {
    currentActor = actors[bound(actorIndexSeed, 0, actors.length - 1)];
    vm.startPrank(currentActor);
    _;
    vm.stopPrank();
}
```

Using multiple actors allows for more granular ghost variable usage as well. This is demonstrated in the functions below:

```solidity
// Unbounded
function deposit(
    uint256 assets,
    uint256 actorIndexSeed
) public virtual useActor(actorIndexSeed) {
    asset.mint(currentActor, assets);

    asset.approve(address(token), assets);

    uint256 beforeBalance = asset.balanceOf(address(this));

    uint256 shares = token.deposit(assets, address(this));

    assertEq(asset.balanceOf(address(this)), beforeBalance - assets);

    sumBalanceOf += shares;

    sumDeposits[currentActor] += assets
}
```

```solidity
// Bounded
function deposit(uint256 assets, uint256 actorIndexSeed) external {
    assets = bound(assets, 0, 1e30);

    super.deposit(assets, actorIndexSeed);
}
```

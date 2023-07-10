## Testing

Forge's flagship feature is testing. Tests are performed using Solidity, and it's blazing fast.

## Unit tests

Unit tests make up the most basic form of testing supported by Forge.

```sh
{{#include ../output/hello_foundry/forge-test:all}}
```

For more information on unit tests, go [to the Testing section](../testing/tests.md).

## Property Based Tests

Property based tests are also supported in forge. They're really effective for testing general behavior and uncovering edge cases.

Writing a fuzz tests is as easy as creating a normal test function with parameters.

```solidity
pragma solidity 0.8.10;

import "forge-std/Test.sol";

contract ContractBTest is Test {
    uint256 testNumber;

    function setUp() public {
        testNumber = 42;
    }

    function add(uint256 a, uint256 b) external returns (uint256) {
        return a + b;
    }

    function test_fuzzAdd(uint256 x, uint256 y) public {
        // We can limit the range of our inputs using bound.
        x = bound(x, 0, 100);
        y = bound(y, 0, 100);
        uint256 result = add(x, y);
        assertEq(x + y, result);
    }
}
```

For more information on property based tests, go [to the fuzz testing page on the Testing section](../testing/fuzz-testing.md).

## Stateful Fuzz Tests

Stateful fuzz tests (also commonly called Invariant Tests in the community) are a really useful form of advanced testing available on Foundry. They allow for a set of invariant expressions to be tested against randomized sequences of pre-defined function calls from pre-defined contracts. After each function call is performed, all defined invariants are asserted.

For more information on stateful fuzz tests, go [to the stateful fuzz testing page on the Testing section](../testing/invariant-testing.md).

## Cheatcodes

Forge has a powerful feature to help you write comprehensive tests for your smart contract suite called cheatcodes. They help you manipulate the state of the local blockchain and EVM, which means you can easily replicate extreme conditions on your tests or mainnet conditions outright. They're easily accessible through the `vm` instance available in Forge Standard Library's `Test` contract.


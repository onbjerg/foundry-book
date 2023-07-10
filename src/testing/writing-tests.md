## Writing Tests

Tests are written in Solidity. If the test function reverts, the test fails, otherwise it passes.

Let's go over the most common way of writing tests, using the [Forge Standard Library](https://github.com/foundry-rs/forge-std)'s `Test` contract, which is the preferred way of writing tests with Forge.

In this section, we'll go over the basics using the functions from the Forge Std's `Test` contract, which is itself a superset of [DSTest](https://github.com/dapphub/ds-test). You will learn how to use more advanced stuff from the Forge Standard Library [soon](./forge-std.md). 

DSTest provides basic logging and assertion functionality. To get access to the functions, import `forge-std/Test.sol` and inherit from `Test` in your test contract:

```solidity
{{#include ../../projects/writing_tests/test/Basic.t.sol:import}}
```

Let's examine a basic test:

```solidity
{{#include ../../projects/writing_tests/test/Basic.t.sol:all}}
```

Forge uses the following keywords in tests:

- `setUp`: An optional function invoked before each test case is run.
    ```solidity
{{#include ../../projects/writing_tests/test/Basic.t.sol:setUp}}
    ```
- `test`: Functions prefixed with `test` are run as a test case.
    ```solidity
{{#include ../../projects/writing_tests/test/Basic.t.sol:testNumberIs42}}
    ```
- `testFail`: The inverse of the `test` prefix - if the function does not revert, the test fails.
    ```solidity
{{#include ../../projects/writing_tests/test/Basic.t.sol:testFailSubtract43}}
    ```
    A good practice is to use the pattern `test_Revert[If|When]_Condition` in combination with the [`expectRevert`](../cheatcodes/expect-revert.md) cheatcode (cheatcodes are explained in greater detail in the following [section](./cheatcodes.md)). Also, other testing practices can be found in the [Tutorials section](../tutorials/best-practices.md). 
    Now, instead of using `testFail`, you know exactly what reverted and with which error:
    ```solidity
{{#include ../../projects/writing_tests/test/Basic2.t.sol:testCannotSubtract43}}
    ```
<br>

Tests are deployed to `0xb4c79daB8f259C7Aee6E5b2Aa729821864227e84`. If you deploy a contract within your test, then
`0xb4c...7e84` will be its deployer. If the contract deployed within a test gives special permissions to its deployer,
such as `Ownable.sol`'s `onlyOwner` modifier, then the test contract `0xb4c...7e84` will have those permissions.

> ⚠️ **Note**
>
> Test functions must have either `external` or `public` visibility. Functions declared as `internal` or
> `private` won't be picked up by Forge, even if they are prefixed with `test`.

### Shared setups

It is possible to use shared setups by creating helper abstract contracts and inheriting them in your test contracts:

```solidity
abstract contract HelperContract {
    address constant IMPORTANT_ADDRESS = 0x543d...;
    SomeContract someContract;
    constructor() {...}
}

contract MyContractTest is Test, HelperContract {
    function setUp() public {
        someContract = new SomeContract(0, IMPORTANT_ADDRESS);
        ...
    }
}

contract MyOtherContractTest is Test, HelperContract {
    function setUp() public {
        someContract = new SomeContract(1000, IMPORTANT_ADDRESS);
        ...
    }
}
```

<br>

> 💡 **Tip**
>
> Use the [`getCode`](../cheatcodes/get-code.md) cheatcode to deploy contracts with incompatible Solidity versions.

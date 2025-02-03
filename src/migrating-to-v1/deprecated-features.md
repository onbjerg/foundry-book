## Deprecated features

With Foundry v1, we've started deprecating features that are considered anti-patterns or might be misleading and confusing for developers. Deprecated means that while you can continue using these features, it is now a discouraged practice and you should plan on migrating away from using these.

### Removing the `invariant` prefix

The `invariant` test prefix has now been deprecated, and the new expected prefix is `statefulFuzz`. This is mainly to have more correctness on naming: Invariants can be tested with regular fuzz tests. The difference between Foundry's fuzz and invariant tests is that fuzz tests are *stateless* while invariant tests are *stateful*. This means that now writing tests in this manner is valid:

```solidity
/// Old, deprecated way of declaring an invariant test
function invariantTestEq() public {
    // Assert your invariants
}

/// New
function statefulFuzzTestEq() public  {
    // Assert your invariants
}
```

### `testFail`

Using `testFail` to write failing tests is now discouraged, because it can introduce footguns in a test. Specifically, it cannot distinguish between revert reasons, and therefore tests may inadvertently pass and this is hard to detect.

A better pattern is to use `expectRevert` to ensure that a function call reverted in a specific way. This way, the test failure is expected explicitly, removing the possibility that the test fails in an unintended, undetectable way.

An example:

```solidity

contract Mock {
    function revert_() public {
        revert("This reverts");
    }
}

contract TestFailDeprecated is Test {
    Mock public mock;

    function setUp() public {
        mock = new Mock();
    }

    /// Deprecated way.
    function testFailReverter() public {
        mock.revert_();
    }

    // Better way, without using testFail.
    /// The call to revert_ has been refactored and is now expected to fail
    /// with expectRevert()
    function test_revertIf_alwaysReverts() public {
        vm.expectRevert("This reverts");
        // Call using `this` to increase depth
        this.exposed_call_Reverter();
    }

    function exposed_callReverter() public {
        mock.revert_();
    }

}
```

### Removing cheatcodes support on some precompiles

It is now impossible to use the following cheatcodes on precompiles: `etch`, `load`, `store`, `deal`. The rationale is that utilizing cheatcodes on precompile addresses (0 < n < 9) can cause unexpected behavior and test failures. Prefer using the `makeAddr` cheatcode for creating addresses to use in tests.
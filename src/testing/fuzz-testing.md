## Fuzz Testing

Forge supports property based testing.

Property-based testing is a way of testing general behaviors as opposed to isolated scenarios.

Let's examine what that means by writing a unit test, finding the general property we are testing for, and converting it to a property-based test instead:

```solidity
{{#include ../../projects/fuzz_testing/test/Safe.t.sol.1:all}}
```

Running the test, we see it passes:

```sh
{{#include ../output/fuzz_testing/forge-test-no-fuzz:all}}
```

This unit test _does test_ that we can withdraw ether from our safe. However, who is to say that it works for all amounts, not just 1 ether?

The general property here is: given a safe balance, when we withdraw, we should get whatever is in the safe.

Forge will run any test that takes at least one parameter as a property-based test, so let's rewrite:

```solidity
{{#include ../../projects/fuzz_testing/test/Safe.t.sol.2:contract_prelude}}
    // ...

{{#include ../../projects/fuzz_testing/test/Safe.t.sol.2:test}}
}
```

If we run the test now, we can see that Forge runs the property-based test, but it fails for high values of `amount`:

```sh
$ forge test
{{#include ../output/fuzz_testing/forge-test-fail-fuzz:output}}
```

The default amount of ether that the test contract is given is `2**96 wei` (as in DappTools), so we have to restrict the type of amount to `uint96` to make sure we don't try to send more than we have:

```solidity
{{#include ../../projects/fuzz_testing/test/Safe.t.sol.3:signature}}
```

And now it passes:

```sh
{{#include ../output/fuzz_testing/forge-test-success-fuzz:all}}
```

You may want to exclude certain cases using the [`assume`](../cheatcodes/assume.md) cheatcode. In those cases, fuzzer will discard the inputs and start a new fuzz run:

```solidity
function testFuzz_Withdraw(uint96 amount) public {
    vm.assume(amount > 0.1 ether);
    // snip
}
```

There are different ways to run property-based tests, notably parametric testing and fuzzing. Forge only supports fuzzing.

### Interpreting results

You might have noticed that fuzz tests are summarized a bit differently compared to unit tests:

- "runs" refers to the amount of scenarios the fuzzer tested. By default, the fuzzer will generate 256 scenarios, but this and other test execution parameters can be setup by the user. Fuzzer configuration details are provided [`here`](#configuring-fuzz-test-execution).
- "μ" (Greek letter mu) is the mean gas used across all fuzz runs
- "~" (tilde) is the median gas used across all fuzz runs

### Configuring fuzz test execution

Fuzz tests execution is governed by parameters that can be controlled by users via Forge configuration primitives. Configs can be applied globally or on a per-test basis. For details on this topic please refer to
 📚 [`Global config`](../reference/config/testing.md) and 📚 [`In-line config`](../reference/config/inline-test-config.md).

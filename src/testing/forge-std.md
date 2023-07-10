## Forge Standard Library Overview

Forge Standard Library (Forge Std for short) is a collection of helpful contracts that make writing tests easier, faster, and more user-friendly.

Using Forge Std is the preferred way of writing tests with Foundry.

It provides all the essential functionality you need to get started writing tests:

- `Vm.sol`: Up-to-date cheatcodes interface
- `console.sol` and `console2.sol`: Hardhat-style logging functionality
- `Script.sol`: Basic utilities for [Solidity scripting](../tutorials/solidity-scripting.md)
- `Test.sol`: A superset of DSTest containing standard libraries, a cheatcodes instance (`vm`), and Hardhat console

Simply import `Test.sol` and inherit from `Test` in your test contract:

```solidity
import "forge-std/Test.sol";

contract ContractTest is Test { ...
```

Now, you can:

```solidity
// Access Hevm via the `vm` instance
vm.startPrank(alice);

// Assert and log using Dappsys Test
assertEq(dai.balanceOf(alice), 10000e18);

// Log with the Hardhat `console` (`console2`)
console.log(alice.balance);

// Use anything from the Forge Std std-libraries
deal(address(dai), alice, 10000e18);
```

To import the `Vm` interface or the `console` library individually:

```solidity
import "forge-std/Vm.sol";
```

```solidity
import "forge-std/console.sol";
```

**Note:** `console2.sol` contains patches to `console.sol` that allows Forge to decode traces for calls to the console, but it is not compatible with Hardhat.

```solidity
import "forge-std/console2.sol";
```

### Standard libraries

Forge Std currently consists of six standard libraries.

#### Std Logs

Std Logs expand upon the logging events from the [`DSTest`](../reference/ds-test.md#logging) library.

#### Std Assertions

Std Assertions expand upon the assertion functions from the [`DSTest`](../reference/ds-test.md#asserting) library.

#### Std Cheats

Std Cheats are wrappers around Forge cheatcodes that make them safer to use and improve the DX.

You can access Std Cheats by simply calling them inside your test contract, as you would any other internal function:

```solidity
// set up a prank as Alice with 100 ETH balance
hoax(alice, 100 ether);
```

#### Std Errors

Std Errors provide wrappers around common internal Solidity errors and reverts.

Std Errors are most useful in combination with the [`expectRevert`](../cheatcodes/expect-revert.md) cheatcode, as you do not need to remember the internal Solidity panic codes yourself. Note that you have to access them through `stdError`, as this is a library.

```solidity
// expect an arithmetic error on the next call (e.g. underflow)
vm.expectRevert(stdError.arithmeticError);
```

#### Std Storage

Std Storage makes manipulating contract storage easy. It can find and write to the storage slot(s) associated with a particular variable.

The `Test` contract already provides a `StdStorage` instance `stdstore` through which you can access any std-storage functionality. Note that you must add `using stdStorage for StdStorage` in your test contract first.

```solidity
// find the variable `score` in the contract `game`
// and change its value to 10
stdstore
    .target(address(game))
    .sig(game.score.selector)
    .checked_write(10);
```

#### Std Math

Std Math is a library with useful mathematical functions that are not provided in Solidity.

Note that you have to access them through `stdMath`, as this is a library.

```solidity
// get the absolute value of -10
uint256 ten = stdMath.abs(-10)
```

<br>

> 📚 **Reference**
>
> See the [Forge Standard Library Reference](../reference/forge-std/) for a complete overview of Forge Standard Library.

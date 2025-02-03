## Breaking changes

### Table of contents
1. [Introduction](#introduction)
2. [`expectEmit`](#expectemit)
3. [`expectCall`](#expectcall)
4. [`expectRevert`](#expectrevert)
5. [Testing internal or library calls with the new changes](#testing-internal-calls-or-libraries-with-expectcall-expectrevert-and-expectemit)

### Introduction

When it comes to breaking changes, there has been a slight rework for the assertion (`expect*`) cheatcodes that removes previously allowed unintended or confusing behavior.

The biggest change is that these cheatcodes will now work only for the next call, not at the "test" level anymore. The philosophy behind this is that by making these cheatcodes more strict through this requirement, tests will be more accurate as the cheatcodes will now only succeed under more strict conditions.

#### `expectEmit`

`expectEmit` previously allowed you to assert that a log was emitted during the execution of the test. With v1, the following changes have been made:

- It now only works for the _next call_. This means if you used to declare all your expected emits at the top of the test, you may now have to move them to just before the next call you perform. Cheatcode calls are ignored. As long as the events are emitted during the next call's execution, they will be matched.
- The events have to be _ordered_. That means, if you're trying to match events [A, B, C], you must declare them in this order.
    - Skipping events is possible. Therefore, if a function emits events [A, B, C, D, E] and you want to match [B, D, E], you just need to declare them in that order.

To illustrate, see the following examples:

```solidity

contract Emitter {
    event A(uint256 indexed a);
    event B(uint256 b);
    event C(uint256 c1, uint256 c2);
    event D(bytes32 d1, uint256 f2, address indexed d3);
    event E(uint256 indexed e);

    /// emit() emits events [A, B, C, D, E]
    function emitEvent() external {
        emit A(1);
        emit B(2);
        emit C(3, 3);
        emit D(bytes32("gm"), 4, address(0xc4f3));
        emit E(5);
    }
}

contract EmitTest is Test {
    Emitter public emitter;

    event A(uint256 indexed a);
    event B(uint256 b);
    event C(uint256 c1, uint256 c2);
    event D(bytes32 d1, uint256 f2, address indexed d3);
    event E(uint256 indexed e);

    function setUp() public {
        emitter = new Emitter();
    }

    // CORRECT BEHAVIOR: Declare all your expectEmits just before the next call,
    // on the test.
    // emitEvent() emits [A, B, C, D, E], and we're expecting [A, B, C, D, E].
    // This passes.
    function testExpectEmit() public {
        // A quick refresher on `vm.expectEmit` params:
        // The first three booleans indicate if you want to check `topic1`, `topic2` and `topic3`.
        // `topic0` is always checked.
        // The last boolean indicates if you want to check the data.
        // If you want to read further, you can go to the `vm.expectEmit` reference: https://book.getfoundry.sh/cheatcodes/expect-emit?highlight=expectEmit#expectemit
        vm.expectEmit(true, false, false, true);
        emit A(1);
        vm.expectEmit(false, false, false, true);
        emit B(2);
        vm.expectEmit(false, false, false, true);
        emit C(3, 3);
        // It's also possible to skip all parameters entirely and just call the cheatcode.
        vm.expectEmit();
        emit D(bytes32("gm"), 4, address(0xc4f3));
        vm.expectEmit();
        emit E(5);
        emitter.emitEvent();
    }

    // CORRECT BEHAVIOR: Declare all your expectEmits just before the next call,
    // on the test.
    // emitEvent() emits [A, B, C, D, E], and we're expecting [B, D, E].
    // This passes.
    function testExpectEmitWindow() public {
        vm.expectEmit();
        emit B(2);
        vm.expectEmit();
        emit D(bytes32("gm"), 4, address(0xc4f3));
        vm.expectEmit();
        emit E(5);
        emitter.emitEvent();
    }

    // INCORRECT BEHAVIOR: Declare all your expectEmits in the wrong order.
    // emitEvent() emits [A, B, C, D, E], and we're expecting [D, B, E].
    // This fails, as D is emitted after B, not before.
    function testExpectEmitWindowFailure() public {
        vm.expectEmit();
        emit D(bytes32("gm"), 4, address(0xc4f3));
        vm.expectEmit();
        emit B(2);
        vm.expectEmit();
        emit E(5);
        emitter.emitEvent();
    }

    // CORRECT BEHAVIOR: Declare all your expectEmits in an internal function.
    // Calling a contract function internally is a JUMP, not a call,
    // therefore it's a valid pattern, useful if you have a lot of events to expect.
    // emitEvent() emits [A, B, C, D, E], and we're expecting [B, D, E].
    // This passes.
    function testExpectEmitWithInternalFunction() public {
        declareExpectedEvents();
        emitter.emitEvent();
    }

    function declareExpectedEvents() internal {
        vm.expectEmit(true, false, false, true);
        emit B(2);
        vm.expectEmit(true, false, false, true);
        emit D(bytes32("gm"), 4, address(0xc4f3));
        vm.expectEmit(true, false, false, true);
        emit E(5);
    }
}
```

As illustrated above, if you'd like to make your tests more brief, you can declare your expected events in an internal function in your test contract to remove visual clutter. This is not an external call, which means they won't count for detecting events.

#### `expectCall`

Just like `expectEmit`, `expectCall` has gotten a similar rework. Before, it worked at the test level, which meant you could declare the calls you expect to see upfront and then start calling external functions. The new behavior is the following one:

- It now only works for the next _call's subcalls_. This means that the call(s) you expect need to happen inside an external call for them to be detected. Calling it at the test level (directly) won't count.
    - This is explained through "depth": the `expectCall` cheatcode is called at a "test-level" depth, so we enforce that the calls we expect to see are made at a "deeper" depth to be able to count them properly. Depth is increased whenever we make an external call. Internal calls do not increase depth.

To illustrate, see the following examples:

```solidity
contract Protocol {
    function doSomething() external {
        // do something...
    }
}

contract Caller {
    Protocol public protocol;

    function setUp() public {
        protocol = new Protocol();
    }

    function foo() public {
        // do something...
    }

    function baz() public {
        // do something...
    }

    function bar() public {
        protocol.doSomething();
        // do something...
    }
}

contract ExpectCallTest is Test {
    Caller public caller;

    function setUp() public {
        caller = new Caller();
    }

    // CORRECT BEHAVIOR: We expect a call to `doSomething()` in the next call,
    // So we declare our expected call, and then call `caller.bar()`, which will
    // call `protocol.doSomething()` internally.
    // `doSomething()` is nested inside `bar`, so this passes.
    function testExpectCall() public {
        vm.expectCall(caller.protocol.address, abi.encodeCall(Protocol.doSomething, ()));
        // This will call bar internally, so this is valid.
        caller.bar();
    }

    // INCORRECT BEHAVIOR: We expect a call to `doSomething()` in the next call,
    // So we declare our expected call, and then call `protocol.doSomething()` directly.
    // `doSomething()` is not nested, but rather called at the test level.
    // This doesn't satisfy the depth requirement described above,
    // so this fails.
    function testExpectCallFailure() public {
        vm.expectCall(caller.protocol.address, abi.encodeCall(Protocol.doSomething, ()));
        // We're calling doSomething() directly here.
        // This is not inside the next call's subcall tree, but rather a test-level
        // call. This fails.
        caller.protocol().doSomething();
    }

    // CORRECT BEHAVIOR: Sometimes, we want to directly expect the call we're gonna
    // call next, but we need to satisfy the depth requirement. In those cases,
    // this pattern can be used.
    // Calling exposed_callFoo using `this` will create a new call context which
    // will increase depth. In its subcall tree, we'll detect `exposed_callFoo`,
    // which will make the test pass.
    // See more info on the best practices section:
    // https://book.getfoundry.sh/tutorials/best-practices#internal-functions
    function testExpectCallWithNest() public {
        vm.expectCall(address(caller), abi.encodeWithSelector(caller.foo.selector));
        this.exposed_callFoo();
    }

    function exposed_callFoo() public {
        caller.foo();
    }
}
```

#### `expectRevert`

`expectRevert` currently allows to expect for reverts at the test-depth level. This means that a revert can be declared at the top of the test, and as long as a function in the test reverts, the test will pass. The new behavior is the following:

- `expectRevert` CANNOT be used with other `expect` cheatcodes. It will fail if it detects they're being used at the same time.
    - The reasoning for this is that calls that revert are rolled back, and events and calls therefore never happen. This restriction helps model real execution behavior.
- `expectRevert` now works only for the next call. If the next call does not revert, the cheatcode will fail and therefore the test will also fail. The depth of the revert does not matter: as long as the next call reverts, either immediately or deeper into its call subtree, the cheatcode will succeed.

To illustrate, see the following examples:

```solidity

contract Reverter {
    function revertWithMessage(string memory message) public pure {
        require(false, message);
    }
    
    function doNotRevert() public {
        // do something unrelated, do not revert.
    }
}

contract ExpectRevertTest is Test {
    Reverter reverter;

    function setUp() public {
        reverter = new Reverter();
    }

    /// CORRECT BEHAVIOR: We expect `revertWithMessage` to revert,
    /// so we declare a expectRevert and then call the function.
    function testExpectRevertString() public {
        vm.expectRevert(bytes("revert"));
        reverter.revertWithMessage("revert");
    }

    // INCORRECT BEHAVIOR: We correctly expect that a function will revert in this
    // test, but we declare the expected revert before the wrong function,
    // `doNotRevert()`, which does not revert. The cheatcode therefore fails.
    function testFailRevertNotOnImmediateNextCall() public {
        // expectRevert should only work for the next call. However,
        // we do not immediately revert, so this test fails.
        vm.expectRevert("revert");
        reverter.doNotRevert();
        reverter.revertWithMessage("revert");
    }
}
```

### Testing internal calls or libraries with `expectCall`, `expectRevert` and `expectEmit`

With the changes to `expectCall` and `expectRevert`, library maintainers might need to modify tests that expect calls or reverts on library functions that might be marked as internal. As an external call needs to be forced to satisfy the depth requirement, an intermediate contract that exposes an identical API to the function that needs to be called can be used.

See the following example for testing a revert on a library.

```solidity
library MathLib {
    function add(uint a, uint b) internal pure returns (uint256) {
        return a + b;
    }
}

library EmitLib {
    event Log();
    function emit() internal {
        emit Log();
    }
}

// Intermediate "mock" contract that calls the library function
// and returns the result.
contract MathLibMock {
    function add(uint a, uint b) external returns (uint256) {
        return MathLib.add(a, b);
    }
}

contract EmitLibMock {
    function log() external {
        EmitLib.emit();
    }
}

contract MathLibTest is Test {
    MathLibMock public mathMock;
    EmitLibMock public emitMock;

    event Log();

    function setUp() public {
        mathMock = new MathLibMock();
        emitMock = new EmitLibMock();
    }

    // INCORRECT BEHAVIOR: MathLib.add will revert due to arithmetic errors,
    // but as the function is marked as internal, it'll be a test level revert
    // instead of a call revert that expectRevert is supposed to catch.
    // The test will fail, and the error will let you know that you have a
    // "dangling" expectRevert.
    function testRevertOnMathLibWithNoMock() public {
        vm.expectRevert(stdError.arithmeticError);
        MathLib.add(type(uint256).max, 1);
    }

    // CORRECT BEHAVIOR: mock.add will revert due to arithmetic errors,
    // and it will be successfully detected by the `expectRevert` cheatcode.
    function testRevertOnMathLibWithMock() public {
        vm.expectRevert(stdError.arithmeticError);
        mathMock.add(type(uint256).max, 1);
    }

    // CORRECT BEHAVIOR: Use the mock contract to expect a call on a lib
    function testCanCallMathLib() public {
        vm.expectCall(address(mathMock), abi.encodeWithSelector(mathMock.add.selector, 1, 2));
        mathMock.add(1, 2);
    }

    // CORRECT BEHAVIOR: The same pattern applies to `expectRevert`.
    function testLibraryEmitWithMock() public {
        vm.expectEmit();
        emit Log();
        emitMock.log();
    }
}
```
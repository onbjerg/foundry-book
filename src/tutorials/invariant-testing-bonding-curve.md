## Invariant Testing Bonding Curve


### Introduction

This tutorial will cover invariant testing, using **Bonding Curve Implementation** as a target example. All invariant tests are written in Solidity using the `Foundry Invariant Testing` feature.

However, this guide is for educational purposes only. The code is not audited. Please do not use it in production.

> 💡 Note: A full implementation of the bonding curve can be found [here](https://github.com/Ratimon/bonding-curves), and for further reading about invariant testing, we can check out the `Invariant Testing` [reference](../forge/invariant-testing.md).

### Quick Start

The general process of invariant testing is that the foundry will call a sequence of pre-defined **actions**  with random input in the target contract.

This will be run multiple times to ensure the correctness of invariants.

> 💡 Note: The process of making an action call with random input is called **fuzzing**.

> 💡 Note: The number of times that a sequence of function calls is generated and run is called **run**.

After each single run, the system will be checked to see whether the defined **invariants** hold true or not.

The key consderation is to define these:

1. **Invariants** (a set of properties that will always remain true)

2. **Actions**  (a set of things that can happen during each run)

To get started, make sure to clone the repository. All commands will be available in the `Makefile`. We can run the fuzzing campagin by running the following command:

```sh
make invariant-LinearBondingCurve
```

> 💡 Note: Other commands for this tutorial can be found in [ `Makefile`](https://github.com/Ratimon/bonding-curves/blob/master/Makefile).

I note that the default configuration (at [ `foundry.toml`](https://github.com/Ratimon/bonding-curves/blob/master/foundry.toml)) for invariant testing is as follows:

```toml
[invariant]
runs          = 1000    # The number of times to run the invariant tests
depth         = 100   # The number of random action calls to make in the invariant tests
fail_on_revert = false
```


### Define Invariants

To specify invariants, we need to build the mental model of thinking about the incorrect states that could possibly be reached.

> 💡 Note: An invariant is a term for a logical assertion about something (a contract in our case) that is always true.


There are two kinds of invariants we can define, including:

1. Function-level invariants

- They should be stateless, or it does not depend on much of our target system.
- They could be tested in an isolated manner.

> 💡 Note: The notable example could be depositing tokens into the contract.

2. System-level invariants

- They should be stateful, or it relies on the smart contract state.
- They depend on much of the entire system and its relevant deployment logic.


> 💡 Note: For example, the system-level invariant for ERC20 is that the total number of ERC20 tokens is always equal to or greater than the sum of the individual ERC 20 token balances of all token holders.


### Define System-level Invariants

In our case, our state variables (found in [`BondingCurve.sol`](https://github.com/Ratimon/bonding-curves/blob/master/src/bondingcurves/BondingCurve.sol)) used to define invariants are:

```solidity
    /** ... */
    abstract contract BondingCurve is IBondingCurve, Initializable, Pausable, Ownable2Step, Timed {
        /** ... */
        /**
        * @notice the total amount of sale token purchased on bonding curve
        *
        */
        UD60x18 public override totalPurchased;

        /**
        * @notice the cap on how much sale token can be minted by the bonding curve
        *
        */
        UD60x18 public override cap;

        /**
        * @notice returns how close to the cap we are
        *
        */
        function availableToSell() public view override returns (UD60x18) {
            return cap.sub(totalPurchased);
        }

        /**
        * @notice balance of accepted token the bonding curve
        * @return the amount of accepted token held in contract and ready to be allocated
        *
        */
        function reserveBalance() public view virtual override returns (UD60x18) {
            return ud(acceptedToken.balanceOf(address(this)));
        }

    /** ... */

    }

```

Then, we could specify and write assertions in [`LinearBondingCurve.invariants.t.sol`](https://github.com/Ratimon/bonding-curves/blob/master/test/invariant/LinearBondingCurve.invariants.t.sol) as follows:

1.  Invariant 1: totalPurchased + avalableToSell = cap

```solidity

    function invariant_totalPurchasedPlusAvalableToSell_eq_cap() public {
        assertEq(
            unwrap(linearBondingCurve.totalPurchased().add(linearBondingCurve.availableToSell())),
            unwrap(linearBondingCurve.cap())
        );
    }

```

2.  Invariant 2: avalableToSell >= 0

```solidity

    function invariant_AvalableToSell_gt_zero() public {
        assertGt(unwrap(linearBondingCurve.availableToSell()), 0);
    }


```

3.  Invariant 3: availableToSell = amount of ERC20 token amount in bonding curve contract

```solidity
    function invariant_AvalableToSell_eq_saleTokenBalance() public {
        assertEq(unwrap(linearBondingCurve.availableToSell()), IERC20(saleToken).balanceOf(address(linearBondingCurve)));
    }
```

4.  Invariant 4:  Poolbalance =  y = f(x = currentTokenPurchased) =  slope/2 * (currentTokenPurchased)^2 + initialPrice * (currentTokenPurchased)


```solidity
    function invariant_Poolbalance_eq_saleTokenBalance() public {
        UD60x18 acceptedTokenSupply = ud(IERC20(acceptedToken).balanceOf(address(linearBondingCurve)));
        assertEq(
            unwrap(LinearCurve(address(linearBondingCurve)).getPoolBalance(acceptedTokenSupply)),
            unwrap(linearBondingCurve.totalPurchased())
        );
    }
```


### Defining Action Logics and Function-level Invariants

Ok !! We have defined some system-level invariants. The next step is then to specify how the action and the sequence of relevant transactions should be performed to break the defined invariants. 

The high-level contents to explore is in [`LinearBondingCurve.invariants.t.sol`](https://github.com/Ratimon/bonding-curves/blob/master/test/invariant/LinearBondingCurve.invariants.t.sol), and the configuration is at `setup()` as follows:


```solidity

import {InvariantOwner} from "./handlers/Owner.sol";
import {InvariantBuyerManager} from "./handlers/Buyer.sol";
import {Warper} from "./handlers/Warper.sol";

contract LinearBondingCurveInvariants is StdInvariant, Test, ConstantsFixture, DeploymentLinearBondingCurve {
    InvariantOwner internal _owner;
    InvariantBuyerManager internal _buyerManager;
    Warper internal _warper;

    /** ... */

        function setUp() public override {
            /** ... */
            _buyerManager =
                new InvariantBuyerManager(address(linearBondingCurve), address(acceptedToken),  address(saleToken) );
            _warper = new Warper(address(linearBondingCurve));
            _owner = new InvariantOwner(address(linearBondingCurve), address(acceptedToken), address(saleToken), staticTime);

            /** ... */

            vm.startPrank(address(_owner));
            Ownable2Step(address(linearBondingCurve)).acceptOwnership();
            vm.stopPrank();

            vm.startPrank(deployer);
            bytes4[] memory selectors = new bytes4[](1);
            selectors[0] = InvariantBuyerManager.purchase.selector;

            // Performs random purchase() calls
            targetSelector(FuzzSelector({addr: address(_buyerManager), selectors: selectors}));
            targetContract(address(_buyerManager));

            selectors[0] = Warper.warp.selector;
            // Performs random warps forward in time
            targetSelector(FuzzSelector({addr: address(_warper), selectors: selectors}));
            targetContract(address(_warper));

            selectors[0] = InvariantOwner.allocate.selector;
            // Performs random allocate() calls
            targetSelector(FuzzSelector({addr: address(_owner), selectors: selectors}));
            targetContract(address(_owner));

            _buyerManager.createBuyer();
            vm.stopPrank();
        }
    /** ... */
    }
    
```

To sum up, we perform random `purchase()` calls, random `warps` forward in time, and  random `allocate()` calls.

This is set up via **Invariant Test Helper Functions** (including **targetContract(address newTargetedContract_)** and **targetSelector(FuzzSelector memory newTargetedSelector_)** ).

> 💡 Note: More details are outlined in the [`Invariant Test Helper Functions`](https://book.getfoundry.sh/forge/invariant-testing#invariant-test-helper-functions) section of the `foundry documentation`

We can think of **Foundry Fuzzer** as an externally owned account and of **Handler** as a smart contract wrapper, including a set of actions that interact with our target contract.

These handlers are specified in **handler files** located in [`test/invariant/handlers`](https://github.com/Ratimon/bonding-curves/blob/master/test/invariant/handlers) as follows:

1.  [`Buyer.sol`](https://github.com/Ratimon/bonding-curves/blob/master/test/invariant/handlers/Buyer.sol) : perform random `purchase()`

We can think of this set of smart contracts as having external stakeholder properties. In general, we want a fuzzer to generate a number of buyers. Now, we define **InvariantBuyerManager** as follows.

It can be seen that **createBuyer()** is called at our `setup()` so that we have one buyer (**InvariantBuyer**) generated from **InvariantBuyerManager**.

```solidity

contract InvariantBuyerManager is Test {

    /** ... */

    InvariantBuyer[] public buyers;

    /** ... */

    function createBuyer() external {
        InvariantBuyer buyer = new InvariantBuyer(_bondingCurve, _underlyingAcceptedToken, _underlyingSaleToken);
        buyers.push(buyer);
    }

    /** ... */

}
```

Then, those generated buyers (**InvariantBuyer**) are supposed to purchase the token from the bonding curve. Now, we are going to write the **function-level invariants**. 

In particular, we want to ensure that both balance states of external smart contracts (ERC20 tokens in our case) are correctly updated after each purchase. The implement is as follows:


```solidity

/** ... */

contract InvariantBuyer is Test {
    LinearBondingCurve internal _bondingCurve;
    MockERC20 internal _underlyingAcceptedToken;
    MockERC20 internal _underlyingSaleToken;

    constructor(address bondingCurve_, address underlyingBuyToken_, address underlyingSaleToken_) {
        _bondingCurve = LinearBondingCurve(bondingCurve_);
        _underlyingAcceptedToken = MockERC20(underlyingBuyToken_);
        _underlyingSaleToken = MockERC20(underlyingSaleToken_);
    }

    function purchase(uint256 amount_) external {
        amount_ = bound(amount_, 1, 1e29); // 100 billion at WAD precision
        uint256 startingBuyBalance = _underlyingAcceptedToken.balanceOf(address(this));
        uint256 startingSaleBalance = _underlyingSaleToken.balanceOf(address(this));
        uint256 saleAmountOut = unwrap(_bondingCurve.calculatePurchaseAmountOut(ud(amount_)));
        deal({token: address(_underlyingAcceptedToken), to: address(this), give: amount_});
        _underlyingAcceptedToken.approve(address(_bondingCurve), amount_);
        _bondingCurve.purchase(address(this), amount_);
        // Ensure successful purchase
        assertEq(_underlyingAcceptedToken.balanceOf(address(this)), startingBuyBalance - amount_);
        assertEq(_underlyingSaleToken.balanceOf(address(this)), startingSaleBalance + saleAmountOut);
    }
}

/** ... */

```


2.  [`Owner.sol`](https://github.com/Ratimon/bonding-curves/blob/master/test/invariant/handlers/Owner.sol) : perform random `allocate()`

Again, we  ensure that the balance state of an external contract (acceptable tokens ) is correctly updated after the token is allocated to the issuer. The implementation is as follows:

```solidity

/** ... */

contract InvariantOwner is Test {

    /** ... */

    function allocate(uint256 amount_) external countCall("allocate") {
        vm.warp(staticTime + 3 weeks);
        amount_ = bound(amount_, 0, _underlyingAcceptedToken.balanceOf(address(_bondingCurve)));
        uint256 startingBuyBalance = _underlyingAcceptedToken.balanceOf(address(this));
        _bondingCurve.allocate(amount_, address(this));
        assertEq(_underlyingAcceptedToken.balanceOf(address(this)), startingBuyBalance + amount_);
    }

    /** ... */

}

/** ... */

```

3.  [`Warper.sol`](https://github.com/Ratimon/bonding-curves/blob/master/test/invariant/handlers/Warper.sol) : perform random `warps` forward in time.

As this system involves time- dependent logics, the issuer cannot allocate tokens if the selling period has not ended. This means that random `allocate()` would always be reverted without the `warps` handler.

For this, we use `Foundry's cheat code`( **vm.warp(uint256)** ) to deal with it.

```solidity

/** ... */

contract Warper is CommonBase, StdCheats, StdUtils {

    /** ... */

    function warp(uint256 warpTime_) external countCall("warp") {
        vm.warp(block.timestamp + bound(warpTime_, 2 weeks, 3 weeks));
    }

    /** ... */

}

/** ... */

```


### Exploring more !!!

> 💡 Note: We acknowledge, use, and get inspiration from the projects [PaulRBerg/prb-math](https://github.com/PaulRBerg/prb-math) and  [maple-labs/revenue-distribution-token](https://github.com/maple-labs/revenue-distribution-token).

> 💡 Reference: We also acknowledge the insightfuls technial writing: [Invariant Testing WETH With Foundry](https://mirror.xyz/horsefacts.eth/Jex2YVaO65dda6zEyfM_-DXlXhOWCAoSpOx5PLocYgw) and [Invariant Testing — Enter The Matrix](https://betterprogramming.pub/invariant-testing-enter-the-matrix-c71363dea37e)
## `cool`

### Signature

```solidity
function cool(address target) external;
```

### Description

Unmarks the account `target` as touched and cools off all of the `target`'s storage slots. 

Can be used for gas testing purposes by doing required setup in the function and then resetting the slots before doing a `gasleft()` check.

See related EIPs:

- [EIP-2929: Gas cost increases for state access opcodes](https://eips.ethereum.org/EIPS/eip-2929)
  - the `COLD_SLOAD_COST` is `2100` gas.
  - the `COLD_ACCOUNT_ACCESS_COST` is `2600` gas.
  - the `WARM_STORAGE_READ_COST` is `100` gas.
- [EIP-2930: Optional access lists](https://eips.ethereum.org/EIPS/eip-2930)

### Examples

```solidity
contract CoolTest is Test {
    uint256 public slot0 = 1;

    function testCool() public {
        // init uints...

        // read slot0 of this contract
        startGas = gasleft();
        val = slot0;
        endGas = gasleft();
        beforeCoolGas = startGas - endGas; // 2100 gas (COLD_SLOAD_COST)

        // cool off this address
        vm.cool(address(this));

        startGas = gasleft();
        val = slot0;
        endGas = gasleft();
        afterCoolGas = startGas - endGas; // 2100 gas

        // see that the cost of reading slot0 is the same after vm.cool
        assertEq(beforeCoolGas, afterCoolGas);

        startGas = gasleft();
        val = slot0;
        endGas = gasleft();
        noCoolGas = startGas - endGas;

        // and back to warm access without using vm.cool
        assertGt(beforeCoolGas, noCoolGas); // 100 gas (WARM_STORAGE_READ_COST)
    }
}
```

### SEE ALSO

- [`store`](./store.md)
- [`load`](./load.md)

Forge Standard Library

- [Std Storage](../reference/forge-std/std-storage.md)

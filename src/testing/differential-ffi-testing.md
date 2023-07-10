## Differential Testing

Forge can be used for differential testing and differential fuzzing. You can even test against non-EVM executables using the `ffi` [cheatcode](../cheatcodes/ffi.md).

### Background

[Differential testing](https://en.wikipedia.org/wiki/Differential_testing) cross references multiple implementations of the same function by comparing each one's output. Imagine we have a function specification `F(X)`, and two implementations of that specification: `f1(X)` and `f2(X)`. We expect `f1(x) == f2(x)` for all x that exist in an appropriate input space. If `f1(x) != f2(x)`, we know that at least one function is incorrectly implementing `F(X)`. This process of testing for equality and identifying discrepancies is the core of differential testing.

Differential fuzzing is an extension of differential testing. Differential fuzzing programatically generates many values of `x` to find discrepancies and edge cases that manually chosen inputs might not reveal.

> Note: the `==` operator in this case can be a custom definition of equality. For example, if testing floating point implementations, you might use approximate equality with a certain tolerance.

Some real life uses of this type of testing include:

- Comparing upgraded implementations to their predecessors
- Testing code against known reference implementations
- Confirming compatibility with third party tools and dependencies

Below are some examples of how Forge is used for differential testing.

### Primer: The `ffi` cheatcode

[`ffi`](../cheatcodes/ffi.md) allows you to execute an arbitrary shell command and capture the output. Here's a mock example:

```solidity
import "forge-std/Test.sol";

contract TestContract is Test {

    function testMyFFI () public {
        string[] memory cmds = new string[](2);
        cmds[0] = "cat";
        cmds[1] = "address.txt"; // assume contains abi-encoded address.
        bytes memory result = vm.ffi(cmds);
        address loadedAddress = abi.decode(result, (address));
        // Do something with the address
        // ...
    }
}
```

An address has previously been written to `address.txt`, and we read it in using the FFI cheatcode. This data can now be used throughout your test contract.

### Example: Differential Testing Merkle Tree Implementations

[Merkle Trees](https://en.wikipedia.org/wiki/Merkle_tree) are a cryptographic commitment scheme frequently used in blockchain applications. Their popularity has led to a number of different implementations of Merkle Tree generators, provers, and verifiers. Merkle roots and proofs are often generated using a language like JavaScript or Python, while proof verification usually occurs on-chain in Solidity.

[Murky](https://github.com/dmfxyz/murky) is a complete implementation of Merkle roots, proofs, and verification in Solidity. Its test suite includes differential tests against OpenZeppelin's Merkle proof library, as well as root generation tests against a reference JavaScript implementation. These tests are powered by Foundry's fuzzing and `ffi` capabilities.

#### Differential fuzzing against a reference TypeScript implementation

Using the `ffi` cheatcode, Murky tests its own Merkle root implementation against a TypeScript implementation using data provided by Forge's fuzzer:

```solidity
function testMerkleRootMatchesJSImplementationFuzzed(bytes32[] memory leaves) public {
    vm.assume(leaves.length > 1);
    bytes memory packed = abi.encodePacked(leaves);
    string[] memory runJsInputs = new string[](8);

    // Build ffi command string
    runJsInputs[0] = 'npm';
    runJsInputs[1] = '--prefix';
    runJsInputs[2] = 'differential_testing/scripts/';
    runJsInputs[3] = '--silent';
    runJsInputs[4] = 'run';
    runJsInputs[5] = 'generate-root-cli';
    runJsInputs[6] = leaves.length.toString();
    runJsInputs[7] = packed.toHexString();

    // Run command and capture output
    bytes memory jsResult = vm.ffi(runJsInputs);
    bytes32 jsGeneratedRoot = abi.decode(jsResult, (bytes32));

    // Calculate root using Murky
    bytes32 murkyGeneratedRoot = m.getRoot(leaves);
    assertEq(murkyGeneratedRoot, jsGeneratedRoot);
}
```

> Note: see [`Strings2.sol`](https://github.com/dmfxyz/murky/blob/main/differential_testing/test/utils/Strings2.sol) in the Murky Repo for the library that enables `(bytes memory).toHexString()`

Forge runs `npm --prefix differential_testing/scripts/ --silent run generate-root-cli {numLeaves} {hexEncodedLeaves}`. This calculates the Merkle root for the input data using the reference JavaScript implementation. The script prints the root to stdout, and that printout is captured as `bytes` in the return value of `vm.ffi()`.

The test then calculates the root using the Solidity implementation.

Finally, the test asserts that the both roots are exactly equal. If they are not equal, the test fails.

#### Differential fuzzing against OpenZeppelin's Merkle Proof Library

You may want to use differential testing against another Solidity implementation. In that case, `ffi` is not needed. Instead, the reference implementation is imported directly into the test.

```solidity
import "openzeppelin-contracts/contracts/utils/cryptography/MerkleProof.sol";
//...
function testCompatabilityOpenZeppelinProver(bytes32[] memory _data, uint256 node) public {
    vm.assume(_data.length > 1);
    vm.assume(node < _data.length);
    bytes32 root = m.getRoot(_data);
    bytes32[] memory proof = m.getProof(_data, node);
    bytes32 valueToProve = _data[node];
    bool murkyVerified = m.verifyProof(root, proof, valueToProve);
    bool ozVerified = MerkleProof.verify(proof, root, valueToProve);
    assertTrue(murkyVerified == ozVerified);
}
```

#### Differential testing against a known edge case

Differential tests are not always fuzzed -- they are also useful for testing known edge cases. In the case of the Murky codebase, the initial implementation of the `log2ceil` function did not work for certain arrays whose lengths were close to a power of 2 (like 129). As a safety check, a test is always run against an array of this length and compared to the TypeScript implementation. You can see the full test [here](https://github.com/dmfxyz/murky/blob/main/differential_testing/test/DifferentialTests.t.sol#L21).

#### Standardized Testing against reference data

FFI is also useful for injecting reproducible, standardized data into the testing environment. In the Murky library, this is used as a benchmark for gas snapshotting (see [forge snapshot](./gas-snapshots.md)).

```solidity
bytes32[100] data;
uint256[8] leaves = [4, 8, 15, 16, 23, 42, 69, 88];

function setUp() public {
    string[] memory inputs = new string[](2);
    inputs[0] = "cat";
    inputs[1] = "src/test/standard_data/StandardInput.txt";
    bytes memory result =  vm.ffi(inputs);
    data = abi.decode(result, (bytes32[100]));
    m = new Merkle();
}

function testMerkleGenerateProofStandard() public view {
    bytes32[] memory _data = _getData();
    for (uint i = 0; i < leaves.length; ++i) {
        m.getProof(_data, leaves[i]);
    }
}
```

`src/test/standard_data/StandardInput.txt` is a text file that contains an encoded `bytes32[100]` array. It's generated outside of the test and can be used in any language's Web3 SDK. It looks something like:

```ignore
0xf910ccaa307836354233316666386231414464306335333243453944383735313..423532
```

The standardized testing contract reads in the file using `ffi`. It decodes the data into an array and then, in this example, generates proofs for 8 different leaves. Because the data is constant and standard, we can meaningfully measure gas and performance improvements using this test.

> Of course, one could just hardcode the array into the test! But that makes it much harder to do consistent testing across contracts, implementations, etc.

### Example: Differential Testing Gradual Dutch Auctions

The reference implementation for Paradigm's [Gradual Dutch Auction](https://www.paradigm.xyz/2022/04/gda) mechanism contains a number of differential, fuzzed tests. It is an excellent repository to further explore differential testing using `ffi`.

- Differential tests for [Discrete GDAs](https://github.com/FrankieIsLost/gradual-dutch-auction/blob/master/src/test/DiscreteGDA.t.sol#L78)
- Differential tests for [Continuous GDAs](https://github.com/FrankieIsLost/gradual-dutch-auction/blob/master/src/test/ContinuousGDA.t.sol#L89)
- Reference [Python implementation](https://github.com/FrankieIsLost/gradual-dutch-auction/blob/master/analysis/compute_price.py)

### Reference Repositories

- [Gradual Dutch Auctions](https://github.com/FrankieIsLost/gradual-dutch-auction)
- [Murky](https://www.github.com/dmfxyz/murky)
- [Solidity Fuzzing Template](https://github.com/patrickd-/solidity-fuzzing-boilerplate)

If you have another repository that would serve as a reference, please contribute it!

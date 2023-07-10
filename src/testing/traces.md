## Understanding Traces

Forge can produce traces either for failing tests (`-vvv`) or all tests (`-vvvv`).

Traces follow the same general format:

```ignore
  [<Gas Usage>] <Contract>::<Function>(<Parameters>)
    ├─ [<Gas Usage>] <Contract>::<Function>(<Parameters>)
    │   └─ ← <Return Value>
    └─ ← <Return Value>
```

Each trace can have many more subtraces, each denoting a call to a contract and a return value.

If your terminal supports color, the traces will also come with a variety of colors:

- **Green**: For calls that do not revert
- **Red**: For reverting calls
- **Blue**: For calls to cheat codes
- **Cyan**: For emitted logs
- **Yellow**: For contract deployments

The gas usage (marked in square brackets) is for the entirety of the function call. You may notice, however, that sometimes the gas usage of one trace does not exactly match the gas usage of all its subtraces:

```ignore
  [24661] OwnerUpOnlyTest::testIncrementAsOwner()
    ├─ [2262] OwnerUpOnly::count()
    │   └─ ← 0
    ├─ [20398] OwnerUpOnly::increment()
    │   └─ ← ()
    ├─ [262] OwnerUpOnly::count()
    │   └─ ← 1
    └─ ← ()
```

The gas unaccounted for is due to some extra operations happening between calls, such as arithmetic and store reads/writes.

Forge will try to decode as many signatures and values as possible, but sometimes this is not possible. In these cases, the traces will appear like so:

```ignore
  [<Gas Usage>] <Address>::<Calldata>
    └─ ← <Return Data>
```

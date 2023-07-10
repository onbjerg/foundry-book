## Tests

Testing on Foundry is done using `forge`. Here we'll learn how to perform basic and more advanced testing and to leverage unique functionality such as cheatcodes.

Forge can run your tests with the [`forge test`](../reference/forge/forge-test.md) command. All tests are written in Solidity.

Forge will look for the tests anywhere in your source directory. Any contract with a function that starts with `test` is considered to be a test. Usually, tests will be placed in `test/` by convention and end with `.t.sol`.

Here's an example of running `forge test` in a freshly created project, that only has the default test:

```sh
{{#include ../output/hello_foundry/forge-test:all}}
```

You can also run specific tests by passing a filter:

```sh
{{#include ../output/test_filters/forge-test-match-contract-and-test:all}}
```

This will run the tests in the `ComplicatedContractTest` test contract with `testDeposit` in the name.
Inverse versions of these flags also exist (`--no-match-contract` and `--no-match-test`).

You can run tests in filenames that match a glob pattern with `--match-path`.

```sh
{{#include ../output/test_filters/forge-test-match-path:all}}
```

The inverse of the `--match-path` flag is `--no-match-path`.

### Logs and traces

The default behavior for `forge test` is to only display a summary of passing and failing tests. You can control this behavior by increasing the verbosity (using the `-v` flag). Each level of verbosity adds more information:

- **Level 2 (`-vv`)**: Logs emitted during tests are also displayed. That includes assertion errors from tests, showing information such as expected vs actual.
- **Level 3 (`-vvv`)**: Stack traces for failing tests are also displayed.
- **Level 4 (`-vvvv`)**: Stack traces for all tests are displayed, and setup traces for failing tests are displayed.
- **Level 5 (`-vvvvv`)**: Stack traces and setup traces are always displayed.

### Watch mode

Forge can re-run your tests when you make changes to your files using `forge test --watch`.

By default, only changed test files are re-run. If you want to re-run all tests on a change, you can use `forge test --watch --run-all`.

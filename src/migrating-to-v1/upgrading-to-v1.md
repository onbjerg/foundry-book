## Upgrading to Foundry V1

### Local Installation

To upgrade, you'll need to reinstall `foundryup`. To do so, open your terminal and run the following command:

```sh
curl -L https://foundry.paradigm.xyz | bash
```

This will reinstall `foundryup`. You can then run `foundryup` on your terminal to download the latest `V1` version.

After this, you're now up to date with the latest stable Foundry version.

### Upgrading your CI

When it comes to CI, you can add a `version` input field with `v1.0.0` or your desired stable release. This will pin the CI to that stable version.

If you do not update the CI, it will stay on nightly.
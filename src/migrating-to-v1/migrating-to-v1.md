## Migrating to Foundry v1

Foundry v1 is the blazing-fast smart contract testing framework you already know, but finally marked as stable and respecting [semver](https://semver.org/). It's the culmination of over a year of work and more than 250 contributors that made it happen.

With the v1 release, very few breaking changes have been included, meaning migration should be mostly painless. This guide will help you understand what these changes are and how to handle them properly.

### What to expect

v1 mainly increases the strictness of several cheatcodes, and accommodating this is the primary change required to migrate to v1. We recommend migrating to v1 as soon as possible, as the stricter cheats may reveal bugs or issues in your code and test suite. 

If migration is time consuming, you can consider upgrading your local foundry installation while migrating tests, but maintaining nightly usage on CI to avoid any failures that might disrupt your workflow.

Aside from this, v1 is still the same Foundry you know.

### I need support. Where do I go?

The [dev Telegram](https://t.me/foundry_rs) is available for general conversation around Foundry and its features, while the [support Telegram](https://t.me/foundry_support) is available for any concerns you may have that are not covered in this guide.
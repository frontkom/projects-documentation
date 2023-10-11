# Ny Media modules update procedure

The following document briefly describes what steps you should take, if you want to make sure
an internal Ny Media module is compatible with Drupal 10.

## Step 1. Verify module's tests workflow is up-to-date

Every module we have SHOULD contain a GitHub workflow configured (e.g. [commerce_google_shopping](https://github.com/nymedia/commerce_google_shopping/tree/8.x-2.x/.github)) that fires phpcs, phpstan
and unit tests for specified PHP and Drupal core version. The workflow should be up-to-date with the one in this repo https://github.com/nymedia/github-actions-run-test-module. It fires tests
against PHP 8.2 and D10 version.

We have a separate workflow - [Merge upstream to downstream](https://github.com/nymedia/github-actions-sync-upstream/actions/workflows/merge-upstream.yml)
that ensures us the module's workflow is up-to-date. If you have permissions to run it follow the steps below. In the brackets, there are input values valid for the moment of writing this doc.
1. Click "Run workflow" dropdown
2. In `upstream-repository` field put the name of the "source" repo containing the workflow config we need to update with (`nymedia/github-actions-run-test-module`)
3. In `upstream-branch` field put the name of the main branch on the upstream-repository (`1.x`)
4. In `downstream-repository` field put the name of the module's repo containing the outdated workflow config (for example `nymedia/commerce_google_shopping`)
5. In `downstream-branch` field put the name of the main branch on the module's repo from pt. 3. (in case of commerce_google_shipping it would be `8.x-2.x`)
6. Click "Run workflow" button and see if it succeeds. If it does, you should have a PR assigned on the module's repo, [looking like this](https://github.com/nymedia/commerce_google_shopping/pull/51)
7. **NEVER EVER SQUASH THE COMMITS IN THE PR.**

## Step 2. Update the module's code to make sure the tests succeed

Once you have the module with workflow up-to-date, and you see te tests failing, make sure they won't.
Most common things to check:
- make sure we're not having any deprecations in code
- make sure in composer.json we don't have a dependency requiring PHP 7.3
- if there's `drush/drush` dependency in composer.json, make sure we use allow also "^11.0 || ^12.0" versions
- make sure we allow Drupal ^10 in the .info.yml file
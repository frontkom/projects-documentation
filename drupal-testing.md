# Drupal standard tests

## Introduction

Almost every Drupal project in Frontkom follows these patterns. If you observe a project deviating from it, you should try to make sure the information here will be directly relevant to the project by adjusting the setup so it's correct

## Types of tests.

We usually have 3 (or 4) types of tests running in projects. It will be 3 if the project is not running phpunit tests in a separate job, and 4 if it does. Here are their names descriptions:

### Static tests

This will run the command `composer test-static`. This command will usually consist of the following (or variations of it):

```
"test-static": [
    "@composer install",
    "@composer validate --no-check-all --no-check-publish",
    "@composer phpcs",
    "@composer phpstan",
    "@composer phpunit",
],
```

In other words. All different kinds of static analysis, code style and unit tests. Here are some more detailed explanation of the different parts:

#### composer install
This installs the dependencies, including test dependencies, since the following parts of the command will require test dependencies (`require-dev` or `require --dev`) to be installed.

#### composer validate --no-check-all --no-check-publish
This validates the `composer.json` file. Most importantly it will fail if the lock file is not up to date. See [common test errors and how to fix them](https://github.com/frontkom/projects-documentation/blob/1.x/common-test-failures.md#exit-code-2-lock-file-is-not-up-to-date) for how to fix this, if it fails.

#### composer phpcs
This will usually invoke `phpcs`, with the following command: `./vendor/bin/phpcs -p -n`. Checks the coding style according to Drupal coding standards.

#### composer phpstan
This will usually run `phpstan`, with the following command: `./vendor/bin/phpstan analyse`. Runs static analysis on the codebase.

#### composer phpunit
This will run phpunit. If the project has a separate PHPUnit job, this will probably not be run as part of the Static tests, but instead be run in the PHPUnit job. See below.

### Functional tests
This will run the command `composer test`. This command will usually consist of the following (or variations of it):

```
"test": [
    "@composer install",
    "./vendor/bin/behat --colors --strict"
],
```

In practice, this will run behat. See separate behat guide for more info.

### Install site from scratch
This tests that the project can be installed from scratch. This is important, so new developers can be onboarded with as little friction as possible.

In practice, this test runs the command `composer site-install`. What exactly is in there will differ a bit, but usuall some variation of:

- Install dependencies with composer
- Install Drupal with `drush site:install --existing-config`
- Import translations
- Import default content

### PHPUnit
This test is at the time of writing opt-in. Meaning that some sites run PHPUnit as part of `composer test-static`, and some run it as a separate job. The reason we some times have to run it as a separate job, is to be able to write tests with [Drupal Test Traits](https://git.drupalcode.org/project/dtt/-/blob/2.x/README.md). This will require a fully installed project.

See separate phpunit guide for more info.

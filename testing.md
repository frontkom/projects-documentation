# Standard Frontkom test setup

In most projects the testing is divided into two parts: Static tests and Functional tests

## Static tests

In most projects you should be able to run all static tests with `composer test-static`. If you want to run one of them instead, follow the instructions below:

### phpstan

### phpcs

### phpunit

## Functional tests

In most projects running `composer test` will run the functional tests. You can also run it more specifically by reading more info below:

### behat

#### Running all tests with behat

```
$ ./vendor/bin/behat --strict
```

### Running a specific test with behat

For example a test called `page.feature`.

```
$ ./vendor/bin/behat tests/features/content/page.feature
```

You can also run a test with a specific tag:

```
$ ./vendor/bin/behat --tags=my-tag
```

# Running tests as pre-commit hooks

If you want to run some of the tests as git hooks to avoid comitting "bad" code, you can do that. It would probably make most sense to only run static tests.

Usually static tests are run with the command `composer test-static`. To run this as a git hook you can simply add this file to a file called `.git/hooks/pre-commit` in your project:

```sh
#!/bin/sh

composer test-static
```

If you think that takes too long, and you just want to run code style checks with phpcs, you can instead change it to something like this:

```sh
#!/bin/sh

vendor/bin/phpcs -pn
```

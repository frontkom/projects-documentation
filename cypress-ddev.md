# Running cypress tests locally (with ddev)

## Running tests

With the project, it should already exist a command to run the cypress tests. It should be `ddev cypress-run`. This command comes from the [DDEV-cypress add-on](https://github.com/tyler36/ddev-cypress)

If you want to watch the tests run in your browser, you could simply do `ddev cypress-run --headed`

## Writing and debugging tests

There should also be a command for opening cypress and executing single tests, and debugging them. This corresponds to `cypress open` which has some more information here: https://docs.cypress.io/app/references/command-line#cypress-open

In ddev this should be `ddev cypress-open`. Here you can browse the tests, and expand or debug them like you want.

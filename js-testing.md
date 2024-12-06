# Standard workflow for JS projects

The default JS workflow consists of 4 types of tests, which are described below

## JS Linting

This will run linting for the project, most commonly eslint. In practice, the workflow will run the npm script `lint`.

To run this locally you would typically do something like this:

```bash
npm run lint
```

Or if your project is set up with ddev for local development:

```bash
ddev npm run lint
```

## Type check

This will run type checks for the project, usually typescript. In practice, the workflow will run the npm script `type-check`

To run this locally you would typically do something like this:

```bash
npm run type-check
```

Or if your project is set up with ddev for local development:

```bash
ddev npm run type-check
```

## Unit tests

This will run unit tests for the project. For example `jest`. In practice, the workflow will run the npm script `unit`

To run this locally you would typically do something like this:

```bash
npm run unit
```

Or if your project is set up with ddev for local development:

```bash
ddev npm run unit
```

## E2E tests

This will run unit tests for the project. For example `cypress`.

There are currently 2 approaches for this. Either the project will be set up to run the npm script `e2e`. Or it will be set up to use the [GitHub action for Cypress](https://github.com/cypress-io/github-action).

If the project uses the npm script `e2e` you can run this locally with something like this:

```bash
npm run e2e
```

Or if your project is set up with ddev for local development:

```bash
ddev npm run e2e
```

If the project uses the cypress github action you can run this locally with something like this:

```bash
npx cypress run
```

If you project is set up with ddev for local development, please see [the docs for cypress and ddev](https://github.com/frontkom/projects-documentation/blob/1.x/cypress-ddev.md)

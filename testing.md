# Standard Frontkom test setup

In most projects the testing is divided into two parts: Static tests and Functional tests

## Static tests

In most projects you should be able to run all static tests with `composer test-static`. 


### Adding static tests to an existing project

Example projects with static tests implemented: 
- [frontkom/kf-cms](https://github.com/frontkom/kf-cms)
- [frontkom/nord](https://github.com/frontkom/nord)
- [frontkom/fagskolen-viken](https://github.com/frontkom/fagskolen-viken)
- [frontkom/simula-backend](https://github.com/frontkom/simula-backend)

#### 1. Add a run-tests.yml file 

Add the following content to `/.github/workflows/run-tests.yml` within your project: 

```
name: Test

on:
  push:
    branches:
      - develop
      - main
  pull_request:
    branches:
      - develop
      - main

jobs:
  drupal-run-tests:
    name: ${{ github.event_name == 'pull_request' && 'PR Validation' || 'Deployment' }}
    uses: Frontkom-DevOps/shared-workflows-drupal/.github/workflows/run-tests.yml@2.x
    secrets: inherit
    permissions:
      packages: read
      contents: write
    with:
      behat_requirements: ''
      ci_settings_file_path: ''
      skip_functional_tests: true
```

NOTE: For non-standard projects you may need to adjust the branch names.

If the project does not yet have any specific functional tests include the `skip_functional_tests: true` otherwise it can be removed.

This test will run on all pull requests and deployments on the branches defined in the .yml file.

The test uses the [standard run-tests.yml workflow](https://github.com/Frontkom-DevOps/shared-workflows-drupal/blob/2.x/.github/workflows/run-tests.yml) which in turn inherits other standard workflows into the test.

#### 2. Update composer.json

As part of the [reusable-test.yml](https://github.com/Frontkom-DevOps/shared-workflows-drupal/blob/2.x/.github/workflows/reusable-test.yml) workflow 2 Composer scripts: `site-install` and `test-static` within the local project are called so these needs to be setup and functional within the local projects `composer.json` file within the `scripts` objects:

`site-install` scripts:
```
"si": "@composer site-install",
"site-install": [
  "@composer install",
  "./vendor/bin/drush --root=$(pwd)/web site:install minimal ${DB_PARAMS} --existing-config --yes",
  "@composer import",
  "./vendor/bin/drush --root=$(pwd)/web cache:rebuild",
  "@composer set-up-default-content"
],
"set-up-default-content": [
  "./vendor/bin/drush default-content-deploy:import --force-override --yes"
],
"import": [
  "./vendor/bin/drush --root=$(pwd)/web cache:clear drush",
  "./vendor/bin/drush --root=$(pwd)/web config:import --yes"
],
```

`test-static` scripts:
```
"phpcs": [
  "./vendor/bin/phpcs -p -n"
],
"phpunit": [
  "./vendor/bin/phpunit"
],
"phpstan": [
  "./vendor/bin/phpstan analyse"
],
"test-static": [
  "@composer install",
  "@composer validate --no-check-all --no-check-publish",
  "@composer phpcs",
  "@composer phpstan",
  "@composer phpunit"
]
```

Examples on live projects: 
- [frontkom/kf-cms](https://github.com/frontkom/kf-cms/blob/develop/composer.json#L95)
- [frontkom/nord](https://github.com/frontkom/nord/blob/develop/composer.json#L180)
- [frontkom/fagskolen-viken](https://github.com/frontkom/fagskolen-viken/blob/develop/composer.json#L249)

The `drupal/core-dev` package either directly or indirectly requires the `phpcs`, `phpunit` and `phpstan` packages so they should all be available within the project as vendor packages.

For the `site-install` test you will likely need to change the projects installer profile from `standard` to `minimal` using a `hook_update_X` within a `.install` file in addition to updating the config `core.extension.yml`. This is a one-time change and **should be monitored with caution when deploying as it had the potential to break things**.

`hook_update_X` example: 

```
/**
 * Change the profile to minimal and install the minimal theme.
 */
function MODULE_update_9001() {
  // The minimal profile has some config objects that depend on stark. To be
  // able to install the profile, we need to enable stark as well. This is fine
  // though, since afterwards, we will import the config, and the enabling of
  // the theme will be reverted automatically.
  \Drupal::service('theme_installer')->install(['stark']);

  $old = 'standard';
  $new = 'minimal';
  \Drupal::configFactory()->getEditable('core.extension')
    ->set('profile', $new)
    ->save();
  drupal_flush_all_caches();

  \Drupal::service('module_installer')->install([$new]);
  \Drupal::service('module_installer')->uninstall([$old]);

  $sc = \Drupal::keyValue('system.schema');
  $weight = 8000;
  if ($weight = $sc->get($old)) {
    $sc->delete($old);
  }
  $sc->set($new, $weight);
  drupal_flush_all_caches();
}
```

`core.extension.yml` example: 

```diff
module:
   .......
   monolog: 0
   monolog_elasticsearch_date_processor: 0
   paragraphs: 11
   tvi: 15
   better_formats: 100
-  standard: 1000
+  minimal: 1000
theme:
   .......
   gin: 0
   stable9: 0
-profile: standard
+profile: minimal
```

To test `site-install` locally, backup your working database, then run:

```
ddev ssh
drush updatedb
composer run site-install
```

#### 3. `phpcs`, `phpunit` and `phpstan` config files

Add the following files to your project root:

**phpcs.xml.dist**:

In Terminal in the codebase root directory run `touch phpcs.xml.dist` and add the following content:

```
<?xml version="1.0"?>
<ruleset name="frontkom">
    <description>The coding standard for Frontkom projects.</description>
    <arg name="extensions" value="inc,install,module,php,profile,test,theme"/>

    <file>./web/modules/custom</file>
    <file>./web/themes/custom</file>

    <rule ref="./vendor/drupal/coder/coder_sniffer/Drupal"/>
</ruleset>
```

This tells `phpcs` what files should be reviewed for coding standards and to use the `drupal/coder/coder_sniffer/Drupal` rules for the coding standards. This should include all custom code only.

[View all Code Sniffer properties](https://github.com/squizlabs/PHP_CodeSniffer/wiki/Customisable-Sniff-Properties).

[How to automatically fix issues with Code Sniffer](https://github.com/squizlabs/PHP_CodeSniffer/wiki/Fixing-Errors-Automatically#using-the-php-code-beautifier-and-fixer).

**phpstan.neon**

First you should attempt to create a baseline file by running the following from within the DDEV container:
```
ddev ssh
./vendor/bin/phpstan analyse --generate-baseline
```
A `phpstan-baseline.neon` file serves as a record of existing errors or warnings in the project at the time the baseline is created so they can be ignored by subsequent analysis. It allows you to suppress pre-existing issues while continuing to analyze new code and catch future errors. The goal is to have this free of errors and warnings. 
If `phpstan-baseline.neon` contains errors or warnings it should be committed to git for use with remote tests on pull requests and deployments.


In Terminal in the codebase root directory run `touch phpstan.neon` and add the following content:

```
parameters:
    level: 2
    paths:
    - web/modules/custom
    - web/themes/custom

includes:
  - phpstan-baseline.neon    
```

This is similar to `phpcs.xml.dist` in that it tells PHPstan what files should be reviewed. 

If no baseline file was generated you can remove the `includes` section of the file.

**phpunit.xml.dist**

In Terminal in the codebase root directory run `touch phpunit.xml.dist` and add the following content:

```
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="http://schema.phpunit.de/4.1/phpunit.xsd"
    backupGlobals="false"
    colors="true"
    bootstrap="vendor/autoload.php"
    verbose="true"
    >
    <testsuites>
        <testsuite name="drupal-composer-project tests">
            <directory>./tests/</directory>
        </testsuite>
    </testsuites>
</phpunit>
```

This tells `phpunit` where to find project specific PHPUnit tests. Ensure the directory exists in the root. @TODO - expand


#### 4. Running static tests

With the configurations in place your `test-static` script should now be functional and you can begin to run static tests on your codebase from within the DDEV container: 

```
ddev ssh
composer test-static
```

If you want to run them individually on your local development environment, enter the DDEV container `ddev ssh` then use the commands below:

***phpstan:*** `composer run phpstan`

***phpcs:*** `composer run phpcs`

***phpunit:*** `composer run phpunit`


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

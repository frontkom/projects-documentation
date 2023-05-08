# Upgrading nymedia projects to PHP 8.1

## Getting the environment ready

- Make sure the project has `drupal/core` version ^9.3, because of [core php requirements](https://www.drupal.org/docs/getting-started/system-requirements/php-requirements) 
- Make sure the project has `elasticsearch/elasticsearch` version ^7, if it uses ES at all
- Make sure the project uses `"phpspec/prophecy-phpunit": "^2",` and `"phpunit/phpunit": "^9",`
- If you didn't update `nymedia/store8-tests` to the newest version, that's the best time to do it. Otherwise you won't know if a bug was introduced by PHP 8.1 upgrade or not.

## Update the docker environment

The following changes should be applied to .env.example file in the project repository and your local .env file
```diff
--- a/.env.example
+++ b/.env.example
@@ -25,7 +25,7 @@ MYSQL_USER=root
 
 DOCROOT=/var/www/html/drupal
 PHP_SERVICE_NAME=php
-PHP_VERSION=7.4
+PHP_VERSION=8.1
```


## Update the composer dependencies

Most of the Ny Media modules are prepared for PHP 8, so "just" update the php version as mentioned below
and update all the dependencies.
```diff
--- a/composer.json
+++ b/composer.json
@@ -194,7 +195,7 @@
         "process-timeout": 2000,
         "sort-packages": true,
         "platform": {
-            "php": "7.4"
+            "php": "8.1"
```

Then, the most important things:
- Run database updates (if any)
- Export config (if it differs)
- Export site-schema (`composer site-schema`)

## Update php version in workflow

Make sure the Github Actions also use PHP 8.1:
```diff
--- a/.github/workflows/run-tests.yml
+++ b/.github/workflows/run-tests.yml
@@ -16,7 +16,7 @@ jobs:
     runs-on: ubuntu-latest
     timeout-minutes: 1
     env:
-      PHP_VERSION: 7.4
+      PHP_VERSION: 8.1
       COMPOSER_VERSION: 2
       DOCKER_CLI_EXPERIMENTAL: enabled
     outputs:
```


## Common issues

### Behat tests are failing because of `Deprecated function: pathinfo() on template_preprocess_image_style`

There's issue for it in d.o, most likely you can fix it by applying this patch to drupal/core
```
"Deprecated function: pathinfo() on template_preprocess_image_style (https://www.drupal.org/project/drupal/issues/3306131)": "https://git.drupalcode.org/project/drupal/-/merge_requests/2680/diffs.patch"
```


### In Github Actions the behat test fails, but I cannot see logs

There's issue for it, so expect this issue to be fixed very soon: https://github.com/nymedia/store8-tests/issues/75

For now, our current workaround is to override the logDumpAfterStep in the project's FeatureContext:

```php
  /**
   * Prints from watchdog at failed step.
   *
   * @AfterStep
   */
  public function logDumpAfterStep(AfterStepScope $scope) {
    $failed = (99 === $scope->getTestResult()->getResultCode());
    if ($failed) {
      $query = \Drupal::database()
        ->select('watchdog', 'w');

      $query
        ->range(0, 30)
        ->fields('w')
        ->orderBy('wid', 'DESC');
      $rsc = $query->execute();
      $table = [];
      while ($result = $rsc->fetchObject()) {
        $table[$result->wid] = (array) $result;
      }
      print_r($table);
    }
  }

```


### In Github Actions, workflow fails on `composer install --prefer-dist --no-interaction`

If you see the error in any workflow job:
```
Error: Failed to execute git show-ref --head -d

fatal: detected dubious ownership in repository at '/__w/center365/center365/drupal/modules/contrib/elasticsearch_helper'
To add an exception for this directory, call:

	git config --global --add safe.directory /__w/center365/center365/drupal/modules/contrib/elasticsearch_helper
```

you should try to modify the key in checkout action in the workflow, like this:

```diff
--- a/.github/workflows/run-tests.yml
+++ b/.github/workflows/run-tests.yml
@@ -85,9 +85,9 @@ jobs:
             ./drupal/modules/nymedia
             ./drupal/themes/contrib
             ./drupal/libraries
-          key: composer-${{ hashFiles('**/composer.lock') }}
+          key: composer-d9-${{ hashFiles('**/composer.lock') }}
           restore-keys: |
-            composer-
+            composer-d9-
       - name: Install dependencies
         run: composer install --prefer-dist --no-interaction
       # TODO: Split the following command into multiple steps and explicitly call each test suite (composer validate, phpcs, phpunit, etc)

```
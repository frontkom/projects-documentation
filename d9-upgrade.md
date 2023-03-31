# Upgrading nymedia projects to Drupal 9

## Getting the environment ready

- Make sure the project is using composer 2
- Make sure the project is using drush 10
- Make sure the project has at least version ^1.0 of all phpstan related modules (`mglaman/phpstan-drupal`, `phpstan/phpstan-deprecation-rules` and `phpstan/phpstan-phpunit`
- Make sure the project has upgrade status enabled
- Make sure the project has `drupal/drupal-extension` version 4 or higher.

When updating `drupal/drupal-extension` make sure the following files is also updated:

`.github/workflows/run-tests.yml`
```diff
        IS_CI_ENVIRONMENT: 1
        COMPOSER_MEMORY_LIMIT: -1
        COMPOSER_ALLOW_SUPERUSER: 1
-       BEHAT_PARAMS: "{\"extensions\": {\"Behat\\\\MinkExtension\":{\"base_url\": \"http://web\"}, \"Drupal\\\\DrupalExtension\": {\"drupal\": {\"drupal_root\": \"/var/www/html/drupal\"}}}}"
+       BEHAT_PARAMS: "{\"extensions\": {\"Drupal\\\\MinkExtension\":{\"base_url\": \"http://web\"}, \"Drupal\\\\DrupalExtension\": {\"drupal\": {\"drupal_root\": \"/var/www/html/drupal\"}}}}"
    # Those are project (phpfpm, nginx, db, elasticsearch) and test (chromedriver) dependencies.
    services:
      php:
```
`behat.yml.dist`

```diff
        - Nymedia\Tests\Context\FeatureContext
        - Nymedia\Tests\Context\ProductContext
  extensions:
-   Behat\MinkExtension:
+   Drupal\MinkExtension:
      files_path: "%paths.base%/tests/dummy_files"
      goutte: ~
      selenium2:
```


## Changing new required settings

You can start by going to `admin/reports/upgrade-status` where this will also be listed. Do the following change to a _committed_ settings.php file, which probably in most project means `drupal/sites/default/settings.php`:

```diff
--- a/drupal/sites/default/settings.php
+++ b/drupal/sites/default/settings.php
@@ -790,5 +790,5 @@
 if (file_exists($app_root . '/' . $site_path . '/settings.local.php')) {
   include $app_root . '/' . $site_path . '/settings.local.php';
 }
-$config_directories['sync'] = '../config/common';
+$settings['config_sync_directory'] = '../config/common';
```

## Upgrade all of the contrib modules

It is extremely important to upgrade all of the contrib modules following a specific pattern:

- Upgrade the module with composer
- Run database updates (if any)
- Export config (if it differs)

To make this as easy as possible, you can use this one-liner that runs all of those commands and asks for the module name:

``` 
clear && echo "What is the machine name of the contrib module (i.e commerce, or address)?" && read -r MODULE_NAME && composer update drupal/$MODULE_NAME && drush updb -y && composer export
``` 

Some modules would require you to also update them with dependencies. In which case you would just append that at the right place in the one-liner:

```
clear && echo "What is the machine name of the contrib module (i.e commerce, or address)?" && read -r MODULE_NAME && composer update drupal/$MODULE_NAME --with-dependencies && drush updb -y && composer export
``` 

Proceed to upgrade each and every one of them, until there are no incomatible contrib modules on the upgrade status page.

If the project is set to use a site schema, remember to commit site schema files along with composer and config changes.

## Upgrade Drupal core

First we do this:

```
composer require 'drupal/core-recommended:^9' 'drupal/core-composer-scaffold:^9' --update-with-dependencies --no-update
``` 

Then, hopefully, if we are lucky. This should now work:

```
composer update "drupal/core*" "symfony/*" composer/installers drush/drush --with-dependencies
```

If it does not work, use this command to try to troubleshoot:

```
# Replace 9.5.1 with whatever is the latest drupal core.
composer why-not drupal/core 9.5.1
```

## Common errors and how to solve them

### Problem with patch from `nymedia_profile`:

If your project crashes because it can not apply the patch we use for settings.php. This one specifically:

```
"Keep setting.php intact, add local config to settings.local.php": "https://github.com/zaporylie/drupal/commit/6f0b4e95f28ae945c6fa2bf0e5e81850d3e695f8.patch",
``` 

Then we need to replace that patch with one that is compatible with Drupal 9. So remove the patch by doing this:

```
        "patches-ignore": {
            "nymediaas/nymedia_profile": {
                "drupal/core": {
                    "Keep setting.php intact, add local config to settings.local.php": "https://github.com/zaporylie/drupal/commit/6f0b4e95f28ae945c6fa2bf0e5e81850d3e695f8.patch"
                }
            },
```

And then add a patch that is compatible with Drupal 9:

``` 
        "patches": {
            "drupal/core": {
                "Keep setting.php intact, add local config to settings.local.php": "https://github.com/eiriksm/drupal/commit/64ad2b820b2f2d93e994b6ed2889586e355ed55f.patch"
            },
```

### Missing phpunit package

If you see this error in the static tests:

``` 
Fatal error: Trait 'Symfony\Bridge\PhpUnit\Legacy\PolyfillAssertTrait' not found in /var/www/html/drupal/sites/simpletest/Assert.php on line 68
``` 

You will have to get a newer phpunit version, and the correct version of phpunit-bridge. So update your composer.json with something like this:

```
diff --git a/composer.json b/composer.json
index 0dfb896b..ab46e64b 100644
--- a/composer.json
+++ b/composer.json
@@ -76,7 +76,8 @@
         "nymediaas/nymedia_ci_overrides": "dev-8.x-1.x",
         "phpstan/phpstan-deprecation-rules": "^1",
         "phpstan/phpstan-phpunit": "^1",
-        "phpunit/phpunit": "^7"
+        "phpunit/phpunit": "^8",
+        "symfony/phpunit-bridge": "^5.3"
``` 

And then run this command:

``` 
composer update phpunit/phpunit symfony/phpunit-bridge --with-dependencies
```

### PhpunitCompatibilityTrait missing

If you get this error in static tests:

```
Scanned file /var/www/html/drupal/core/tests/Drupal/Tests/PhpunitCompatibilityTrait.php does not exist.
```

Simply replace the `PhpunitCompatibilityTrait.php` with `PhpUnitCompatibilityTrait.php` in your phpstan.neon file, because the file was renamed from Drupal Core 9.1

### Crash with entity_reference_revisions

Right after upgrade you might get this error:

``` 
Error: Call to undefined method Drupal\Core\Entity\ContentEntityType::getLowercaseLabel() in /var/www/html/drupal/modules/contrib/entity_reference_revisions/entity_reference_revisions.views.inc on line 102 #0 /var/www/html/drupal/core/modules/views/src/ViewsData.php(236): entity_reference_revisions_views_data()
``` 

This probably means you are using an outdated patch for entity_reference_revisions. You may or may not have to remove the patch via composer.json, and `patches_ignore` like this:

```
        "patches-ignore": {
            "nymediaas/nymedia_commerce_profile": {
                "drupal/entity_reference_revisions": {
                    "Views doesn't recognize relationship to host (https://www.drupal.org/node/2799479)": "https://www.drupal.org/files/issues/2019-10-07/2799479-91-DO_NOT_COMMIT.patch"
                }
            }
        },
```

In addition, if you have the patch locally in the project, remove that one, or rather replace it so it looks like this:

``` 
            "drupal/entity_reference_revisions": {
                "Views doesn't recognize relationship to host (https://www.drupal.org/project/entity_reference_revisions/issues/2799479)": "https://www.drupal.org/files/issues/2021-04-16/entity_reference_2799479-158-no-tests.patch"
            },
```

### Not able to update `file_mdm` module

Solved by requiring image_effects 3 and running update with dependencies:

```
composer require drupal/image_effects:^3 --no-update
composer update drupal/image_effects --with-dependencies
``` 

### Crashing version of addressing library

If you get this error:

``` 
PHP Fatal error:  Declaration of Drupal\address\Repository\AddressFormatRepository::processDefinition($countryCode, array $definition) must be compatible with CommerceGuys\Addressing\AddressFormat\AddressFormatRepository::processDefinition(string $countryCode, array $definition): array in /app/drupal/modules/contrib/address/src/Repository/AddressFormatRepository.php on line 38
``` 

You can simply downgrade to addressing version 1.3.0. For example by doing this:

```  
composer require "commerceguys/addressing:~1.3.0"
```

### Unable to create coupons after Commerce 2.24 update (not user 1)

Administrator role needs the permission ```administer commerce_promotion```.

### Site installation crashes when installing ckeditor_media_embed
If you composer site-install crashes with:
```
> ./vendor/bin/drupal ckeditor_media_embed:install

Error: ] Command "ckeditor_media_embed:install", is not a valid command name.   
```

Go to the main composer.json and change
`"./vendor/bin/drupal ckeditor_media_embed:install"` to `"./vendor/bin/drush ckeditor_media_embed:install"`

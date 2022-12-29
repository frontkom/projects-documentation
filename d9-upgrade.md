# Upgrading nymedia projects to Drupal 9

## Getting the environment ready

- Make sure the project is using composer 2
- Make sure the project is using drush 10
- Make sure the project has at least version ^1.0 of all phpstan related modules (`mglaman/phpstan-drupal`, `phpstan/phpstan-deprecation-rules` and `phpstan/phpstan-phpunit`
- Make sure the project has upgrade status enabled
- Make sure the project has `drupal/drupal-extension` version 4 or higher.

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


## Common errors and how to solve them

### Not able to update `file_mdm` module

Solved by requiring image_effects 3 and running update with dependencies:

```
composer require drupal/image_effects:^3 --no-update
composer update drupal/image_effects --with-dependencies


### Crashing version of addressing library

If you get this error:

``` 
PHP Fatal error:  Declaration of Drupal\address\Repository\AddressFormatRepository::processDefinition($countryCode, array $definition) must be compatible with CommerceGuys\Addressing\AddressFormat\AddressFormatRepository::processDefinition(string $countryCode, array $definition): array in /app/drupal/modules/contrib/address/src/Repository/AddressFormatRepository.php on line 38
``` 

You can simply downgrade to addressing version 1.3.0. For example by doing this:

```  
composer require "commerceguys/addressing:~1.3.0"
``` 

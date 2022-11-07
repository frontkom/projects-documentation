# Local installation of Nymedia Drupal projects

## Short version

First place the following in settings.local.php:

```php
$settings['environment'] = 'development';
```

Then run the composer script `site-install`:

```bash
# Replace db params with something appropriate. Like mysql://root@localhost/project_local
DB_PARAMS="<DB_PARAMS>" composer site-install
# If you have already installed the site, and simply want to re-install, or if you have
# entered the database credentials yourself in settings.php or settings.local.php, you
# can use the command without DB_PARAMS:
composer site-install
```

## Longer version

### Configure the codebase to be able to install

Ny Media policy makes it impossible (well, hard) to accidentally run a site install command on a codebase. The reason for this is so no-one would accidentally run it on an environment that is in use (i.e staging or production). The only exclusion to this rule is if the project has actively been marked as a development environment. We do this by placing the following in `settings.local.php`:

```php
$settings['environment'] = 'development';
```

### (optional) Enter the database settings into settings.local.php

### Run the composer site-install command

# Local installation of Nymedia Drupal projects

## Short version

1. Clone the repository
2. Install dependencies

3. Place the following in settings.local.php:

```php
$settings['environment'] = 'development';
```

4. Run the composer script `site-install`:

```bash
# Replace db params with something appropriate. Like `--db-params=mysql://root:root@localhost/mydb`
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

### Run `composer install`

Even though this will also run as part of `composer site-install`, it might be useful to run this first, to make sure scaffolding is in place, and some packages might be needed as part of environment checks in `composer site-install`. 

### (optional) Enter the database settings into settings.local.php

If your local development setup makes this easier, you can now add your local database settings to `settings.local.php`. You can also skip this if you want to pass the database settings as part of the site install command below.

### Run the composer site-install command

If you have database settings defined in settings.php, you can now simply run this command:

```
composer site-install
``` 

You can also pass along the database settings to this command, and it will be added to `settings.local.php` automatically. As an example, if you have a database available at localhost, with the default port, and the user is `root` and the password is `root`. And you want your database to be named `mydb`. Then run this as the command:

```
DB_PARAMS="--db-url=mysql://root:root@localhost/mydb" composer site-install

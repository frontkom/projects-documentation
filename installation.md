# Local installation of Nymedia Drupal projects

## Short version

First place the following in settings.local.php:

```php
$settings['environment'] = 'development';
```

Then run the composer script `site-install`:

```bash
# Replace db params with something appropriate. Like mysql://root@localhost/project_local
DB_PARAMS="<DB_PARAMS>" composer si
# If you have already installed the site, and simply want to re-install, or if you have
# entered the database credentials yourself in settings.php or settings.local.php, you
# can use the command without DB_PARAMS:
composer si
```

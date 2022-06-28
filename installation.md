# Installation of nymedia Drupal projects

## Short version

First place the following in settings.local.php:

```php
$settings['environment'] = 'development';
```

Then run the composer script `site-install`:

```bash
# Replace db params with something appropriate. Like mysql://root@localhost/vikans_local
DB_PARAMS="<DB_PARAMS>" composer si
# If you have already installed the site, and simply want to re-install, you
# can use the command without DB_PARAMS:
composer si
```

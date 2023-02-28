# Elastic search 

## Elastic search locally 

The easiest way to use elastic search locally is to use docker.

Depending on your project setup, you might have to define this in other files (like a Docker compose file or a ddev config file). Either way, the container name should be the same, and the startup options similar.

## Identifying the version used in a project 

Currently, when writing this, all our projects use the same version of elastic search in production.

## Drupal settings

### Settings for elastic search helper

Usually we define this in settings.php. If you have an elastic search server running on the host localhost and port 9200 you would indicate that in the following way:

```php
$config["elasticsearch_helper.settings"]["elasticsearch_helper"]['host'] = 'lo';
$config["elasticsearch_helper.settings"]["elasticsearch_helper"]['port'] = "80";
```

```php
$settings['elasticsearch_indexer']['index_name'] = 'sig_halvorsen_prod';
$settings['elasticsearch_indexer']['index_type'] = 'product';
```

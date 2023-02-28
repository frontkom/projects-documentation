# Elastic search 

## Elastic search locally 

The easiest way to use elastic search locally is to use docker.

Depending on your project setup, you might have to define this in other files (like a Docker compose file or a ddev config file). Either way, the container name should be the same, and the startup options similar.

## Identifying the version used in a project 

Currently, when writing this, all our projects use the same version of elastic search in production.

## Drupal settings

### Settings for elastic search helper

Usually we define this in `settings.local.php`. If you have an elastic search server running on the host localhost and port 9200 you would indicate that in the following way:

```php
$config["elasticsearch_helper.settings"]["elasticsearch_helper"]['host'] = 'localhost';
$config["elasticsearch_helper.settings"]["elasticsearch_helper"]['port'] = "9200";
```

These are by coincidence the default settings, so if your setup is identical, you should not have to specify this. If your elastic search hostname and/or port is different you would indicate it in `settings.local.php` like above.

### Settings for elastic search indexer

This has to be specified in `settings.local.php`.

Usually, if the project name is a shop, indexing products, you would want settings something like this:

```php
$settings['elasticsearch_indexer']['index_name'] = 'my_project';
$settings['elasticsearch_indexer']['index_type'] = 'product';
```



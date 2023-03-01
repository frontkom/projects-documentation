# Elastic search 

## Elastic search locally 

The easiest way to use elastic search locally is to use docker.

Depending on your project setup, you might have to define this in other files (like a Docker compose file or a ddev config file). Either way, the container name should be the same, and the startup options similar.

Even though our production use version 5.5.2 (it's managed by AWS) we usually use 5.5.3 locally. The docker image is called docker.elastic.co/elasticsearch/elasticsearch:5.5.3`.

Here is one way to start that with docker:

```
docker run -d -p 9500:9200 -e "xpack.security.enabled=false" -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:5.5.3
``` 

## Identifying the version used in a project 

Currently, when writing this, all our projects use the same version of elastic search in production. That version is 5.5.2. Yes, this is quite old, and we are in the process of trying to upgrade it, but it takes time since so many sites are using it.

If you are wondering what your specific project is using, you should be able to consult the yml file for the tests, where there should be an elasticsearch image as well. Here is one example of finding that (usually inside .github/workflows/run-tests.yml`):

```
      elasticsearch:
        image: docker.elastic.co/elasticsearch/elasticsearch:5.5.3
        env:
          xpack.security.enabled: false
          discovery.type: single-node
```

That means you can use the same image locally, and with those environment variables. The example in the yml above corresponds to the `docker run

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

### (Re)indexing the search

Depending on the settings in the above section you should now know if your project uses the index type "node" or "product". In most cases it will be "product". If that is the case, this command will delete the index, set up index templates, queue all items for indexing, and run the indexing queue for you:

``` 
drush eshd product && drush eshs product && drush eshr product && drush queue-run elasticsearch_helper_indexing
```

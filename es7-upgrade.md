# Upgrading nymedia projects to Elastic Search 7

This document is a more detailed description of updating steps mentioned [here](https://github.com/nymedia/elasticsearch_indexer/tree/9.x#upgrade-from-8x-1x-to-9x)

## Update your docker environment

If you use docker on your local, make sure you update the elasticsearch container:
```diff
--- a/.docker/docker-compose.dev.yml
+++ b/.docker/docker-compose.dev.yml
@@ -39,12 +39,20 @@ services:
       - backend
 
   elasticsearch:
-    image: docker.elastic.co/elasticsearch/elasticsearch:5.5.3
+    image: docker.elastic.co/elasticsearch/elasticsearch:7.17.9
     networks:
       - backend
     environment:
-      - xpack.security.enabled=false
-      - discovery.type=single-node
+      - node.name=store
+      - node.master=true
+      - cluster.name=es-cluster
+      - bootstrap.memory_lock=true
+      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
+      - "cluster.initial_master_nodes=store"
+    ulimits:
+      memlock:
+        soft: -1
+        hard: -1
     volumes:
       - elasticsearch:/usr/share/elasticsearch/data
```

- The ulimits has to be at least 65536
- `node.name` has to be the same as variable `$settings['elasticsearch_indexer']['index_name']` configured on the project
- `cluster.initial_master_nodes` has to be the same as above

You might need to update `vm.max_map_count` in your system with [this instructions](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/docker.html#_set_vm_max_map_count_to_at_least_262144)
before you start the container.


## Update the composer dependencies

Update the following components to at least the following versions:
- `"elasticsearch/elasticsearch": "^7.0.0",`
- `"drupal/elasticsearch_helper": "dev-7.x",`
- `"nymediaas/elasticsearch_indexer": "9.0.0-alpha1",`
- `"nymediaas/elasticsearch_ui": "9.0.0-alpha1",`
- `"nymediaas/nymedia_autocomplete": "9.0.0-alpha1",`

Then, the most important things:
- Run database updates (if any)
- Export config (if it differs)
- Export site-schema (`composer site-schema`)

## Update settings.local.php and ci.settings.php with new ES parameters:

- New ES server config
```php
// Change from
$config['elasticsearch_helper.settings']['elasticsearch_helper']['host'] = 'elasticsearch';
$config['elasticsearch_helper.settings']['elasticsearch_helper']['port'] = '9200';

// to
$config['elasticsearch_helper.settings']['hosts'][0]['host'] = 'elasticsearch';
$config['elasticsearch_helper.settings']['hosts'][0]['port'] = '9200';
```
- New ES index plugin name
```php
$settings['elasticsearch_indexer']['index_plugin'] = 'product';
```


## Update ES version in workflow

Make sure the Github Actions also use ES 7 image:
```diff
--- a/.github/workflows/run-tests.yml
+++ b/.github/workflows/run-tests.yml
@@ -185,10 +185,15 @@ jobs:
           username: ${{ secrets.GTHB_PACKAGES_USERNAME }}
           password: ${{ secrets.GTHB_PACKAGES_TOKEN }}
       elasticsearch:
-        image: docker.elastic.co/elasticsearch/elasticsearch:5.5.3
+        image: docker.elastic.co/elasticsearch/elasticsearch:7.17.9
         env:
-          xpack.security.enabled: false
-          discovery.type: single-node
+          node.name: store
+          node.master: true
+          cluster.name: es-cluster
+          bootstrap.memory_lock: true
+          ES_JAVA_OPTS: -Xms512m -Xmx512m
+          cluster.initial_master_nodes: store
+        options: --ulimit memlock=-1:-1
```


## Common issues
TBD

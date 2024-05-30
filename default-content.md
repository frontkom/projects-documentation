# Default content

We try to have default content on all projects, as far as we can manage that. This makes development and testing much easier. To do this, we use the module [default_content_deploy](https://www.drupal.org/project/default_content_deploy). We also use the module [better_normalizers](https://www.drupal.org/project/better_normalizers)

## Installation

First download the module with composer:

```
composer require drupal/default_content_deploy drupal/better_normalizers
```

Then enable it:

```
drush pm:enable default_content_deploy better_normalizers
```

The last step is to add a path to the stored content to settings.php. For example something like this:

```diff
diff --git a/drupal/sites/default/settings.php b/drupal/sites/default/settings.php
index 11cd616..7ed20d8 100755
--- a/drupal/sites/default/settings.php
+++ b/drupal/sites/default/settings.php
@@ -758,6 +758,11 @@
 
 $settings['config_sync_directory'] = '../config/common';
 
+/**
+ * Directory for default content deploy.
+ */
+$settings['default_content_deploy_content_directory'] = '../content';
+
```

## Import content by default when installing the project locally

Usually you would want to install content by default when you install a project from scratch. But it could also be useful to update the content on an existing site. This is why we define a composer script for it, which is to be called at the end of the composer script for `site-install`. For example something like this:

```diff
         "site-install": [
-            "./vendor/bin/drush --root=$(pwd)/drupal si base_profile --existing-config ${DB_PARAMS} -y"
+            "./vendor/bin/drush --root=$(pwd)/drupal si base_profile --existing-config ${DB_PARAMS} -y",
+            "@composer set-up-default-content"
+        ],
+        "set-up-default-content": [
+            "./vendor/bin/drush default-content-deploy:import -y --force-override"
```

## Exporting content

You should be able to read more about the different commands from default_content_deploy by invoking them with the `--help` flag. Here is a couple of useful examples.

### Exporting a specific node you have been working on

So I was working on a node that was made with gutenberg. Let's export it. I start by taking a note of what the node id is. In my case this is "5". The I run this command:

```
drush default-content-deploy:export node --entity_id=5
```

This gave me the following output:

```
 [notice] Exported 1 entities of the "node" entity type.
```

That sounds promising. However, the actual media item that I inserted in the middle of the content is not exported. To do that, I visit the media overview at `admin/content/media`. I find my image, and take a note of its id. In my case it's 1. So I export that as well:

```
drush default-content-deploy:export-with-references media --entity_id=1
```

This now exported the media entity, but also its references, most importantly the file referrence. It will actually also export the user entity, but we really don't care about that (I would recommend not to commit that).

That's all there is to it!

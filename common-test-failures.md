## Common test errors and how to fix them

### Exit code 2 (lock file is not up to date)

Some times your tests will fail because your lock file is not up to date. This might not be very obvious, since it's part of a complete validation, but these will be the last lines in such a test:

```
./composer.json is valid, but with a few warnings
See https://getcomposer.org/doc/04-schema.md for details on the schema
The lock file is not up to date with the latest changes in composer.json, it is recommended that you run `composer update`.
License "GPL-2.0+" is a deprecated SPDX license identifier, use "GPL-2.0-or-later" instead
Script @composer validate --no-check-all --no-check-publish handling the test-static event returned with error code 2
``` 

From the output it is certainly not obvious, but there is only one error here. And it's this:

```
The lock file is not up to date with the latest changes in composer.json, it is recommended that you run `composer update`.
```

Seemingly the error might be just a general error code 2, or maybe even about the licence (this is a warning), but the thing that makes this test fail is the lock file. This can happen if you are working in a branch for some time, and there are several updates to develop that does composer changes in the meantime. Luckily it's very easy to fix.

- Pull the branch down to your own machine
- Run the command `composer update nothing`. Sometimes this does not work (especially on composer 2). If that is the case you can also try to update an invalid package. Something like `composer update derpy/derp`.

This will hopefully give you a diff in the file composer.lock, which you can now commit. Theoretically several changes, but the important (and failing) part is the content-hash on line 7 (at the time of writing). So a diff that you want and that will probably fix things for you is something along these lines:

```
diff --git a/composer.lock b/composer.lock
index c73cfeecf..23cb2a1e0 100644
--- a/composer.lock
+++ b/composer.lock
@@ -4,7 +4,7 @@
         "Read more about it at https://getcomposer.org/doc/01-basic-usage.md#installing-dependencies",
         "This file is @generated automatically"
     ],
-    "content-hash": "d59a7e6373cc589b0b585e11774cdfc3",
+    "content-hash": "ef485df919b9f2aaa407f7a68d625367",
``` 

Commit this, and push to your branch, and things should work way better this time?️

### Error with Chrome crashing

If you get an error with Chrome tab crashed, and session ID not found something like this:
```
unknown error: session deleted because of page crash
      from tab crashed
        (Session info: headless chrome=73.0.3683.103)
```

Typically this will be as part of a behat test, and inside a javascript test more specifically.

One typical reason this fails is because the "shm-size" for chrome is too small when running inside a docker container. Like for example inside a docker container inside of github actions. One source for such a discussion is over here: [https://github.com/elgalu/docker-selenium/issues/20](https://github.com/elgalu/docker-selenium/issues/20) 

To fix this problem you just have to disable that shm stuff entirely. It will make the test go a bit slower probably, as this makes some memory stuff use the disk instead. But let's not think about that right now?

```
diff --git a/behat.yml.dist b/behat.yml.dist
index f0c0559..7ee3486 100644
--- a/behat.yml.dist
+++ b/behat.yml.dist
@@ -28,6 +32,7 @@ default:
               - "--headless"
               - "--disable-gpu"
               - "--no-sandbox"
+              - "--disable-dev-shm-usage"
```

### Error with site schema and/or clean-repo-check.

An example of this failing would be output something like this:

```
> ./vendor/bin/drush --root=$(pwd)/drupal cc drush
 [success] 'drush' cache was cleared.
> ./vendor/bin/drush --root=$(pwd)/drupal cex -y
 [notice] Differences of the active config to the export directory:
+------------+---------------------------------------------------------+-----------+
| Collection | Config                                                  | Operation |
+------------+---------------------------------------------------------+-----------+
|            | commerce_checkout.commerce_checkout_flow.default        | Update    |
|            | core.entity_view_display.commerce_order.default.user    | Update    |
|            | core.entity_view_display.commerce_order.default.default | Update    |
+------------+---------------------------------------------------------+-----------+


 // The .yml files in your export directory (../config/common) will be deleted  
 // and replaced with the active config.: yes.                                  

 [success] Configuration successfully exported to ../config/common.
../config/common
> ./vendor/bin/drush site-schema --format=json > site-schema.json
> git diff site-schema.json
> git diff
> /bin/sh ./vendor/nymedia/clean-repo-check/clean-repo-check.sh
Script /bin/sh ./vendor/nymedia/clean-repo-check/clean-repo-check.sh handling the test event returned with error code 1
diff --git a/site-schema.json b/site-schema.json
index 80369d0..7a6adc9 100644
--- a/site-schema.json
+++ b/site-schema.json
@@ -152,12 +152,12 @@
     {
         "type": "schema",
         "module": "commerce_order",
-        "value": "8209"
+        "value": "8213"
     },
     {
         "type": "schema",
         "module": "commerce_payment",
-        "value": "8206"
+        "value": "8208"
     },
     {
         "type": "schema",
@@ -172,12 +172,12 @@
     {
index 80369d0..7a6adc9 100644
--- a/site-schema.json
+++ b/site-schema.json
@@ -152,12 +152,12 @@
     {
         "type": "schema",
         "module": "commerce_order",
-        "value": "8209"
+        "value": "8213"
     },
     {
         "type": "schema",
         "module": "commerce_payment",
-        "value": "8206"
+        "value": "8208"
     },
     {
         "type": "schema",
@@ -172,12 +172,12 @@
     {
         "type": "schema",
         "module": "commerce_product",
-        "value": "8210"
+        "value": "8212"
     },
     {
         "type": "schema",
         "module": "commerce_promotion",
-        "value": "8205"
+        "value": "8207"
     },
     {
         "type": "schema",
Working tree is not clean:
HEAD detached at pull/461/merge
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   config/common/commerce_checkout.commerce_checkout_flow.default.yml
	modified:   config/common/core.entity_view_display.commerce_order.default.default.yml
	modified:   config/common/core.entity_view_display.commerce_order.default.user.yml
	modified:   site-schema.json

no changes added to commit (use "git add" and/or "git commit -a")
Error: Process completed with exit code 1.
``` 

This means the test ended up changing 3 config files, and the site schema. This is not acceptable for the tests. To fix this you do two things:

- Try to reinstall the site (with `composer site-install`) and see if you get the same config changes ready to commit. If that is the case, add those changes to your PR and this problem is solved
- To update the site schema, run the command `composer site-schema`. This will update the file site-schema.json, and you can in turn commit this and hopefully end up with a clean repo in the test

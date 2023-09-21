# Upgrade from Drupal 9 to Drupal 10

## Common errors and how to fix them

### Problem with `request_stack`.

```
TypeError: MyClass::__construct(): Argument #X ($requestStack) must be of type Drupal\Core\Http\RequestStack, Symfony\Component\HttpFoundation\RequestStack given
```

The request stack service now returns the request stack class from Symfony. So do not type hint it as the Drupal class. To fix it, just replace all instances in your class with the symfony one.

Something like this:

```diff
@@ -4,12 +4,12 @@
 
 use Drupal\advancedqueue\Job;
 use Drupal\Core\Database\Connection;
-use Drupal\Core\Http\RequestStack;
 use Drupal\Core\State\StateInterface;
 use Drupal\node\NodeInterface;
 use Psr\Log\LoggerAwareTrait;
 use Psr\Log\LoggerInterface;
+use Symfony\Component\HttpFoundation\RequestStack;
 
 /**
@@ -43,7 +43,7 @@ class MyClass {
```

### Problem with the goutte driver for behat

```
n GoutteFactory.php line 66:
                                                           
  Install MinkGoutteDriver in order to use goutte driver.  

```

Can probably be fixed with something like this:

```diff
--- a/behat.yml.dist
+++ b/behat.yml.dist
@@ -20,12 +20,7 @@ default:
       files_path: '%paths.base%/tests/files'
       ajax_timeout: 15
       base_url: http://127.0.0.1:8888
-      goutte: ~
+      browserkit_http: ~
```

### Problem using screenshot package we usually use

```
In ExtensionManager.php line 194:
                                                                                 
  `Bex\Behat\ScreenshotExtension` extension file or class could not be located.  
```

Can be fixed by removing this from the file `behat.yml.dist`:

```diff
diff --git a/behat.yml.dist b/behat.yml.dist
index f272e51..4f5a8e8 100644
--- a/behat.yml.dist
+++ b/behat.yml.dist
@@ -20,10 +20,6 @@ default:
         - Nymedia\Tests\Context\FeatureContext
         - Nymedia\Tests\Context\ProductContext
   extensions:
-    Bex\Behat\ScreenshotExtension:
-      image_drivers:
-        local:
-          screenshot_directory: "%paths.base%/screenshots"
```

# Upgrade from Drupal 9 to Drupal 10

## Common errors and how to fix them

### Problem with PHPUnit tests and ignoreFile

```
PHP Fatal error:  Uncaught InvalidArgumentException: Unknown configuration option "ignoreFile". in /var/www/html/vendor/symfony/phpunit-bridge/DeprecationErrorHandler/Configuration.php:302
```

This comes from the fact that Drupal supplies a config value to `symfony/phpunit-bridge`, which basically requires version 6. So while it's possible (dependency-wise) to have `symfony/phpunit-bridge` version 5.x installed together with Drupal 10, it really will crash the unit tests. The solution is quite simple though. Something like this:

```
--- a/composer.json
+++ b/composer.json
@@ -124,7 +124,7 @@
         "phpstan/phpstan-deprecation-rules": "^1.0",
         "phpstan/phpstan-phpunit": "^1.0",
         "phpunit/phpunit": "^9",
-        "symfony/phpunit-bridge": "^5.3"
+        "symfony/phpunit-bridge": "^6"
```

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

### Problem with unknown screenshot class

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

### Problem with unknown Browser Context class

```
In UninitializedContextEnvironment.php line 45:
                                                                                 
  `Behatch\Context\BrowserContext` context class not found and can not be used.  
                                                                                 
```

[This package is abandoned](https://github.com/Behatch/contexts) and we can no longer use it. Remove it from `behat.yml.dist`:

```diff
--- a/behat.yml.dist
+++ b/behat.yml.dist
@@ -16,24 +16,19 @@ default:
         - Drupal\DrupalExtension\Context\MarkupContext
         - Drupal\DrupalExtension\Context\MessageContext
         - Drupal\DrupalExtension\Context\DrushContext
-        - Behatch\Context\BrowserContext
         - Nymedia\Tests\Context\FeatureContext
         - Nymedia\Tests\Context\ProductContext
```

If you haven't done so already, you should probably also remove it from composer.json:

```diff
     },
     "require-dev": {
-        "bex/behat-screenshot": "^2.1",
         "drupal/coder": "^8.3.20",
```

### Problem with event subscribers and parameters

```
Argument #1 ($event) must be of type Symfony\Component\HttpKernel\Event\GetResponseEvent, Symfony\Component\HttpKernel\Event\RequestEvent given
```

Typically this means you have to change the class expected. To something like this:

```diff
+++ b/src/EventSubscriber/CartTokenSubscriber.php
@@ -10,7 +10,7 @@ use Drupal\Core\Entity\EntityTypeManagerInterface;
 use Drupal\Core\TempStore\SharedTempStoreFactory;
 use Psr\Log\LoggerInterface;
 use Symfony\Component\EventDispatcher\EventSubscriberInterface;
-use Symfony\Component\HttpKernel\Event\GetResponseEvent;
+use Symfony\Component\HttpKernel\Event\RequestEvent;
 use Symfony\Component\HttpKernel\KernelEvents;
 
 /**
@@ -83,10 +83,10 @@ final class CartTokenSubscriber implements EventSubscriberInterface {
   /**
    * Loads the token cart data and resets it to the session.
    *
-   * @param \Symfony\Component\HttpKernel\Event\GetResponseEvent $event
+   * @param \Symfony\Component\HttpKernel\Event\RequestEvent $event
    *   The response event, which contains the current request.
    */
-  public function onRequest(GetResponseEvent $event) {
+  public function onRequest(RequestEvent $event) {
```

## Twig syntax errors

Drupal 10 upgrades Twig from 2 to 3, so there are some differences. Here are some things you might encounter:

### Unexpected token "name" of value "if"

```
Twig\Error\SyntaxError: Unexpected token "name" of value "if" ("end of statement block" expected). in Twig\TokenStream->expect()
```

This happens if you add a condition to your `for` loop. It was supported, and documented in Twig 2.x: https://twig.symfony.com/doc/2.x/tags/for.html#adding-a-condition

Not that this is an indication, but that paragraph does not exist for the `3.x` documentation: https://twig.symfony.com/doc/3.x/tags/for.html

It also states in the `2.x` documentation some caveats to using it. So bottom line, one should not use it, and it does not work with Twig 3. The easiest fix is something like this:

```
+++ b/drupal/themes/custom/store/partials/components/cart/commerce-coupon-redemption-form.html.twig
@@ -33,11 +33,13 @@
       <div class="coupon-redemption-form__coupons coupon-redemption-form__coupons--multiple">
         <h3> {{ 'Applied coupons'|t }} </h3>
         <table>
-          {% for key, coupon in form.coupons if key|first != '#' %}
-            <tr>
-              <td> {{ coupon.code }} </td>
-              <td> {{ coupon.remove_button }} </td>
-            </tr>
+          {% for key, coupon in form.coupons %}
+            {% if key|first != '#' %}
+              <tr>
+                <td> {{ coupon.code }} </td>
+                <td> {{ coupon.remove_button }} </td>
+              </tr>
+            {% endif %}
```

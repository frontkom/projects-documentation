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

### Problem with deprecation in PHPUnit tests

If your Unit tests result with something like this
```
OK (8 tests, 47 assertions)

Remaining self deprecation notices (15)

  15x: Creation of dynamic property Mock_ProductInterface_017f50b1::$__metadata is deprecated
    15x in KlarnaSubscriberTest::testUnitShippingWeight from Drupal\Tests\store_checkout\Unit

```
and you wonder how to solve that, here's a suggestion below.

1. Create a phpunit-ignore file with the pattern to ignore
```
# Patterns of messages to ignore in PHPUnit tests.

%Creation of dynamic property .*__metadata is deprecated%
```
2. Add the phpunit-ignore file to the phpunit config
```diff
diff --git a/phpunit.xml.dist b/phpunit.xml.dist
index 477083cc..4043b5be 100644
--- a/phpunit.xml.dist
+++ b/phpunit.xml.dist
@@ -9,6 +9,7 @@
 >
     <php>
       <env name="SIMPLETEST_DB" value="sqlite://localhost//tmp/test.sqlite"/>
+      <env name="SYMFONY_DEPRECATIONS_HELPER" value="ignoreFile=./phpunit-ignore"/>
     </php>
     <testsuites>
         <testsuite name="drupal-composer-project tests">
```
Link to docs where the solution comes from - https://symfony.com/doc/current/components/phpunit_bridge.html#ignoring-deprecations

**You might consider using `baselineFile` instead of `ignoreFile`. Note that baselineFile doesn't allow the regular expressions for message to ignore**

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

But if you actually need a replacement for the screenshot extension, try using https://github.com/drevops/behat-screenshot with this basic setup:

```
composer require drevops/behat-screenshot
```

```diff
diff --git a/behat.yml.dist b/behat.yml.dist
index 046f74f4..39ee431e 100644
--- a/behat.yml.dist
+++ b/behat.yml.dist
@@ -20,12 +20,13 @@ default:
         - Drupal\DrupalExtension\Context\MarkupContext
         - Drupal\DrupalExtension\Context\MessageContext
         - Drupal\DrupalExtension\Context\DrushContext
+        - DrevOps\BehatScreenshotExtension\Context\ScreenshotContext
   extensions:
+    DrevOps\BehatScreenshotExtension:
+      dir: '%paths.base%/screenshots'
+      fail: true
+      fail_prefix: 'failed_'
+      purge: false

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

### Issues caused by old drush version

If you run into this annoyng deprecation messages all over the CI or with every drush command you execute
```
Deprecated: Use of "static" in callables is deprecated in phar:///usr/local/bin/drush/vendor/webmozart/assert/src/Assert.php on line 1973
```
OR you can't see any logs in the CI behat tests because of this weird error (which is caused by the message above):
```
    ╳  The "--nocolor" option does not exist.  
    ╳                                            
    ╳  
    ╳   (RuntimeException)
    │
    └─ @AfterStep # Nymedia\Tests\Context::logDumpAfterStep()
```

You should try updating `drush/drush` to version 12.

Pro tip: At the moment when I'm writing this, I think our PHP containers are using Drush 11.6, so I had to use `./vendor/bin/drush` in the workflow instead of the global one

Pro tip 2: Because of inconsistency between global and local drush there was a runtime exception in `Nymedia\Tests\Context::logDumpAfterStep()`. I decided to override the dumping errors after step to be able to see what's inside

```diff
+  /**
+   * Prints from watchdog at failed step.
+   *
+   * @AfterStep
+   */
+  public function logDumpAfterStep(AfterStepScope $scope) {
+    $failed = (99 === $scope->getTestResult()->getResultCode());
+    if ($failed) {
+      $query = \Drupal::database()
+        ->select('watchdog', 'w');
+
+      $query
+        ->range(0, 30)
+        ->fields('w')
+        ->orderBy('wid', 'DESC');
+      $rsc = $query->execute();
+      $table = [];
+      while ($result = $rsc->fetchObject()) {
+        $table[$result->wid] = (array) $result;
+      }
+      print_r($table);
+    }
+  }
```


### JS error caused by usage of `once()`

If you run into such error:
```
Error: Uncaught TypeError: $link.once is not a function ; Lineno: 73 ; Colno: 25
```

You need to go through all the custom JS libraries in the codebase and update it like:
```diff
-      $('.btn-cancel').once('cancel').on('click', function (event) {
+      $(once('cancel', '.btn-cancel')).on('click', function (event) {        
```

```diff
-                  $link.once().click(function (e) {
+                  $(once('product_link_click', $link)).click(function (e) {
```
Afterwards, make sure the library you updated has `core/once` library set as dependencies
```diff
  dependencies:
    - core/jquery
    - core/drupal
+    - core/once

```

## Twig syntax errors

Drupal 10 upgrades Twig from 2 to 3, so there are some differences. Here are some things you might encounter:

### A template that extends another one cannot include content outside Twig blocks

```
Twig\Error\SyntaxError: A template that extends another one cannot include content outside Twig blocks. Did you forget to put the content inside a {% block %} tag? in Twig\Parser->filterBodyNodes()
```

This will happen if you extend another template (for example like this):

```
{% extends "input.html.twig" %}
```

And then proceed to put things outside the `block` tag. Something like this is not allowed:

```
{% apply spaceless %}
  {%
    set classes = [
      'btn',
      type == 'submit' ? 'js-form-submit',
      icon and icon_position and not icon_only ? 'icon-' ~ icon_position,
    ]
  %}
  {% block input %}
```

Usually with the corresponding end tag outside as well:

```
  {% endblock %}
{% endapply %}
```

The fix in this case is to just move it inside. Like this:

```
+++ b/drupal/themes/custom/store/global/templates/form-elements/input--button.html.twig
@@ -22,7 +22,6 @@
  * @see template_preprocess_input()
  */
 #}
-{% apply spaceless %}
   {%
     set classes = [
       'btn',
@@ -31,6 +30,7 @@
     ]
   %}
   {% block input %}
+    {% apply spaceless %}
     {% if icon and icon_only %}
       <button{{ attributes.addClass(classes, 'icon-only') }}>
         <span class="sr-only">{{ label }}</span>
@@ -44,5 +44,5 @@
       {% endif %}
     {% endif %}
     {{ children }}
+    {% endapply %}
   {% endblock %}
-{% endapply %}
```

### Unexpected "spaceless"

```
Twig\Error\SyntaxError: Unexpected "spaceless" tag (expecting closing tag for the "block" tag defined near line 67). in Twig\Parser->subparse()
```

I actually can not find a reference to `spaceless` being a tag, but I assume this is a relic of old twig, and it has been deprecated for a while, and now finally removed. There is no `spaceless` tag any more.

Instead you would have to use `apply`: https://twig.symfony.com/doc/3.x/tags/apply.html

So a simple fix would be something like this:

```diff
+++ b/drupal/themes/custom/store/partials/components/blocks/media-banner/block--bundle--media-banner.html.twig
@@ -80,13 +80,13 @@

-    {% spaceless %}
+    {% apply spaceless %}
     ...
-    {% endspaceless %}
+    {% endapply %}
```

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

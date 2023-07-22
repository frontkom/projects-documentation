# Upgrade from Drupal 9 to Drupal 10

## Common errors and how to fix them

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

# PHPUnit for Drupal

Our workflows has an optional setting that make it possible to run PHPUnit tests as separate jobs. This will be required if you want to use PHPUnit using DTT.

To use it as a separate job simply do the following:

### Ensure the project has a composer command called phpunit

In practice, this means you can invoke `composer phpunit`. This can be something simple as:

```json
{
  "scripts": {
    "phpunit": [
      "./vendor/bin/phpunit"
    ],
    "...rest of script section and rest of file": {}
  }
}
```

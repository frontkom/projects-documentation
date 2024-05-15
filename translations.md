# Managing Translations in Drupal

Managing translations is a crucial part of making your Drupal site accessible to users from different linguistic backgrounds. Here are the steps to manage translations in your Drupal-based solution:

1. **Add New Translation Strings**: Add new translation strings to your modules by using Drupal translation helper function. For example, `t('Hello World')`.

2. **Extract New Copy**: Extract the new copy into the translation file. You can do this with the following command: `ddev composer run dump-translations`.

3. **Open the Modified File**: Open the modified file with a text editor or a Po file editor. One such editor is [poedit](https://github.com/vslavik/poedit).

4. **Add the Needed Translations**: Add the translations for the new strings in the opened file.

5. **Update the Translations**: Run the following command to update the translations in your Drupal site: `ddev drush locale-update`.

6. Check for Updates: If you don't see the updates immediately, run the following command: `ddev drush cr`. This command clears all caches in Drupal, which should make the new translations visible.

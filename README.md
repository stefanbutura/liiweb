# liiweb

[![Build Status](https://travis-ci.com/stefanbutura/liiweb.svg?branch=master)](https://travis-ci.org/drupal-composer/drupal-project)


## Install development environment from scratch

Make sure you have Drush 9 installed. This guide helps installing the 'default' site on the developer computer.

1. Create a database liiweb in MySQL
2. Clone this repository this project in `/home/user/work/liiweb/website` folder
3. Create a virtual host in Apache which should look like this. For consistency please use the same domain.
```apacheconfig
<VirtualHost *:80>
  ServerName liiweb.test
  DocumentRoot /home/user/work/liiweb/website/web/
  <Directory /home/user/work/liiweb/website/web/>
    AllowOverride All
    Require all granted
  </Directory>
  <FilesMatch ".+\.php$">
    SetHandler "proxy:unix:/run/php/php7.3-fpm.sock|fcgi://localhost"
  </FilesMatch>
</VirtualHost>
```
4. Create Drupal settings file
```bash
cp web/sites/example.settings.local.php web/sites/default/settings.local.php
```
Then open the configuration file and set the missing variables: `$settings['hash_salt']`, `$databases['default']['default'` at minimum. If available, configure additional settings: Solr integration etc.

At this stage if you visit http://liiweb.test it should open the Drupal installation procedure. Proceed further.

5. Install the instance using Drush
```shell script
drush site:install --existing-config -y
drush cim sync -y
drush cr
```
6. Open the local instance

When you open the instance http://liiweb.test again you should be able to log in with the username and passwords set by Drush.


## Update local instance

If you already have a local instance installed and configured with code and database, use `git` to get the latest developments from the `master` branch or switch to another branch you wish to test and pull changes, then execute the following commands:
```shell script
drush updatedb -y
drush cim sync -y
drush updatedb -y
drush locale:check -y
drush locale:update -y
drush cr
```

Which imports the new configuration, applies all the pending updates and imports new translations - if available.

## How to commit local changes

Step 1. Export any configuration chances done to your Drupal instance (i.e. add new fields, change settings).

```shell script
drush cex -y
```

Step 2. Stage for commit code and configuration changes (YML files). We recommend using `git add -p` to commit only relevant changes. Sometimes a configuration export might export other changes not necessarily related to current functionality.

```shell script
git add -p /path/to/modified/file
git add /path/to/new/file
git commit -m "refs #123 Added new fields"
```

TODO: Add multi-site configuration details here.

## Updating Drupal Core

Follow the steps below to update your core files.

1. Run `composer update drupal/core webflo/drupal-core-require-dev "symfony/*" --with-dependencies` to update Drupal Core and its dependencies.
1. Run `git diff` to determine if any of the scaffolding files have changed. Review the files for any changes and restore any customizations to `.htaccess` or `robots.txt`.
1. Commit everything all together in a single commit, so `web` will remain in sync with the `core` when checking out branches or running `git bisect`.
1. In the event that there are non-trivial conflicts in step 2, you may wish to perform these steps on a branch, and use `git merge` to combine the updated core files with your customized files. This facilitates the use of a [three-way merge tool such as kdiff3](http://www.gitshah.com/2010/12/how-to-setup-kdiff-as-diff-tool-for-git.html). This setup is not necessary if your changes are simple; keeping all of your modifications at the beginning or end of the file is a good strategy to keep merges easy.


## FAQ


### How can I install contrib modules?

https://www.drupal.org/docs/develop/using-composer/using-composer-to-install-drupal-and-manage-dependencies

### How can I apply patches to downloaded modules?

If you need to apply patches (depending on the project being modified, a pull
request is often a better solution), you can do so with the
[composer-patches](https://github.com/cweagans/composer-patches) plugin.

To add a patch to drupal module foobar insert the patches section in the extra
section of composer.json:
```json
"extra": {
    "patches": {
        "drupal/foobar": {
            "Patch description": "URL or local path to patch"
        }
    }
}
```

Acquia Drupal Recommended Project
====

This is a project template providing a great out-of-the-box experience for new Drupal 10 projects hosted on Acquia. It is based on the [Drupal Recommended Project](https://github.com/drupal/recommended-project/tree/9.0.x), with the principal difference being the addition of several modules and packages that provide the best possible out-of-the-box experience for Acquia customers.

This project includes the following packages and configuration:
* [Drupal core](https://www.drupal.org/project/drupal)
* [Drupal core scaffold](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold)
* [Acquia CMS](https://github.com/acquia/acquia-cms-starterkit) (Starter kit)
* [Drush](https://github.com/drush-ops/drush) (Drupal CLI and development tool)
* [Asset Packagist](https://asset-packagist.org/) repository, package, and configuration
* [Acquia environment detection](https://github.com/acquia/drupal-environment-detector)
* [Acquia platform memcache settings](https://github.com/acquia/memcache-settings)
* Best practices for Drupal development, testing and project architecture

The Acquia CMS starter kit allows you to install Drupal for a given style of CMS:

| Name  | Description |
| ------------- | ------------- |
| Acquia CMS Enterprise Low-code  | The low-code starter kit will install Acquia CMS with Site Studio and a UIkit. It provides drag and drop content authoring and low-code site building. An optional content model can be added in the installation process.  |
| Acquia CMS Community  | The community starter kit will install Acquia CMS. An optional content model can be added in the installation process.  |
| Acquia CMS Headless  | The headless starter kit preconfigures Drupal for serving structured, RESTful content to 3rd party content displays such as mobile apps, smart displays and frontend driven websites (e.g. React or Next.js).  |

## Installation of Drupal 10 + ACMS + UI Kit + BLT + Lando

### Installation on top of this repo:
* Git clone the repo
```
git clone git@github.com:pranitjha/d10acms.git
```
* Run composer install
```
composer install
```
* Run lando start
```
lando start
```
* Run blt setup
```
blt setup
```

### Fresh installation steps:
* Create a new project using Composer:
```
composer create-project acquia/drupal-recommended-project <project_name>
```
* Move to the directory
```
cd <project_name>
```
* Initilize lando
```
lando init --source cwd --recipe drupal9 --webroot docroot --name <project_name>
```
* Add BLT
```
composer require acquia/blt -W
```
* Update composer.json for blt plugin for site studio
```
composer config repositories.blt-site-studio '{"type": "vcs", "url": "https://github.com/pranitjha/blt-site-studio.git", "no-api": true}'
```
* Add site studio blt plugin
```
composer req acquia/blt-site-studio
```
* Add site studio theme
```
composer require acquia/cohesion-theme
```
* Update .lando.yml file
```
name: <project_name>
recipe: drupal9
config:
  webroot: docroot
  php: '8.1'
  composer_version: '2'
services:
  appserver:
    config:
      php: .lando/php.ini
    overrides:
      environment:
        DRUSH_OPTIONS_URI: "https://<project_name>.lndo.site/"
tooling:
  blt:
    service: appserver
    cmd: /app/vendor/bin/blt
```
* Update blt/blt.yml file
```
project:
  prefix: <project_name>
  machine_name: <project_name>
  human_name: '<project_name>'
  profile:
    name: minimal
  local:
    hostname: '${project.machine_name}.lndo.site'
    protocol: https
    uri: '${project.local.protocol}://${project.local.hostname}'
cm:
  core:
    install_from_config: true
deploy:
  tag_source: true
validate:
  twig:
    functions: [drupal_entity]
site-studio:
  cohesion-import: true
  sync-import: false
  package-import: true
  rebuild: true
```
* Add `blt/example.local.blt.yml` file
```
# Override any settings as necessary by copying to project.local.yml
#project:
#  local:
#    protocol: http
#    hostname: mysite.dev

# You can set custom project aliases in drush/site-aliases/aliases.drushrc.php.
# All local:* targets are run against drush.aliases.local.
#drush:
#  aliases:
#    local: local.mysite.dev

drupal:
  db:
    database: drupal9
    username: drupal9
    password: drupal9
    host: database

project:
  local:
    protocol: http
    hostname: '${project.machine_name}.lndo.site'
```
* Update docroot/sites/default/settings/local.settings.php file, Check the DB credentials and update with below details:
```
$db_name = 'drupal9';

/**
 * Database configuration.
 */
$databases['default']['default'] = [
  'database' => $db_name,
  'username' => 'drupal9',
  'password' => 'drupal9',
  'host' => 'database',
  'port' => '3306',
  'driver' => 'mysql',
  'prefix' => '',
];
```
And for local setup, you can add site studio keys in the local.settings.php file.
```
// Cohesion Settings.
$config['cohesion.settings']['api_key'] = '<API_KEY>';
$config['cohesion.settings']['organization_key'] = '<ORGANISATION_KEY>';
$settings['dx8_no_api_keys'] = TRUE;
```
* Add couple of new settings file to docroot/sites/default/settings/includes.settings.php and docroot/sites/default/settings/site_studio.settings.php
```
// includes.settings.php
$additionalSettingsFiles = [
  // e.g,( DRUPAL_ROOT . "/sites/$site_dir/settings/foo.settings.php" )
  DRUPAL_ROOT . "/sites/default/settings/site_studio.settings.php",
];

foreach ($additionalSettingsFiles as $settingsFile) {
  if (file_exists($settingsFile)) {
    // phpcs:ignore
    require $settingsFile;
  }
}
```
and,
```
// site_studio.settings.php
// Define site_studio_sync directory for package storage.
$settings['site_studio_sync'] = '../config/site_studio_sync';

// Export site studio config as expanded multiline.
$settings['site_studio_package_multiline'] = TRUE;
```
* Add new directory for site_studio packages 'config/site_studio_sync'
* Run lando
```
lando start
```
* Turn off the allow-plugins.acquia/blt
```
lando composer config --no-plugins allow-plugins.acquia/blt false
```
* Run blt setup
```
lando blt setup
```
* Clear Cache
```
lando drush cr
```
* Generate one time login link
```
lando drush uli
```
* Enable `Claro` as admin theme and `site studio minimal` as default theme.
* Enable site studio related modules
* Enable few more modules like (path, image, menu_ui, config_ignore, toolbar)
* Add configuration for config ignore by adding `cohesion_*` in the ignore list.
* Download UI Kit from https://sitestudiodocs.acquia.com/7.0/user-guide/download-primitives-uikit
* Upload the ui kit package to the url: `'/admin/cohesion/sync/import'`
* Export config
```
lando drush cex
```
* Export Site studio config (https://sitestudiodocs.acquia.com/7.0/user-guide/site-studio-drush-commands)
```
lando drush sitestudio:package:export
```

Once you create the project, you can and should customize `composer.json` and the rest of the project to suit your needs. You will receive updates from any dependent packages, but not from the project template itself. It's yours to keep!

For instance, you can remove a provided package by running the following command and committing the changed `composer.json` and `composer.lock` to Git:
```
composer remove acquia/mysql56
```

You should only commit changes to `composer.json` and `composer.lock`. Do not commit files in the `vendor`, `docroot/core`, and similar directories (these are ignored by the provided `.gitignore` file). In order to run your application in another environment, you’ll need to run `composer install` to reinstall these assets. [Acquia Code Studio’s](https://docs.acquia.com/code-studio/) Auto DevOps feature can do this automatically when deploying to Acquia Cloud.

## Other Drupal versions

Drupal 10 is installed by default. To install Drupal 9, use the below command:
```
composer create-project acquia/drupal-recommended-project:^1
```

## Next steps

After creating your project, if you'd also like to use Acquia BLT, do the following:
* Add BLT via Composer with `composer require acquia/blt`
* Install the [BLT Launcher](https://github.com/acquia/blt-launcher) and follow the rest of the [BLT setup guide](https://docs.acquia.com/blt/install/next-steps/).
* Set up automated testing using BLT recipes and plugins such as [BLT Behat](https://github.com/acquia/blt-behat) and the [Acquia Drupal Spec Tool](https://github.com/acquia/drupal-spec-tool).

# License

Copyright (C) 2020 Acquia, Inc.

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License version 2 as published by the Free Software Foundation.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License for more details.

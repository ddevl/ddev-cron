[![tests](https://github.com/ddev/ddev-cron/actions/workflows/tests.yml/badge.svg)](https://github.com/ddev/ddev-cron/actions/workflows/tests.yml) ![project is maintained](https://img.shields.io/maintenance/yes/2024.svg)

# DDEV-CRON <!-- omit in toc -->

- [Introduction](#introduction)
- [Getting started](#getting-started)
- [Implementation](#implementation)
  - [\*.cron](#cron)
- [Useful sites and debugging](#useful-sites-and-debugging)
- [Examples](#examples)
  - [Logging current time](#logging-current-time)
  - [Drupal cron](#drupal-cron)
  - [Laravel cron](#laravel-cron)
  - [OpenMage cron](#openmage-cron)
  - [TYPO3 scheduler](#typo3-scheduler)
  - [WordPress cron](#wordpress-cron)

## Introduction

This DDEV add-on helps to execute a command in the web container based on a cron schedule. Cron is a classic Linux/Unix service with a well-known configuration syntax.

The add-on:

- Installs and runs the cron service inside the web container
- Adds an example job that writes out the current time.

*This extension is designed to be a generic implementation. See [Running TYPO3 Cron inside the web container](https://github.com/ddev/ddev-contrib/tree/master/recipes/cronjob) for a specific example of a manual setup.*

## Getting started

- Install the DDEV cron add-on:

  For DDEV v1.23.5 or above run

  ```shell
  ddev add-on get ddev/ddev-cron
  ```

  For earlier versions of DDEV run

  ```shell
  ddev get ddev/ddev-cron
  ```

- Add at least one `.ddev/web-build/*.cron` file. This will be automatically added to crontab on startup. See [Implementation](#implementation)
- Restart DDEV to apply changes:

  ```shell
  ddev restart
  ```

## Implementation

This extension does the following:

- Adds required cron service to the web container.
- Configures the cron service using `.ddev/web-build/cron.conf`.
- Adds all `.ddev/web-build/*.cron` files to crontab scheduler.

### *.cron

This addon uses `*.cron` files to populate crontab. This allows projects to track and manage cron jobs via git.

On `ddev start`, all `.ddev/web-build/*.cron` files are:

- Copied into the `/etc/cron.d`.
- Have their permissions updated.
- Added to crontab.

See `.ddev/web-build/time.cron.example` and [Examples](#examples) section below for specific example.

## Useful sites and debugging

- [crontab guru](https://crontab.guru/) is a helpful for generating cron schedule expressions.
- For `crontab` usage, see [crontab man page](https://manpages.debian.org/buster/cron/crontab.1.en.html).
- Check crontab by running `ddev exec crontab -l`.
- If you want the cron to run on your local time instead of UTC, make sure to set `timezone` in your `.ddev/config.yaml`.
- To help debug, connect to the web container session (`ddev ssh`) and manually run the commands to confirm expected results.
- If you are running a CMS command that requires access to the database, set the environment variable `IS_DDEV_PROJECT=true`

## Examples

The following examples are provide as guides.
PRs are welcome for changes and updates for current best practices for specific frameworks.

### Logging current time

This addon provides an example that can check if the cron service is running.
Every minute, it writes the current time (UTC timezone) to `./time.log`.

- Rename `.ddev/web-build/time.cron.example` to `.ddev/web-build/time.cron`
- Restart the DDEV project to start the time example.
- After at least a minute, you should see `./time.log` containing the web container's current time.

### Drupal cron

- Create a `./.ddev/web-build/drupal.cron` file
- Add the following code to run the drupal scheduler every 10 minutes and write to a log file.
  - `DDEV_PHP_VERSION` value must match your project's PHP version.

```cron
*/10 * * * * IS_DDEV_PROJECT=true DDEV_PHP_VERSION=8.1 /var/www/html/vendor/bin/drush cron | tee -a /var/www/html/cron-log.txt
```

### Laravel cron

- Create a `./.ddev/web-build/laravel.cron` file
- Add the following code to run the laravel scheduler every minute.

```cron
* * * * * cd /var/www/html && IS_DDEV_PROJECT=true php artisan schedule:run >> /dev/null 2>&1
```

### OpenMage cron

- Create a `./.ddev/web-build/openmage.cron` file
- Add the following code to run the OpenMage scheduler every minute.

```cron
* * * * * /var/www/html/cron.sh
```

### TYPO3 scheduler

- Create a `./.ddev/web-build/typo3.cron` file
- Add the following code to run the typo3 scheduler every minute and write to a log file.

```cron
* * * * * cd /var/www/html && IS_DDEV_PROJECT=true vendor/bin/typo3 scheduler:run -vv | tee -a /var/www/html/scheduler-log.txt
```

### WordPress cron

- Create a `./.ddev/web-build/wordpress.cron` file
- Add the following code to trigger the WordPress scheduler.

```cron
*/15 * * * * IS_DDEV_PROJECT=true DDEV_PHP_VERSION=8.1 cd /var/www/html && /usr/local/bin/wp cron event run --due-now 2>&1 | tee -a /var/www/html/cron.log
```

- This configuration will run the WordPress scheduler every 15 minutes and will create a `cron.log` file in the root of your project

**Contributed and maintained by [@tyler36](https://github.com/tyler36) based on the original [Running TYPO3 Cron inside the web container](https://github.com/ddev/ddev-contrib/tree/master/recipes/cronjob) by [@thomaskieslich](https://github.com/thomaskieslich)**

**Originally Contributed by [@thomaskieslich](https://github.com/thomaskieslich) in <https://github.com/ddev/ddev-contrib/tree/master/recipes/cronjob>)**

# Migrating from single language site

In case you want to add fluent to an existing site to add multi language functionality you need to:

## Install fluent

use composer to install fluent, see [installation](installation.md)

## Configure fluent
* add locales

You can either do this in the backend, or for the first setup you can utitlise `default_records` to add the locales to the db. 
A fluent.yml might look like:

```
---
Name: myfluentconfig
After: '#fluentconfig'
---

TractorCow\Fluent\Model\Locale:
  default_records:
    nl:
      Title: German
      Locale: de_DE
      URLSegment: de
      IsGlobalDefault: 1
    en:
      Title: English
      Locale: en_GB
      URLSegment: en
```

When you run `dev/build?flush` again, this adds the records to the database if the locales table is still empty.

## Publish available pages in your default locale

Publishing of all your pages depends on the size of your site.
Now your site is broken, cause no pages have been published and added as translated page in your default locale. 

### Small sites (dozens of pages)

You can either publish all pages manually or use [publishall](https://docs.silverstripe.org/en/4/developer_guides/debugging/url_variable_tools/#building-and-publishing-urls) to publish all pages in bulk.
If you run `/admin/pages/publishall` in your browser  your site will be fixed again and you can start adding translated content.  

### Medium sites (hundreds of pages)

Use the `InitialPageLocalisation` dev task to either only localise or localise & publish your pages.
This dev task can be run either via CLI or queued as a job if Queued jobs module is installed.

Localise only example

```
dev/tasks/initial-page-localisation-task
```

Localise & publish example

```
dev/tasks/initial-page-localisation-task publish=1
```

### Large sites (thousands of pages)

For sites of this size you have to setup / implement your own solution.
Two options are recommended:

#### Repeatable dev task triggered by CRON

You can hook up the `InitialPageLocalisation` dev task to Cron to run it several times until your whole site is localised and published.
Use the `limit` option to set how many pages are to be processed within one execution.

Note that the dev task will skip pages which are already localised so eventually the dev task will stop updating your data.
Dev task produces output which informs you about how many pages were localised.
If the number of localised pages is zero it means that your whole site is now localised.

Example below will localise & publish five pages on each run.

```
dev/tasks/initial-page-localisation-task publish=1&limit=5
```

If you have a multiple environment setup it is recommended to execute this on your non-production environment and move the DB with assets to your production environment after all pages are published to minimise site downtime.

#### Page publish job

If your project has Queued jobs module installed you can create a job which publishes a single page.
Queue a job for every page you want to publish & localise and wait until all jobs are completed.
Jobs can be run in parallel which can lead to faster overall execution but require more setup and implementation effort compared to dev tasks.

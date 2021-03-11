---
title: "Building a dbt package" # to do: update this to creating
id: "building-packages"
---

## Assumed knowledge
This article assumes you are familiar with [packages](package-management), and are familiar with administering a repository on GitHub.

Developing a package is an advanced use of dbt — if you're new to the tool, we recommend that you first use the product for your own company's analytics before creating a new package.

## 1. Assessing whether a package is the right solution
Packages typically contain either:
- macros that solve a particular analytics engineer problem — for example, [auditing the results of a query](https://hub.getdbt.com/fishtown-analytics/audit_helper/latest/), [generating code](https://hub.getdbt.com/fishtown-analytics/codegen/latest/), or [adding additional schema tests to a dbt project](https://hub.getdbt.com/calogica/dbt_expectations/latest/).
- models for a common dataset, for example a dataset for a common software product like [MailChimp](https://hub.getdbt.com/fivetran/mailchimp/latest/) or, [Snowplow](https://hub.getdbt.com/fishtown-analytics/snowplow/latest/), or even models for metadata about your data stack like [Snowflake query spend](https://hub.getdbt.com/gitlabhq/snowflake_spend/latest/) and [the artifacts produced by `dbt run`](https://hub.getdbt.com/tailsdotcom/dbt_artifacts/latest/).

## 2. Create your new project
:::info Using the CLI for package development
We recommend that you use the CLI for package development — the development workflow often involves installing a local copy of your package in another dbt project — this currently isn't easily achieved in dbt Cloud.
:::

1. Use the [dbt init](init) command to create a new dbt project, which will be your package:
```shell
$ dbt init [package_name]
```
2. Create a public GitHub¹ repo, named `dbt-<package_name>`, e.g. `dbt-mailchimp`. Follow the GitHub instructions to link this to the dbt project you just created.
3. Update the `name:` of the project in `dbt_project.yml` to your package name, e.g. `mailchimp`.
4. Define the allowed dbt versions by using the [`require-dbt-version` config](require-dbt-version).

¹Currently, our package registry only supports packages that are hosted in GitHub.

## 3. Developing the package
Now that your project is set up, you can start working on your macros and/or models. We've included more tips below for writing high-quality package code.

When working on your package, we often find it useful to install a local copy of the package in another dbt project — this workflow is described [here](https://discourse.getdbt.com/t/contributing-to-an-external-dbt-package/657).
## 4. Add integration tests (optional)
We recommend that you implement integration tests to confirm that the package works as expected.

1. Create a subdirectory named `integration_tests`
2. In this subdirectory, create a new dbt project — you can use the `dbt init` command to do this. However, our preferred method is to copy the files from an existing `integration_tests` project, like the ones [here](https://github.com/fishtown-analytics/dbt-codegen/tree/HEAD/integration_tests) (removing the contents of the `macros`, `models` and `tests` folders since they are project-specific)
2. Install the package in the `integration_tests` subdirectory by using the `local` syntax, and then running `dbt deps`

<File name='packages.yml'>

```yml
packages:
    - local: ../ # this means "one directory above the current directory"
```

</File>

4. Add resources to the package (seeds, models, tests) so that you can successfully run your project, and compare the output with what you expect. The exact appraoch here will vary depending on your packages. In general you will find that you need to:
    - Add mock data via a [seed](seeds) with a few sample (anonymized) records. Configure the `integration_tests` project to point to the seeds instead of raw data tables.
    - Add more seeds that represent the expected output of your models, and use the [dbt_utils.equality](https://github.com/fishtown-analytics/dbt-utils#equality-source) test to confirm the output of your package, and the expected output matches.
    - This pattern can be seen in [the `audit-helper` integration tests](https://github.com/fishtown-analytics/dbt-audit-helper/tree/master/integration_tests), and the [snowplow integration tests](https://github.com/fishtown-analytics/snowplow/tree/master/integration_tests).

5. Confirm that you can run `dbt run` and `dbt test` succesfully.

5. (Optional): Use a CI tool, like CircleCI or GitHub Actions, to automate running your dbt project when you open a new Pull Request. For inspiration, check out our [CircleCI config for the snowplow package](https://github.com/fishtown-analytics/snowplow/blob/master/.circleci/config.yml), which runs tests against our four main warehouses. Note: this is an advanced step — if you are going down this path, you may find it useful to say hi on [dbt Slack](https://community.getdbt.com/).

## 5. Deploying the docs for your package (optional)
The dbt docs site can help a prospective user of your package understand the code you've written. As a result, it might be useful to deploy the site generated by `dbt docs generate` when sharing your package.

The easiest way we've found to do this is to use [GitHub Pages](https://pages.github.com/).

1. On a new git branch, run `dbt docs generate`. If you have integration tests set up (above), use the integration-test project to do this.
2. Move the following files into a directory named `docs` ([example](https://github.com/fivetran/dbt_ad_reporting/tree/HEAD/docs)): `catalog.json`, `index.html`, `manifest.json`, `run_results.json`.
3. Merge these changes into the main branch
4. Enable GitHub pages on the repo in the settings tab, and point it to the “docs” subdirectory
4. GitHub should then deploy the docs at `<org-name>.github.io/<repo-name>`, like this: [fivetran.github.io/dbt_ad_reporting](https://fivetran.github.io/dbt_ad_reporting/)

## 6. Release your package
Create a new [release](https://docs.github.com/en/github/administering-a-repository/managing-releases-in-a-repository) once you are ready for others to use your work! Be sure to use [semantic versioning](https://semver.org/) when naming your release.

In particular, if new changes will cause errors for users of earlier versions of the package, be sure to use _at least_ a minor release (e.g. go from `0.1.1` to `0.2.0`).

The release notes should contain an overview of the changes introduced in the new version. Be sure to call out any changes that break the existing interface!


## 7. Add the package to hub.getdbt.com
Our package registry, [hub.getdbt.com](https://hub.getdbt.com/), gets updated by the [hubcap script](https://github.com/fishtown-analytics/hubcap). To add your package to hub.getdbt.com, create a PR on the [hubcap repository](https://github.com/fishtown-analytics/hubcap) to include it in the `hub.json` file.

### multi-warehouse compatibility
- using dbt utils
- dispatch macros
- enabled models

### Generalized transformations
- Use the dbt SQL code conventions
- in particular, name the models with the data source in mind, e.g. `mailchimp_campaigns`, not just `campaigns`.

### handling edge cases:
What if someone needs to use a different schema name?

- Allow for customization by using variables for source names -> But what if someone

What if someone is using mutliple packges in their project / has conflicts?
->

What if someone has a name

## Package best practices

:::info

Have a tip you don't see here? Feel free to suggest an edit!

:::

**1. Keep it generic**

The code in your package should be specific to a given dataset, topic, or domain. A Mailchimp package should only contain models that transform raw Mailchimp data, for instance.

**2. Use variables to point to raw data**
(need to update to mention sources)
ETL services often allow users to configure the schema in which their data resides. For that reason, it's a bad idea to hardcode raw data tables (eg. `"mailchimp"."campaigns"`) in your base models. Instead, select from [vars](var) in your base models so end-users can point your package to their source data. You can find an example of variable configuration [here](https://github.com/fishtown-analytics/mailchimp/blob/master/dbt_project.yml#L12). The base model SQL should look something like this:

<File name='models/base/mailchimp_base_campaigns.sql'>

```sql
{{ config(materialized='ephemeral') }}

select * from {{ var('mailchimp:campaigns_table') }}
```

</File>

**3. Install upstream packages from dbt Hub**

If your package relies on another package (for example, you use some of the cross-database macros from [dbt-utils](https://hub.getdbt.com/fishtown-analytics/dbt_utils/latest/), see below), we recommend you install the package from [dbt Hub](https://hub.getdbt.com), specifying a version range like so:

<File name='packages.yml'>

```yaml
packages:
  - package: fishtown-analytics/dbt_utils
    version: ">0.1.23"
```

</File>

When packages are installed from dbt Hub, dbt is able to handle duplicate dependencies – using a range of versions helps prevent version conflicts.

**4. Implement cross-database compatibility**

Many SQL functions are specific to a particular database. For example, the function name and order of arguments to calculate the difference between two dates varies between Redshift, Snowflake and BigQuery, and no similar function exists on Postgres! To help with this, we've made a number of macros that compile to valid SQL snippets on each of our [fully supported databases](available-adapters) – check them out [here](https://github.com/fishtown-analytics/dbt-utils#cross-database). While we generally don't recommend you implement these in a project designed for one warehouse, we do recommend you use them in a package that will be used on multiple data warehouses.

**5. Prefix model names**

Many datasets have a concept of a "user" or "account" or "session". To make sure things are unambiguous in dbt, prefix all of your models with `[package_name]_`. For example, `mailchimp_campaigns.sql` is a good name for a model, whereas `campaigns.sql` is not.

**6. Default to views**

dbt makes it possible for users of your package to override your model materialization settings. In general, default to materializing models as `view`s instead of `table`s. Base models, as always, should be materialized as `ephemeral`.

**7. Test and document your models**

It's critical that you [test](building-a-dbt-project/tests) your models using both schema tests and, when appropriate, custom data tests. This will give your end users confidence that your package is actually working on top of their dataset as intended.

**8. Include useful GitHub artifacts**
- Readme with:
    - installation instructions (including and configurations required)
    - for macro packages: usage instructions
    - for modelling packages: description of models
- PR templates, issue templates, even a changelog is good

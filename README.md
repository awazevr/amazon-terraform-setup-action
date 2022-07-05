# Example Shared Infrastructure Terraform Modules

This repo contains the following example code:

* [mysql](/mysql): An example of how to deploy MySQL on top of Amazon's Relational Database Service (RDS).  

Note: This code is solely for demonstration purposes. This is not production-ready code, so use at your own risk. If 
you are interested in battle-tested, production-ready Terraform code, check out [Gruntwork](http://www.gruntwork.io/).



## Inputs

| Name               | Description                              | Type           | Default | Required |
| ------------------ | ---------------------------------------- | -------------- | ------- | -------- |
| `example_required` | An example of a required input variable  | `list(string)` | -       | yes      |
| `example_optional` | An example of an optional input variable | `map(string)`  | `{}`    | no       |



## Outputs

| Name      | Description                    |
| --------- | ------------------------------ |
| `example` | An example of an output value  |



## How do you use these modules?

To use a module, create a  `terragrunt.hcl` file that specifies the module you want to use as well as values for the
input variables of that module:

```hcl
locals {
  # Automatically load environment-level variables
  environment_vars = read_terragrunt_config(find_in_parent_folders("env.hcl"))

  # Extract out common variables for reuse
  env = local.environment_vars.locals.environment
}

# Terragrunt will copy the Terraform configurations specified by the source parameter, along with any files in the
# working directory, into a temporary folder, and execute your Terraform commands in that folder.
terraform {
  source = "git@github.com:awazevr/aws-mysql-module.git//mysql?ref=v1.0.0"
}

# Include all settings from the root terragrunt.hcl file
include {
  path = find_in_parent_folders()
}

# These are the variables we have to pass in to use the module specified in the terragrunt configuration above
inputs = {
  name           = "mysql_${local.env}"
  instance_class = "db.t2.micro"

  allocated_storage = 20
  storage_type      = "standard"

  master_username = "admin"

  tags = {
    environment = local.env
    project     = "demo"
    team        = "the A team"
  }
}
```

(*Note: the double slash (`//`) in the `source` URL is intentional and required. It's part of Terraform's Git syntax 
for [module sources](https://www.terraform.io/docs/modules/sources.html).*)

You then run [Terragrunt](https://github.com/gruntwork-io/terragrunt), and it will download the source code specified 
in the `source` URL into a temporary folder, copy your `terragrunt.hcl` file into that folder, and run your Terraform 
command in that folder: 

```
> terragrunt apply
[terragrunt] Reading Terragrunt config file at terragrunt.hcl
[terragrunt] Downloading Terraform configurations from git::git@github.com:gruntwork-io/terragrunt-infrastructure-modules-example.git//path/to/module?ref=v0.0.1
[terragrunt] Copying files from . into .terragrunt-cache
[terragrunt] Running command: terraform apply
[...]
```

Check out the [terragrunt-infrastructure-live-example 
repo](https://github.com/gruntwork-io/terragrunt-infrastructure-live-example) for examples and the [Keep your Terraform 
code DRY documentation](https://github.com/gruntwork-io/terragrunt#keep-your-terraform-code-dry) for more info.



## How do you change a module?

### Local changes

Here is how to test out changes to a module locally:

1. `git clone` this repo.
1. Update the code as necessary.
1. Go into the folder where you have the `terragrunt.hcl` file that uses a module from this repo (preferably for a 
   dev or staging environment!).
1. Run `terragrunt plan --terragrunt-source <LOCAL_PATH>`, where `LOCAL_PATH` is the path to your local checkout of
   the module code. 
1. If the plan looks good, run `terragrunt apply --terragrunt-source <LOCAL_PATH>`.   

Using the `--terragrunt-source` parameter (or `TERRAGRUNT_SOURCE` environment variable) allows you to do rapid, 
iterative, make-a-change-and-rerun development.


### Releasing a new version - [Semantic Release](https://semantic-release.gitbook.io/semantic-release/)

### How does it work?

### Commit message format

**semantic-release** uses the commit messages to determine the consumer impact of changes in the codebase. Following formalized conventions for commit messages, semantic-release automatically determines the next [semantic version](https://semver.org/) number, generates a changelog and publishes the release.

By default, semantic-release uses [Angular Commit Message Conventions](https://github.com/angular/angular/blob/master/CONTRIBUTING.md#-commit-message-format). The commit message format can be changed with the preset or config options of the [@semantic-release/commit-analyzer](https://github.com/semantic-release/commit-analyzer#options) and [@semantic-release/release-notes-generator](https://github.com/semantic-release/release-notes-generator#options) plugins.

Tools such as [commitizen](https://github.com/commitizen/cz-cli) or [commitlint](https://github.com/commitizen/cz-cli) can be used to help contributors and enforce valid commit messages.

The table below shows which commit message gets you which release type when semantic-release runs (using the default configuration):

| Commit message | Release type |
| ----------- | ----------- |
| fix(pencil): stop graphite breaking when too much pressure applied | Fix Release |
| feat(pencil): add 'graphiteWidth' option | Feature Release |
| BREAKING CHANGE: The graphiteWidth option has been removed. The default graphite width of 10mm is always used for performance reasons.| (Note that the BREAKING CHANGE: token must be in the footer of the commit) |


### Automation with CI

**semantic-release** is meant to be executed on the CI environment after every successful build on the release branch. This way no human is directly involved in the release process and the releases are guaranteed to be [unromantic and unsentimental](http://sentimentalversioning.org/).

### Triggering a release

For each new commit added to one of the release branches (for example: **master**, **next**, **beta**), with **git push** or by merging a pull request or merging from another branch, a CI build is triggered and runs the semantic-release command to make a release if there are codebase changes since the last release that affect the package functionalities.

# common-automation-framework

## Overview

This repository is used to seed component dependencies of another repository (example: a terraform module) and is applied using the [repo](https://gerrit.googlesource.com/git-repo/) utility.  Launch has updated the repo utility to allow for substituting environment variables in the manifests to allow for more dynamic behavior.  This is located [here](https://github.com/nexient-llc/git-repo).

This controls how different Launch Platform tooling dependencies are handled.

## Getting Started

Note: this assumes you have an HTTP access token for bitbucket stored securely in your `~/.netrc` file.

* Copy [`templates/seed_makefile`](./templates/seed_makefile) to a new folder.
* Create or copy some terraform code
* Create or copy an asdf .tool-versions file
* run `asdf install`
* run `make clean init-clean`
* run `export GITBASE=https://github.com/nexient-llc`
* run `make configure`

Then you can run: `make tfmodule/plan`.

## Manifests

Manifests are found in [`manifests/*/seed/manifest.xml`](./manifests/*/seed/manifest.xm).  They are retrieved through the `-m`, `-u` and `-b` options to `repo init`, which would be invoked by a `make configure` from a skeleton-based repository.  A skeleton Makefile is located in [`templates/seed_makefile`](./templates/seed_makefile).

An example using [`manifests/terraform_modules/seed/manifest.xml`](./manifests/terraform_modules/seed/manifest.xml) would be initialized using these entries in a Makefile:

```
REPO_MANIFESTS_URL ?= https://github.com/saathvika8/common-automation-framework-test.git
REPO_BRANCH ?= main
REPO_MANIFEST ?= manifests/terraform_modules/seed/manifest.xml

# Settings to pull in Launch version of repo:
REPO_URL=https://github.com/nexient-llc/git-repo
REPO_REV=envsubst

export REPO_REV
export REPO_URL
...

configure: configure-git-hooks
	repo --color=never init --no-repo-verify \
		-u "$(REPO_MANIFESTS_URL)" \
		-b "$(REPO_BRANCH)" \
		-m "$(REPO_MANIFEST)"
	repo envsubst
	repo sync
```

This example is using Launch's modified version of the repo utility which adds support for the `envsubst` sub-command.

Manifests in the [git-connection](./git-connection/) directory are used to define `remote` and `default` references, providing the base git URLs for all other `project` items defined in the manifests.  In this version, the git base is defined as an environment variable, which would be replaced locally by the `repo envsubst` command.

It is intended that the `repo` command would start from a seed manifest residing at [`manifests/*/seed/manifest.xml`](./manifests/manifests/*/seed/manifest.xml).

Items under [`artifacts`](artifacts/) are not intended to be directly consumed, but are used as lin targets when `repo sync` is executed.

## Notes

* You *must* define your BitBucket credentials in your `~/.netrc` and that *must* use an HTTP access token.  Also, that token *must* have some expiration date, otherwise BitBucket will lock your account after a few accesses.

* The seed_makefile under templates is a template for a makefile that is initially placed in the repo of a new project to boostrap it with the DSO platform.  This can also be created by using a skeleton project to start a new module.

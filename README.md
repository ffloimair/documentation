# Description

This repository contains the Asterisk Documentation project.

# Recent Changes

The packages used to create the documentation were upgraded on February 2 2024.  While there are no breaking changes, some new capabilities were added that, if used going forward, won't render correctly with the older versions.  So, if you build documentation locally, you should delete your local virtualenv and re-create it using the new requirements.txt file.  The new features will be documented separately.

# Static Documentation

The static documentation contained in the ./docs/ directory is written directly in markdown.  The publish process uses [mkdocs](https://www.mkdocs.org) and [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) to generate the HTML web site.  The directory structure is fairly straightforward so if you wish to contribute, you should fork this repository and submit pull requests against files in that directory.

All contributions are subject to the 
[Creative Commons Attribution-ShareAlike 3.0 United States](LICENSE.md) license.

# Markdown Flavor
The docs are written in standard markdown, not GitHub Flavored markdown.  There are lots of extensions available though.  Most of the extensions provided by [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/reference/) are enabled except those only available to paying sponsors and a few that don't make sense in this environment.

> [!NOTE]
> The conversion process that moved the documentation from the Confluence Wiki used equals signs `=====` and dashes `----` to denote headers.  The new infrastructure renders those correctly but they do not get added to the page's table of contents.  Going forward, you should always use hash signs `#`, `##`, `###`, etc to denote headers.

# Dynamic Documentation

The dynamic documentation includes the pages generated from Asterisk itself and includes:
* AGI_Commands
* AMI_Actions
* AMI_Events
* Asterisk_REST_Interface
* Dialplan_Applications
* Dialplan_Functions
* Module_Configuration

The publish process gets this information directly from the Asterisk CreateDocs job (which runs nightly) and generates markdown.  For this reason, all changes to the dynamic documentation need to be made in the Asterisk source code itself.  

The AGI, AMI, Dialplan and Module documentation comes from the documentation embedded in the provider modules and generated by CreateDocs running `xmldoc dump` from the Asterisk CLI.  You can also use a core-en_US.xml or full-en_US.xml file generated from a local asterisk build.  See below for more information.

The Asterisk_REST_Interface documentation comes from the markdown files generated by CreateDocs running `make ari-stubs`.  You can also use locally generated markdown files.  See below for more information.

# Build/Test Dependencies

Dependencies for documentation can be installed using the included requirements.txt file.

```
$ pip3 install -r requirements.txt
```

If you don't want to install the requirements for the current user or globally, you can create a virtual environment specific to this directory first...

```
$ python -m venv ./.venv
$ source .venv/bin/activate
(.venv)$ pip3 install -r requirements.txt
# run `deactivate` when done
(.venv)$ deactivate
$ 
```

The next time you want to test, you can just activate the virtual environment without needing to install the dependencies again.  The `.venv` directory is already in the `.gitignore` file.

You'll also need the [`gh`](https://cli.github.com) tool to pull down the dynamic documentation from the CreateDocs job.

> [!NOTE]
> The documentation build process no longer uses the `mike` python package.

# Building and Testing the Documentation

## Prepare

Everything is done from the [`documentation`](https://github.com/asterisk/documentation) repository so, if you haven't already, clone it and check out the `main` branch.

```
$ git clone -b main https://github.com/asterisk/documentation
$ cd documentation
```

Create a `Makefile.inc` file with some configuration variables.  This file must **not** be checked in.  Here are the contents to use:

```
# BUILD_DIR := <somepath>  # Defaults to ./temp

# The following 2 DEPLOY_ variables are only needed if you
# need to deploy the built site to some other repo.
# The nightly job uses this to publish the site to
# GitHub pages.
# DEPLOY_REMOTE := <git_remote>
# DEPLOY_BRANCH := <gh pages branch>  # Defaults to gh-pages

# The comma-separated list of branches for which dynamic
# documentation should be built when doing a `make` without
# specifying BRANCH=<branch>.
BRANCHES := 16,18,19,20

# If you don't want to build the static documentation at all...
# NO_STATIC=yes

# If you don't want the resulting HTML minified, set NO_MINIFY.
# Minification can reduce the space required to host the full
# site by about 30% but it does take over double the time to
# generate the site.
# NO_MINIFY=yes

# If you want to serve the resulting site with mkdocs serve,
# you can specify any additional options to pass to it here:
# SERVE_OPTS := -a [::]:8000
```

If you're planning on using local sources for the dynamic Asterisk documentation, you'll *also* need to create a `Makefile.<branch>.inc` file *for each branch*.

```
# If you want to use a local XML file to generate the
# AGI, AMI, Dialplan and Module_Configuration documentation,
# specify it here.
# ASTERISK_XML_FILE := <somepath>/asterisk/doc/core-en_US.xml
#
# If you want to use local markdown files for the ARI
# documentation, specify a path to a directory containing
# the markdown generated by "make ari-stubs".
# ASTERISK_ARI_DIR := <somepath>/asterisk/doc/rest-api
#
# If you want to use local XML but skip processing ARI
# altogether, set this variable to "yes".
# SKIP_ARI := yes

# If either ASTERISK_XML_FILE or ASTERISK_ARI_DIR are not set,
# that documentation source will be downloaded from the
# CreateDocs job.
```

## Build and Deploy

### To build the entire site:

```
$ make
```

This will build the static pages and the dynamic pages for all branches listed in the `BRANCHES` variable.  The fully functioning site will be created at `$(BUILD_DIR)/site`.  You can serve that directory with `make serve`, which runs `mkdocs serve` or a web server of your own choosing. `mkdocs serve` rebuilds the entire site when it starts so you may want to use a standard web server if you plan on doing this a lot.  If you do use `make serve`, you can use the `SERVE_OPTS` variable in Makefile.inc to add additional options to the `mkdocs serve` command line.

### To build just the static pages

```
$ make BRANCHES=
```

### To build just 2 branches

Building branches does need at least a skeleton static layout so `static-setup` will be built first if it hasn't already been built.

```
$ make BRANCHES=18,20 
```

If you only want the skeleton static documentation, you can add `NO_STATIC=yes` to that command line...

```
$ make BRANCHES=18,20 NO_STATIC=yes
```


### To build just 1 branch

If you're always going to build just 1 branch's dynamic documentation, you can skip the `Makefile.<branch>.inc` file and just place everything in the main `Makefile.inc`:

Makefile.inc:

```
ASTERISK_XML_FILE := <path>/core-en_US.xml
ASTERISK_ARI_DIR := <path>/rest-api
BRANCHES := 20
NO_STATIC := yes
```

And just run `make`.


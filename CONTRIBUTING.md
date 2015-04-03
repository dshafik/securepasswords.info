# Contributing

> **Note:** *Do not* submit pull-requests against the `gh-pages` branch

It's super easy to contribute to [SecurePasswords.info](http://securepasswords.info). All you need is some Markdown knowledge.

## Forking

You should be working in a fork, see the [following documentation](https://help.github.com/articles/fork-a-repo/)

## Before Making Any Changes

### Fetch The Latest Changes from upstream

> On the `master` branch

```sh
$ git fetch --all
$ git rebase upstream/master
```

### Create a New Branch

```sh
$ git checkout -b reason-for-changes
```

### Install Sculpin

In order to test your edited or new examples, you'll need to have Sculpin installed
in your dev environment. Please see the [Sculpin "Get Started"](https://sculpin.io/getstarted/) guide
for installation instructions.

## Editing an Existing Example:


Find the file in `_posts`, e.g. `php.md` and edit using Markdown.


## Creating a New Example

Create a new file in `_posts` (e.g. my-framework.md)

Each file starts with the meta-data:

```
---
title: "PHP"
layout: "post"
color: "fff"
background: "757eb1"
author: "Davey Shafik"
author_url: "http://daveyshafik.com"
author_twitter: "dshafik"
gravatar: "fee39f0c0ffb29d9ac21607ed188be6b"
sponsor: "Engine Yard"
sponsor_url: "https://www.engineyard.com"
sponsor_twitter: EngineYard
sponsor_image: "http://securepasswords.info/themes/dshafik/securepasswords.info/assets/images/engineyard.png"
---
```

- title: The title of the page
- layout: The layout to use (always post)
- color: The text color for the tile on the homepage
- background: The background color for the tile on the homepage
- author: The author name (optional)
- author_url: Authors URL (optional)
- author_twitter: Authors Twitter handle (optional)
- gravatar: Gravatar email hash (optional)
- sponsor: Sponsor name (optional)
- sponsor_url: Sponsor URL (optional)
- sponsor_twitter: Sponsor Twitter handle (optional)
- sponsor_image: Sponsor logo image (optional)

## Testing Your Changes

In the root directory, to test run:

```sh
$ sculpin generate --watch --server
```
Then visit <http://localhost:8000>


## After Making Your Changes

### Commit Your Changes

```sh
$ git add _posts/FILE.md
$ git commit -m "DESCRIPTION OF CHANGES"
$ git push origin master
```


## Publishing Your Changes

> **Note:** You should only do this to test, you should not perform pull-requests against this branch.

In the root directory, to publish run:

```sh
$ ./publish.sh "COMMIT MESSAGE HERE"
```

## Pushing Changes Back Upstream

To contribute your changes back, simply perform a [Pull Request](https://help.github.com/articles/using-pull-requests/) against the master branch.

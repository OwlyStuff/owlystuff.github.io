---
layout: post
title:  "npm publish"
date:   2016-05-18
type: platform
intro: The Amazium CSS framework is now published on npm, let us show you how we keep deployment simple.
---

![NPM Travis and GitHub Logos](img/npm-travis-github-logos.jpg)

## tl;dr;

With a small amount of setup, you can build, tag and release new versions of your code with a simple;

```
npm version minor && git push owlystuff master --tags
```

# The Detail

## SCSS to CSS

Amazium is written using SCSS (you will see more of it cleaned up using SCSS power over time), but for ease of use, we provide a pre-built
CSS distribution.
We manage the build pipeline using [```gulp-sass```](https://www.npmjs.com/package/gulp-sass).

### Installation and Config

First up, we need the gulp-sass library

```sh
install gulp-sass --save-dev
```

We have two very simple gulp tasks configured, one for compilation, and one for watching file changes (for auto recompilation). Both very standard, nothing new or exciting here.

```javascript
'use strict';

var gulp = require('gulp');
var sass = require('gulp-sass');

gulp.task('compile', function () {
  return gulp.src('./scss/*.scss')
    .pipe(sass().on('error', sass.logError))
    .pipe(gulp.dest('./dist/'));
});

gulp.task('watch', function () {
  gulp.watch('./scss/*.scss', ['compile']);
});
```

## Publishing a new Release

Once we are happy that all the code for the next release has been merged into the master branch (Generally via PR's), then we only have to
perform a couple of steps in order to get it published.

We locally ensure our master branch is up to date, and that we have all the tags too

```sh
git pull owlystuff master --tags
```

Work out what the new version should be ([semver](http://semver.org/)).

Latest published version is shown with ```npm version```

npm version allows manual definition of the new version

```sh
npm version 4.0.0-beta.5
```

Or you can tell it to increment a major, minor or patch version using one of;

```
npm version major
npm version minor
npm version patch
```

Running any of those commands performs a few things for us;

 * Updates the package.json version entry to the new version
 * commits that change to git
 * tags the changes as the specific version

All that leaves us to do, is push the new tag to GitHub.

```
git push owlystuff master --tags
```

At this point, travis takes over of building and publishing to npm

Our travis config is very simple and just executes our gulp command

```yaml
language: node_js
node_js:
  - '4'
sudo: false
install:
  - npm install
script:
  - node_modules/.bin/gulp compile
```

The first travis deploy entry builds the CSS, creates a release in GitHub and attaches the built CSS file.

```yaml
deploy:
  - provider: releases
    api_key:
      secure: ENCRYPTED_GITHUB_API_TOKEN
    file: dist/amazium.css
    skip_cleanup: true
    on:
      repo: OwlyStuff/amazium
      tags: true
  #...
```

After which, the second deploy entry sends to new built CSS as a new release to npm

```yaml
deploy:
  #...
  - provider: npm
    email: chris@sedlmayr.co.uk
    skip_cleanup: true
    api_key:
      secure: ENCRYPTED_NPM_API_TOKEN
    on:
      repo: OwlyStuff/amazium
      tags: true
```

All done, now check out your [latest release on npm](https://www.npmjs.com/package/amazium)!

# GitHub Plugin for Travis CLI [![Build Status](https://travis-ci.org/travis-ci/travis-cli-gh.png?branch=master)](https://travis-ci.org/travis-ci/travis-cli-gh)

This plugin for the [Travis Command Line Client](https://github.com/travis-ci/travis#readme) adds commands that interact with the [GitHub API](http://developer.github.com/v3/) rather than the Travis API. This plugin requires Ruby 2.0 or newer.

#### Why a plugin?

So why is this a plugin instead of its stand-alone command line tool?

* To demonstrate how to write a plugin for the travis gem and distribute it as gem itself.
* To rely on the existing code (auth, configuration, enterprise integration, etc) and infrastructure (like fancy zsh completion, debugging, error reporting).
* Some commands are [Travis specific](#gh-signature).
* All the commands are useful for scripting workflows around Travis CI.

That said, most of this could probably make it into a stand-alone tool.

## Usage

This plugin adds the following commands: [`gh-cat`](#gh-cat), [`gh-fetch`](#gh-fetch), [`gh-login`](#gh-login), [`gh-signature`](#gh-signature), [`gh-upload`](#gh-upload), [`gh-whoami`](#gh-whoami) and  [`gh-write`](#gh-write). All these commands will use the Travis API endpoint for automatically figuring out which GitHub API endpoint to use (relevant for setups using GitHub Enterprise).

### `gh-cat`

This displays one (or more) files from a repository:

    $ travis gh-cat .travis.yml -r rails/rails
    script: 'ci/travis.rb'
    before_install:
      - travis_retry gem install bundler
      - "rvm current | grep 'jruby' && export AR_JDBC=true || echo"

### `gh-fetch`

Just like `travis gh-cat`, but stores the contents in a file by the same name instead of displaying it. Allows you to specify a path to store it under via `-p` and overrides existing files with `-f`.

### `gh-login`

This command basically mimics [`travis login`](https://github.com/travis-ci/travis#login) with the notable difference that it does not delete the token on GitHub afterwards (as it will be needed for running other commands) and does not send the GitHub token to Travis CI. Instead, it will store the GitHub token for later use.

Note that it might not be necessary to run this command if you already have a GitHub token available (for instance, in your .netrc).

### `gh-signature`

This command lets you calculate the signature [used for web hooks](http://about.travis-ci.org/docs/user/notifications/#Authorization):

    $ travis gh-signature
    Signature used for travis-ci/travis web hooks: 57ca05aa0db5d27d20b2df24e27e2a191deeb7dd37075d921b76b95eddf60b9b

You can use this signature in your web application to verify that a hook was indeed fired by Travis CI.

Like [other commands](https://github.com/travis-ci/travis#repository-commands), it uses the current directory to determine the repository slug. It is also possible to provide it explicitly:

    $ travis gh-signature -r sinatra/sinatra
    Signature used for travis-ci/travis web hooks: 68db16bb1ec6e38e31c3eg35f38f3b2102effc8ee48186e1032c87c106feeg71

This signature should be kept somewhat secret, otherwise other people can send you fake web hooks.

### `gh-upload`

    GitHub plugin: upload one or more files to the repository.
        -h, --help                       Display help
        -E, --[no-]explode               don't rescue exceptions
            --debug                      show API requests
        -X, --enterprise [NAME]          use enterprise setup (optionally takes name for multiple setups)
        -r, --repo SLUG                  repository to use (will try to detect from current git clone)
        -b, --branch BRANCH              branch to write to
        -m, --message MESSAGE            commit message
        -C, --create                     only create new files
        -U, --update                     only update existing files
        -c, --committer USER             set committer (format: "name <email>", defaults to current user)
        -a, --author USER                set author (format: "name <email>", defaults to committer)
        -A, --append                     append to file if it already exists
        -p, --prefix DIR                 prefix to store it under

Like [`gh-write`](#gh-write), but uploads a file by the same name.

    $ travis gh-upload ../somwhere/example.txt -p test
    Uploading ../somwhere/example.txt as test/example.txt

### `gh-whoami`

Displays which GitHub account you are logged in with on GitHub.

    $ travis gh-whoami
    logged in as rkh

### `gh-write`

    GitHub plugin: write stdin to the given path in the repository.
    Usage: travis gh-write path [options]
        -h, --help                       Display help
        -E, --[no-]explode               don't rescue exceptions
            --debug                      show API requests
        -X, --enterprise [NAME]          use enterprise setup (optionally takes name for multiple setups)
        -r, --repo SLUG                  repository to use (will try to detect from current git clone)
        -b, --branch BRANCH              branch to write to
        -m, --message MESSAGE            commit message
        -C, --create                     only create new files
        -U, --update                     only update existing files
        -c, --committer USER             set committer (format: "name <email>", defaults to current user)
        -a, --author USER                set author (format: "name <email>", defaults to committer)
        -A, --append                     append to file if it already exists

Reads from stdin and writes it to the given file in the repository.

    $ travis gh-write example -r example/repo
    Hi folks!
    <Ctrl+d>

## Installation

You install it like any other gem:

    $ gem install travis-cli-gh

We also push pre-releases to rubygems.org on a regular basis:

    $ gem install travis-cli-gh --pre

### Running against a local clone

If you want to run commands from a local clone of the travis-cli-gh repository (say, if you want to contribute a patch), make sure you have the lib directory on your load path.

Here's an example:

    $ ruby -Ilib -S travis gh-signature

## Version History

**0.1.0** (December 29, 2013)

* Initial release
* Add `travis gh-cat` (displays the contents of a file on GitHub)
* Add `travis gh-fetch` (download a file from a repository)
* Add `travis gh-login` (authenticates against GitHub and stores the token)
* Add `travis gh-signature` (generates the signature used to verify Travis CI web hooks)
* Add `travis gh-upload` (upload one or more files to the repository)
* Add `travis gh-whoami` (displays the user you are logged in as on GitHub)
* Add `travis gh-write` (write stdin to the given path in the repository)

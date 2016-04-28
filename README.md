# Heroku Buildpack for executing and deploying Ember CLI apps as part of another app 

This buildpack's purpose is to install node and ember-cli to enable deploying
ember apps as part of another app. Say you've got a Laravel or a Rails app and
you need to deploy the ember build to the public files path. This is the
buildpack for you. It doesn't configure any webserver instances, just fetches
the necessary dependencies and executes the required ember build commands to
deploy your emberapp.

Forked from
[heroku-buildpack-ember-cli](https://github.com/tonycoco/ember-buildpack-ember-cli).

## Usage

Typically this will be included as an additional buildpack. So configure your
heroku app like normally, then add this top the *top* of the buildpack list.

    $ heroku buildpacks:add --app your-app-name --index 1 https://github.com/imipolexg/heroku-buildpack-ember-builder

## Configuration

You'll need to set a few environment variables fort his to work.

### Ember app root

You'll need to specifiy the relative path of your Emberapp within your git
repo. So if your emberapp's `package.json` is stored in `resources/emberapp/package.json` do:

    $ heroku config:set EMBER_ROOT="resources/emberapp"

### Build environment 

    $ heroku config:set EMBER_ENV=production

### Before and After Hooks

Have the buildpack run your own scripts before and after the `ember build` by
creating a `hooks/before_hook.sh` or `hooks/after_hook.sh` file in your Ember
CLI application:

    $ mkdir -p hooks

For a before build hook:

    $ touch hooks/before_hook.sh
    $ chmod +x hooks/before_hook.sh

For an after build hook:

    $ touch hooks/after_hook.sh
    $ chmod +x hooks/after_hook.sh

*See below for examples.*

#### Example Before Hook: Compass

[Compass](http://compass-style.org) can be installed using the before build hook. Create `hooks/before_hook.sh` and add the following script:

```bash
#!/usr/bin/env bash

export GEM_HOME=$build_dir/.gem/ruby/2.2.0
export PATH=$GEM_HOME/bin:$PATH

if test -d $cache_dir/ruby/.gem; then
  status "Restoring ruby gems directory from cache"
  cp -r $cache_dir/ruby/.gem $build_dir
  HOME=$build_dir gem update compass --user-install --no-rdoc --no-ri
else
  HOME=$build_dir gem install compass --user-install --no-rdoc --no-ri
fi

rm -rf $cache_dir/ruby
mkdir -p $cache_dir/ruby

if test -d $build_dir/.gem; then
  status "Caching ruby gems directory for future builds"
  cp -r $build_dir/.gem $cache_dir/ruby
fi
```

### Force Rebuilds

Sometimes it is necessary to rebuild NPM modules or Bower dependencies from
scratch.  This can become necessary when updating Ember or EmberCLI midway
through a project and cleaning the Bower and NPM caches doesn't always refresh
the cache in the Dyno during the next deployment.  In those cases, here is a
simple and clean way to force a rebuild.

To force a rebuild of NPM modules *and* Bower dependencies:

    heroku config:set REBUILD_ALL=true
    git commit -am 'rebuild' --allow-empty
    git push heroku master
    heroku config:unset REBUILD_ALL

To force a rebuild of just the NPM modules:

    heroku config:set REBUILD_NODE_PACKAGES=true
    git commit -am 'rebuild' --allow-empty
    git push heroku master
    heroku config:unset REBUILD_NODE_PACKAGES

To force a rebuild of Bower dependencies:

    heroku config:set REBUILD_BOWER_PACKAGES=true
    git commit -am 'rebuild' --allow-empty
    git push heroku master
    heroku config:unset REBUILD_BOWER_PACKAGES

### Caching

The Ember CLI buildpack caches your NPM and Bower dependencies by default. This
is similar to the [Heroku Buildpack for
Node.js](https://github.com/heroku/heroku-buildpack-nodejs). This makes typical
deployments much faster. Note that dependencies like
[`components/ember#canary`](http://www.ember-cli.com/#using-canary-build-instead-of-release)
will not be updated on each deploy.

To [purge the cache](https://github.com/heroku/heroku-repo#purge_cache) and
reinstall all dependencies, run:

    $ heroku plugins:install https://github.com/heroku/heroku-repo.git
    $ heroku repo:purge_cache -a APPNAME

## Troubleshooting

Clean your project's dependencies:

    $ npm cache clear
    $ bower cache clean
    $ rm -rf node_modules bower_components
    $ npm install --no-optional
    $ bower install

Be sure to save any Bower or NPM resolutions. Now, let's build your Ember CLI
application locally:

    $ ember build

Check your `git status` and see if that process has made any changes to your
application's code. Now, try your Heroku deployment again.

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## Contributors

This buildpack is a fork of
[heroku-buildpack-ember-cli](https://github.com/tonycoco/heroku-buildpack-ember-cli),
which it quite easy to hack this variation up. Thanks to everyone who worked on
that. 

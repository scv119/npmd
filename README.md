# npmd

distributed npm client.

[![build status](https://secure.travis-ci.org/dominictarr/npmd.svg)](http://travis-ci.org/dominictarr/npmd)


`npmd` is an alternative npm client that uses local replication and smart caching
improve performance by eliminating unnecessary network round-trips.
It is intended for use in the antipodes, via 3g, in airplanes, submarines, up trees, and in caves.
But it is still faster if you live in california too.

## install

install carefully.

``` js
npm install npmd@1 -g --carefully
```

## npmd@1

I've recently rewritten the heart of npmd,
adding [npmd-cache](https://github.com/dominictarr/npmd-cache)
and changing how [npmd-resolve](https://github.com/dominictarr/npmd-resolve) and
[npmd-install](https://github.com/dominictarr/npmd-install) retrive modules.

Syncing metadata was one of the best ideas in npmd@0, but also one of the most annoying.
Syncing is gone for now, but it will return soon as an option. npmd@1 is much more simple,
and, by using [Alan Gutierrez's](https://twitter.com/bigeasy) pure javascript database
[Locket](https://github.com/bigeasy/locket), npmd will not require a compiled node addon
and so will be easy to run even on windows. (this is still in development, so bear with us)

### npmd-cache

npmd keeps a cache of modules you have installed, but it works different to npm's cache.

npm keeps a cache of modules at versions, see `~/.npm/{module}/{version}` and of npm docs
that npm has downloaded `~/.npm/{module}/.cache.json`. Unfortunately, this does not work well
offline, because it does not cache modules that where installed via `git` or `http` urls.
(any module with a largish dependency tree is likely to have at least one of these)

npmd-cache works differently, it's cache is divided into two parts - the mutable side,
and the immutable side. The immutable side stores tarballs, and the mutable side stores module ids.
What is a module id? there are several ways to identify a module in npm. The best is
by it's name and version `{module}@{version}` but you can also install tarballs from
http or git urls (if they return something containing a package.json).

There is no standard way to tell wether a url refers to an immutable resource.
That means that the next time you request that url it may be give a different response.
Even `{module}@{version}` is _nearly_ immutable, but not completely, because module versions can be deleted.

However, code changes relatively slowly, and if you are working offline (freedom from distractions)
so npmd stores what tarball shasum a module id points to, so you can install that module again later.
Since the tarballs are identified by their shasum, so you can always grab an exact version.

### npmd-resolve

`npmd` splits out resolving a dependency tree from installing a dependency tree, but in npm these two things
are tangled together. `npmd-resolve` takes a module id, or `{module}@{versionRange}` and builds a json
object that represents all the modules which should be installed, and the shasums they should have.

The format of this object is compatible with the json generated by the `npm shrinkwrap` command,
which can be used to install exactly the same deps again. 

`npmd-resolve` in bundled inside `npmd`, but it can also be used as a standalone tool.

``` js
npm install npmd-resolve -g

npmd-resolve npmd > npmd.deps.json
```
this can then be fed into `npmd-install`

### npmd-install

`npmd-install` takes a dependency tree with shasums and extracts them into a node_modules folder.
it is bundled inside `npmd` or can be installed as a standalone tool.

``` js
npm install npmd-install -g

npmd-install < npmd.deps.json
```

note, since `npmd-install` and `npmd-resolve` may both write to the mutable database inside `npmd-cache`,
you cannot run them both at the same time. If you want to feed the output of one into the other, you must do so via a file.

``` js
npmd-resolve browserify > b; npmd-install < b; rm b
```

This will get fixed at some point, but for now the simplest is just to use `npmd install browserify`

## help

display help files

```
npmd help $command
```

## install

install a module. if the module's dependencies are in the cache,
then `npmd` will install without making a single network round trip!

```
npmd install browserify --greedy
```

`--greedy` is optional, if enabled, the dependency tree is flattened as much as possible.
so you have less duplication.

use `--global` to install a command globally.

## resolve

resolve all module versions required to install a given module.
will write json to stdout in the same format as npm-shrinkwrap. 

```
npmd resolve request
```

## License

MIT

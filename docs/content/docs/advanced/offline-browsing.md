---
title: How to offer offline browsing experience with Phenomic
---

_What if users that already visit your website once can access it when they are
offline, without a internet connection?_

Guess what, **Phenomic supports offline browsing out of the box.**
It's very easy to offer an offline experience to your users.
You just have to turn a flag on.

There are currently two different technologies to enable offline browsing.

## AppCache

AppCache is the oldest way to offer offline supports but it's a bit brutal since
you can only choose what to save when the website opens.
All modern browsers support AppCache, including IE 10.

- [Learn about AppCache](http://www.html5rocks.com/en/tutorials/appcache/beginner/)
- [AppCache browser support from caniuse](http://caniuse.com/#search=appcache)

## Service Worker

⚠️ **Service workers only works when using _HTTPS_, for security reasons.**
_Having modified network requests wide open to man in the middle attacks would
be really bad_.
An exception exists for ``http://localhost`` to help you during development.

Service Worker are a specific Web worker that allows more flexible behavior
than Appcache.
For example, you can easily cache a single HTML entry point with
CSS and JavaScript files at the start, and save for offline usage all other
requested content on demand.

- [Learn about Service Worker](http://www.html5rocks.com/en/tutorials/service-worker/introduction/)
- [Service worker browser support from caniuse](http://caniuse.com/#search=service-worker)

⚠️ **Notice**

> If you use AppCache and Service Worker on a page, browsers that don’t support
> Service Worker but do support AppCache will use that,
> and **browsers that support both will ignore the AppCache and let
> Service Worker take over**.
> - [from MDN](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers#Registering_your_worker)

### Network first, cache as fallback

⚠️ To always offer up to date and fresh content, we choose to use a
"network first" approach. Cache will only be used as fallback (eg: no internet
access).

## How to enable offline support for Phenomic?

To enable this feature, add an ``offline`` field to ``package.json`` under
``phenomic`` section.

```js
{
  // ...
  "phenomic": {
    "offline": true
  }
  // ...
}
```

Available values are:

### ``"offline": false``

Default value.
Disable `offline` support.

### ``"offline": true``

Use default configuration (see below) which will cache all resources
generated by the build process
(except the ``assets`` folder, which will only be cached **on demand**).

### ``"offline": { /* Object */ }``

An object is accepted and must contain 3 keys.
Please be aware that you don't need to define all of 3 keys.
They will be defined with their default values.

#### `serviceWorker: boolean = true`

Enable/Disable Service Worker separately

#### `appcache: boolean = true | { [key]: boolean}`

Enable/Disable AppCache separately. You can also provide an object to choose
what to cache from the ``cachePatterns`` (see below).
Here is the default value:

```js
appcache: {
  // By default, this only cache the onInstall + afterInstall patterns
  onInstall: true,
  afterInstall: true,
  // onDemand: cannot be emulated (so cached) with appcache
},
```

#### `cachePatterns: { [key]: Array<string> }`:

By default cache all content generated by the build,
except HTML files (which are useless if JavaScript is on - a single HTML
is cached and is used as a bootstrap for the JavaScript client code).

You can define your own glob patterns to match files to cache.
Here is the default value

```js
cachePatterns: {
  // cache the App Shell when Service Worker is installed
  onInstall: [ "index.html", "phenomic.*" ],
  // cache all known content after the Service worker has been installed
  afterInstall: [ "**", ":assets:" ],
  // cache all other content on demand
  onDemand: [ ],
  // excludes dotfiles, sourcemaps files and HTML files (only one is enough for
  // offline usage)
  excludes: [ "**/.*", "**/*.map", "**/*.html" ],
},
```

⚠️ **Some special strings**

- ``:assets:`` can be used to catch all assets in your ``assets`` folder
  (not handled by webpack).
- ``:rest:`` can be used to catch all unused files (in other patterns).

Phenomic uses [globby](https://www.npmjs.com/package/globby) for matching files
in ``dist`` folder.

Checkout [globby documentation for more information](https://www.npmjs.com/package/globby)

When you will build your website and you may notice some new files in ``dist``
folder such as ``manifest.appcache``, ``sw.js``
(depending on the options you provided).

⚠️ **Note**: AppCache support will not be enabled in development mode to
avoid the pain it can cause.

---

## FAQ

### How can I provide my own Service Worker logic?

Under the hood, Phenomic uses the webpack
[offline-plugin](https://github.com/NekR/offline-plugin), so you can just use
this one directly in your webpack configuration with your own options.
For even more flexibility, you can check
[sw-precache](https://github.com/GoogleChrome/sw-precache)
or
[sw-toolbox](https://github.com/GoogleChrome/sw-toolbox)


### Can you show me some useful glob patterns ?

Here are some useful patterns that should covers most use cases
(along with the default patterns)

#### Cache everything initially

This is the default behavior, just use ``"offline": true,``.

## Only cache stuff on demand

```js
cachePatterns: {
  afterInstall: [],
  onDemand: [ "**", ":assets:" ],
  // for other keys, default values will be used
},
```

#### Only cache the "App Shell"

```js
cachePatterns: {
  // app shell is cached by default onInstall
  // so we just need to remove patterns for others keys
  afterInstall: [],
  onDemand: [],
},
```

#### Only cache App Shell and markdown content on install, not bundled assets

```js
cachePatterns: {
  afterInstall: [ "**/*.json" ], // reminder: markdown are transformed as json files
  onDemand: [ ":rest:", ":assets:" ], // will catch all others results on demand
  // for other keys, default values will be used
},
```

# Service Worker Example

![offline first poster](https://github.com/wildhaber/offline-first-sw/blob/master/offline-first-sw.jpg "offline first poster")

## Features

 - Custom offline page
 - Custom 404 page
 - Cache blacklist rules for resources that always comes from network
 - Individual TTL settings for different file extensions for a rolling cache refresh respecting offline-first 
 - Easy to customize for your specific needs
 - Cleanup legacy cache on update
 - Automatic Caching of relative content defined with `<link rel='index|next|prev|prefetch'>`

## Installation & Usage

### Install the Service Worker

Simply copy [**sw.js**](https://github.com/wildhaber/offline-first-sw/blob/master/sw.js) in your root directory:

```shell
# simple wget-snippet or do it manually
# cd /your-projects-root-directory/
wget https://raw.githubusercontent.com/wildhaber/offline-first-sw/master/sw.js
```

and launch the Service Worker with the following snippet:

```js
<script>
    if('serviceWorker' in navigator) {

        /**
         * Define if <link rel='next|prev|prefetch'> should
         * be preloaded when accessing this page
         */
        const PREFETCH = true;

        /**
         * Define which link-rel's should be preloaded if enabled.
         */
        const PREFETCH_LINK_RELS = ['index','next', 'prev', 'prefetch'];

        /**
         * prefetchCache
         */
        function prefetchCache() {
            if(navigator.serviceWorker.controller) {

                let links = document.querySelectorAll(
                    PREFETCH_LINK_RELS.map((rel) => {
                        return 'link[rel='+rel+']';
                    }).join(',')
                );

                if(links.length > 0) {
                    Array.from(links)
                        .map((link) => {
                            let href = link.getAttribute('href');
                            navigator.serviceWorker.controller.postMessage({
                                action : 'cache',
                                url : href,
                            });
                        });
                }


            }
        }

        /**
         * Register Service Worker
         */
        navigator.serviceWorker
            .register('/sw.js', { scope: '/' })
            .then(() => {
                console.log('Service Worker Registered');
            });

        /**
         * Wait if ServiceWorker is ready
         */
        navigator.serviceWorker
            .ready
            .then(() => {
                if(PREFETCH) {
                    prefetchCache();
                }
            });

    }
</script>
```

For AMP-Pages use:

```html
<script async custom-element="amp-install-serviceworker" src="https://cdn.ampproject.org/v0/amp-install-serviceworker-0.1.js"></script>
```

```html
<amp-install-serviceworker
	src="/sw.js"
	data-iframe-src="https://ampbyexample.com/sw.html"
	layout="nodisplay">
</amp-install-serviceworker>
```

Get more information for [AMP-Install-Serviceworker](https://ampbyexample.com/components/amp-install-serviceworker/) at [ampbyexample.com](https://ampbyexample.com/components/amp-install-serviceworker/)

### Install the manifest.json

The manifest is an important part but not required. Read more about [manifest.json on Google Developers](https://developers.google.com/web/fundamentals/getting-started/codelabs/your-first-pwapp/).

Copy the [**manifest.json**](https://github.com/wildhaber/offline-first-sw/blob/master/manifest.json) into your root directory and link it within your markup.

```shell
# simple wget-snippet or do it manually
# cd /your-projects-root-directory/
wget https://raw.githubusercontent.com/wildhaber/offline-first-sw/master/manifest.js
```

And create the following link-snippet within your page's `<head>`-Section:

```html
<link rel="manifest" href="/manifest.json">
```

---

## Configuration

### Service Worker

#### CACHE_VERSION `{ number }`

```js
const CACHE_VERSION = 1;
```

The CACHE_VERSION is important when you update your service-worker. The cache-location will be updated and the old legacy cache is getting deleted.

#### BASE_CACHE_FILES  { array }

```js
const BASE_CACHE_FILES = [
    '/style.css',
    '/script.js',
    '/search.json',
    '/manifest.json',
    '/favicon.png',
];
```
Define files that in this list which always needs to be cached from the beginning.

#### OFFLINE_CACHE_FILES  { array }

```js
const OFFLINE_CACHE_FILES = [
    '/style.css',
    '/script.js',
    '/offline/index.html',
];
```

Define files necessary for your offline page.

#### NOT_FOUND_CACHE_FILES { array }

```js
const NOT_FOUND_CACHE_FILES = [
    '/style.css',
    '/script.js',
    '/404.html',
];
```

Define files necessary for your 404 page.

#### OFFLINE_PAGE { string }

```js
const OFFLINE_PAGE = '/offline/index.html';
```

Deliver this page when the user is offline and the page is not cached already.


#### NOT_FOUND_PAGE { string }

```js
const NOT_FOUND_PAGE = '/404.html';
```

Deliver this page when the user lands on a page with `Status-Code >= 400`.

#### MAX_TTL { object }

```js
const MAX_TTL = {
    '/': 3600,
    html: 3600,
    json: 86400,
    js: 86400,
    css: 86400,
};
```

This is a key-value mapping file extensions with a certain maximal time-to-live (**in seconds** not milliseconds). This is how long the chache will be active until the page is getting refreshed from the network.

Not specified extensions with stay cached until a SW-Update.

```js
// 60 = 1 minute
// 3600 = 1 hour
// 86400 = 1 day
// 604800 = 1 week
// 2592000 = 30 days (~ 1 month)
// 31536000 = 1 year
```

#### CACHE_BLACKLIST { array}

```js
const CACHE_BLACKLIST = [
    (str) => {
	    // str = URL of the resource
	    // apply this rule when you do not want to cache external files
        return !str.startsWith('https://yourwebsite.tld');
    },
];
```

This is a list of functions a `URL` gets checked against if it should be cached or not. If one method returns `true` this resource will not be cached.

#### SUPPORTED_METHODS { array }

```js
const SUPPORTED_METHODS = [
    'GET',
];
```

Only methods in this list will be cached.

---

## MIT License

Copyright (c) 2017 Raphael Wildhaber, https://github.com/wildhaber/

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

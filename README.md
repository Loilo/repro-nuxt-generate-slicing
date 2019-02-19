# Reproduction of [nuxt/nuxt.js#4982](https://github.com/nuxt/nuxt.js/issues/4982)

> Using `npm run generate`, Nuxt.js slices HTML content when nested routes are present.

## Reproduction Steps
Install dependencies:
```bash
npm install
```

Generate the static site:
```bash
npm run generate
```

Take a look at the generated `/dist/index.html`, it's probably broken (i.e. contains invalid HTML).

## Investigation
I'm not able to exactly hunt this issue down, but here's what I've found out so far.

### Prerequisites
Take these notes with a grain of salt, as this was basically a blackbox test. However, I tried to be as careful as possible with the following assumptions.

* The broken build only affects nested routes:
  * It only occurs with a `/pages/index.vue`, not with any other pages.
  * It only occurs if an according `/pages/index` directory for nested routes exists.
  * It only occurs if that `index` directory contains an `index.vue`. The content of that file is irrelevant to the outcome.
* Building behavior is kind of inconsistent. I assume that some caching is behind this which I'm not able to bypass.
  * When `npm run generate` directory produced invalid HTML once, repeating the command will keep generating the same invalid HTML as long as the source doesn't change.
  * Very rarely, `npm run generate` will create *valid* HTML after source code in the `/pages` code changed.
    * I have no idea which kind of changes lead to this behavior.
    * Also, the correct HTML will *only* be emitted once or twice, any further builds will output the same invalid HTML.
* I reduced the `nuxt.config.js` to the bare minimum, however I could not detect any impact of the Nuxt configuration so far.

### Build Evolution
I was lucky enough to catch one of the mentioned "correct builds" and compare it to the builds generated afterwards. The according `/pages/index.vue` was:

```vue
<template>
  <div>
    <h1>This is the only page on this site.</h1>
  </div>
</template>
```

The results roughly confirm what I assumed — it looks like some generated HTML is partly overwritten with other HTML, but not completely:

#### `/dist/index.html` after the first `npm run generate`, 1042 bytes

```html
<!doctype html>
<html data-n-head-ssr data-n-head="">
  <head data-n-head="">
    <title data-n-head="true"></title><link rel="preload" href="/_nuxt/114379b8f0c90aceb188.js" as="script"><link rel="preload" href="/_nuxt/b0dfa225c967768dd7fb.js" as="script"><link rel="preload" href="/_nuxt/10469b7c8ad18cdd633a.js" as="script"><style data-vue-ssr-id="17cfdfa9:0">.nuxt-progress{position:fixed;top:0;left:0;right:0;height:2px;width:0;opacity:1;transition:width .1s,opacity .4s;background-color:#000;z-index:999999}.nuxt-progress.nuxt-progress-notransition{transition:none}.nuxt-progress-failed{background-color:red}</style>
  </head>
  <body data-n-head="">
    <div data-server-rendered="true" id="__nuxt"><!----><div id="__layout"><!----></div></div><script>window.__NUXT__={layout:"default",data:[{},{}],error:null,serverRendered:!0}</script><script src="/_nuxt/114379b8f0c90aceb188.js" defer></script><script src="/_nuxt/b0dfa225c967768dd7fb.js" defer></script><script src="/_nuxt/10469b7c8ad18cdd633a.js" defer></script>
  </body>
</html>

```

#### `/dist/index.html` after the second `npm run generate`, 1220 bytes

```html
<!doctype html>
<html data-n-head-ssr data-n-head="">
  <head data-n-head="">
    <title data-n-head="true"></title><link rel="preload" href="/_nuxt/114379b8f0c90aceb188.js" as="script"><link rel="preload" href="/_nuxt/b0dfa225c967768dd7fb.js" as="script"><link rel="preload" href="/_nuxt/10469b7c8ad18cdd633a.js" as="script"><link rel="preload" href="/_nuxt/188f4cf10fd04e26664b.js" as="script"><style data-vue-ssr-id="17cfdfa9:0">.nuxt-progress{position:fixed;top:0;left:0;right:0;height:2px;width:0;opacity:1;transition:width .1s,opacity .4s;background-color:#000;z-index:999999}.nuxt-progress.nuxt-progress-notransition{transition:none}.nuxt-progress-failed{background-color:red}</style>
  </head>
  <body data-n-head="">
    <div data-server-rendered="true" id="__nuxt"><!----><div id="__layout"><div><h1>This is the only page on this site.</h1></div></div></div><script>window.__NUXT__={layout:"default",data:[{},{}],error:null,serverRendered:!0}</script><script src="/_nuxt/114379b8f0c90aceb188.js" defer></script><script src="/_nuxt/188f4cf10fd04e26664b.js" defer></script><script src="/_nuxt/b0dfa225c967768dd7fb.js" defer></script><script src="/_nuxt/10469b7c8ad18cdd633a.js" defer></script>
  </body>
</html>

```

#### `/dist/index.html` after the third (and every further) `npm run generate`, 1220 bytes

```html
<!doctype html>
<html data-n-head-ssr data-n-head="">
  <head data-n-head="">
    <title data-n-head="true"></title><link rel="preload" href="/_nuxt/114379b8f0c90aceb188.js" as="script"><link rel="preload" href="/_nuxt/b0dfa225c967768dd7fb.js" as="script"><link rel="preload" href="/_nuxt/10469b7c8ad18cdd633a.js" as="script"><style data-vue-ssr-id="17cfdfa9:0">.nuxt-progress{position:fixed;top:0;left:0;right:0;height:2px;width:0;opacity:1;transition:width .1s,opacity .4s;background-color:#000;z-index:999999}.nuxt-progress.nuxt-progress-notransition{transition:none}.nuxt-progress-failed{background-color:red}</style>
  </head>
  <body data-n-head="">
    <div data-server-rendered="true" id="__nuxt"><!----><div id="__layout"><!----></div></div><script>window.__NUXT__={layout:"default",data:[{},{}],error:null,serverRendered:!0}</script><script src="/_nuxt/114379b8f0c90aceb188.js" defer></script><script src="/_nuxt/b0dfa225c967768dd7fb.js" defer></script><script src="/_nuxt/10469b7c8ad18cdd633a.js" defer></script>
  </body>
</html>
88f4cf10fd04e26664b.js" defer></script><script src="/_nuxt/b0dfa225c967768dd7fb.js" defer></script><script src="/_nuxt/10469b7c8ad18cdd633a.js" defer></script>
  </body>
</html>

```

### Observations
As you can see:

* The first build is valid HTML but does not contain the actual content of the page.
* The second build is valid HTML and seems to deliver the expected output.
* The third build contains the complete first build and is filled up with the contents of the second build.
  That's hard to describe in words, so I'll use JavaScript to express it more precisely:
  ```js
  content_3 = content_1 + content_2.substr(content_1.length)
  ```
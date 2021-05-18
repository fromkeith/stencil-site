---
title: Hydrate App
description: Hydrate App
url: /docs/hydrate-app
contributors:
  - adamdbradley
  - dgautsch
  - bitflower
---

# Hydrate App

The hydrate app is a bundle of your same components, but compiled so they can be hydrated on a NodeJS server, and generate HTML. This is the same NodeJS app which prerendering uses internally, and the same one which Angular Universal SSR uses for Ionic. Like Stencil components, the hydrate app itself is not restricted to one framework.

_Note that Stencil does **NOT** use Puppeteer for SSR or prerendering._

## How to Use the Hydrate App

Server side rendering (SSR) can be accomplished in a similar way to prerendering. Instead of using the `--prerender` CLI flag, you can add `{ type: 'dist-hydrate-script' }` to your `stencil.config.js`. This will generate a `hydrate` app in your root project directory that can be imported and used by your Node server.

After publishing your component library, you can import the hydrate app into your server's code like this:

```javascript
import { hydrateDocument } from 'yourpackage/hydrate';
```

The app comes with two functions, `hydrateDocument` and `renderToString`. An advantage to `hydrateDocument` is that it can take an existing [document](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDocument) which was already parsed. The `renderToString` function instead inputs a raw html string, and does the parsing to a document.

**hydrateDocument**: You can use `hydrateDocument` as a part of your server's response logic before serving the web page. `hydrateDocument` takes two arguments, a [document](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDocument) and a config object. The function returns a promise with the hydrated results. The resulting html can be parsed from the `html` property in the returned results.

*Example taken from Ionic Angular server*

 ```javascript
import { hydrateDocument } from 'yourpackage/hydrate';

export function hydrateComponents(doc) {
  return hydrateDocument(doc)
    .then((hydrateResults) => {
      // execute logic based on results
      console.log(hydrateResults.html);
      return hydrateResults;
    });
}
```

#### hydrateDocument Options

| Property | Type | Description |
|----------|------|-------------|
| `canonicalUrl` | string | Sets the `href` attribute on the `<link rel="canonical">` tag within the `<head>`. If the value is not defined it will ensure a canonical link tag is no included in the `<head>`. |
| `constrainTimeouts` | boolean | Constrain `setTimeout()` to 1ms, but still async. Also only allows `setInterval()` to fire once, also constrained to 1ms. Defaults to `true`. |
| `clientHydrateAnnotations` | boolean | Include the HTML comments and attributes used by the clientside JavaScript to read the structure of the HTML and rebuild each component. Defaults to `true`. |
| `cookie` |  string | Sets `document.cookie` |
| `direction` |  string | Sets the `dir` attribute on the top level `<html>`. |
| `excludeCommponents` |  string[] | Component tag names listed here will not be prerendered, nor will hydrated on the clientside. Components listed here will be ignored as custom elements and treated no differently than a `<div>`. |
| `language` |  string | Sets the `lang` attribute on the top level `<html>`. |
| `maxHydrateCount` |  number | Maximum number of components to hydrate on one page. Defaults to `300`. |
| `referrer` |  string |  Sets `document.referrer` |
| `removeScripts` |  boolean | Removes every `<script>` element found in the `document`. Defaults to `false`. |
| `removeUnusedStyles` |  boolean | Removes CSS not used by elements within the `document`. Defaults to `true`. |
| `resourcesUrl` |  string | The url the runtime uses for the resources, such as the assets directory. |
| `runtimeLogging` |  boolean | Prints out runtime console logs to the NodeJS process. Defaults to `false`. |
| `staticComponents` |  string[] | Component tags listed here will only be prerendered or serverside-rendered and will not be clientside hydrated. This is useful for components that are not dynamic and do not need to be defined as a custom element within the browser. For example, a header or footer component would be a good example that may not require any clientside JavaScript. |
| `timeout` | number |  The amount of milliseconds to wait for a page to finish rendering until a timeout error is thrown. Defaults to `15000`. |
| `title` | string |  Sets `document.title`. |
| `url` | string |  Sets `location.href` |
| `userAgent` | string |  Sets `navigator.userAgent` |

**renderToString**: The hydrate app also has a `renderToString` function that allows you to pass in an html string that also returns a promise of `HydrateResults`. The second parameter is a config object that can alter the output of the markup. Like `hydrateDocument`, the resulting string can be parsed from the `html` property.

*Example taken from Ionic Core*

```javascript
const results = await hydrate.renderToString(srcHtml, {
  prettyHtml: true,
  removeScripts: true
});

console.log(results.html);
```

#### renderToString Options

See `hydrateDocument Options` above as well.

| Property | Type | Description |
|----------|------|-------------|
| `approximateLineWidth` | number | Sets an approximate line width the HTML should attempt to stay within. Note that this is "approximate", in that HTML may often not be able to be split at an exact line width. Additionally, new lines created is where HTML naturally already has whitespce, such as before an attribute or spaces between words. Defaults to `100`. |
| `prettyHtml` | boolean | Format the HTML in a nicely indented format. Defaults to `false`. |
| `removeAttributeQuotes` | boolean | Remove quotes from attribute values when possible. Defaults to `true`. |
| `removeBooleanAttributeQuotes` | boolean | Remove the `=""` from standardized `boolean` attributes, such as `hidden` or `checked`. Defaults to `true`. |
| `removeEmptyAttributes` | boolean | Remove these standardized attributes when their value is empty: `class`, `dir`, `id`, `lang`, and `name`, `title`. Defaults to `true`. |
| `removeHtmlComments` | boolean | Remove HTML comments. Defaults to `true`. |
| `afterHydrate` | `(document: any): any` `\| Promise<any>` | Runs after the `document` has been hydrated. |
| `beforeHydrate` | `(document: any): any` `\| Promise<any>` | Runs before the `document` has been hydrated. |

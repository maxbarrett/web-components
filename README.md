# Native web components - no polyfills

To get the HTML imports working in the browser we need an HTTP server running:
```sh
python -m SimpleHTTPServer;
```

View index.html at:
```sh
http://localhost:8000
```


### HTML Imports

Test for HTML import browser support:
```sh
function supportsImports() {
  return 'import' in document.createElement('link');
}
```

Imports from the same URL are retrieved and parsed once, so the script in an import is only executed the first time.

An import link doesn't include the content at that location. It tells the browser to fetch the document to use later – Recommended to put imports in the `<head>`. 

While scripts execute at import time, stylesheets, markup, and other resources need to be added to the main page explicitly. 

An import can access its own DOM and/or the DOM of the page that's importing it.

Javascript in the import is executed in the context of the window that contains the importing document. So window.document refers to the main page document.

Imports do not block parsing of the main page. However, scripts inside them are processed in order = defer-like behavior while maintaining proper script order.

Wrapping content in a `<template>` gives benefit of making content inert until used – scripts don't run until the template is added to the DOM.

[Vulcanize](https://github.com/Polymer/vulcanize) is an npm build tool from the Polymer team that recursively flattens a set of HTML Imports into a single file. Think of it as a concatenation build step for Web Components.

Imports block rendering of the main page. This is similar to `<link rel="stylesheet">`. The reason the browser blocks rendering on stylesheets is to minimize FOUC – Imports behave similarly because they can contain stylsheets.

Imports don't block parsing of the main page. Scripts inside imports are processed in order but don't block the importing page.

To be completely asynchronous and not block the parser or rendering, use the `async` attribute:
```sh
<link rel="import" href="/path/to/import_that_takes_5secs.html" async>
```

Scripts in an import are processed in order, but do not block the main document parsing.

HTML Imports parallelise HTML parsing - first time the browser has been able to run two (or more) HTML parsers in parallel.

To load content from another domain, the import location needs to be CORS-enabled.

Notes taken from [HTML5 Rocks](http://www.html5rocks.com/en/tutorials/webcomponents/imports/)


#Web components
...




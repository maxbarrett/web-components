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

Browser support test for HTML imports:
```sh
function supportsImports() {
  return 'import' in document.createElement('link');
}
```

Add an import to the page:
```sh
<link rel="import" href="/path/to/imports/stuff.html">
```


Imports from the same URL are retrieved and parsed once, so the script in an import is only executed the first time.

An import link tells the browser to fetch the document for use later – put them in the `<head>`. 

While scripts execute at import time, stylesheets, markup, and other resources need to be added to the main page explicitly. 

An import can access its own DOM and/or the DOM of the page that's importing it.

Javascript in the import is executed in the context of the window that contains the importing document – window.document refers to the main page.

Imports do not block parsing of the main page – Giving parallelised HTML parsing for the first time.

Imports block rendering of the main page, similar to `<link rel="stylesheet">` to minimize FOUC.

Scripts inside them are processed in order, but do not block the main document parsing. So you get defer-like behavior while maintaining proper script order.

Wrapping content in a `<template>` gives benefit of making content inert until used – scripts don't run until the template is added to the DOM.

To be completely asynchronous and not block the parser or rendering, use the `async` attribute:
```sh
<link rel="import" href="/path/to/import_that_takes_5secs.html" async>
```

[Vulcanize](https://github.com/Polymer/vulcanize) is an npm build tool that recursively flattens a set of HTML Imports into a single file - a concatenation build step for Web Components.

To load content from another domain, the import location needs to be CORS-enabled.

Notes taken from [HTML5 Rocks](http://www.html5rocks.com/en/tutorials/webcomponents/imports/)


### HTML `<template>` tag

Browser support test for HTML templates:
```sh
function supportsTemplate() {
  return 'content' in document.createElement('template');
}
```


Content is effectively inert until activated
Script doesn't run, images don't load, audio doesn't play until the template is used
Content is considered not to be in the document
Templates can be placed anywhere inside of <head>, <body>, or <frameset>

To use a template, clone it into the document
```sh
var t = document.querySelector('#mytemplate');
var clone = document.importNode(t.content, true);
document.body.appendChild(clone);
```

Outer templates will not activate inner templates. Nested templates require that their children also be manually activated.


http://www.html5rocks.com/en/tutorials/webcomponents/template/




### Custom elements

http://www.html5rocks.com/en/tutorials/webcomponents/customelements/

createdCallback - fires when an instance of the element is created
attachedCallback - fires when injected into the document
detachedCallback - fires when removed from the document
attributeChangedCallback - fires when an attribute is added, removed, or changed


### Shadow DOM

http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/

http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom-201/

http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom-301/
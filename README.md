# Native web components - no polyfills

To get the HTML imports working in the browser we need an HTTP server running:
```sh
python -m SimpleHTTPServer
```

View index.html at:
```sh
http://localhost:8000
```





## HTML Imports

Browser support test for HTML imports:
```sh
function supportsImports() {
  return 'import' in document.createElement('link');
}
```

An import link tells the browser to fetch the document for use later. Recommended to put in `<head>`:
```sh
<link rel="import" href="/path/to/import.html">
```

To access the content of an import, use the link element's `.import` property:
```sh
var content = document.querySelector('link[rel="import"]').import;
```

Imports from the same URL are retrieved and parsed once, hence an import's script is only executed the first time.

Scripts execute at import time, unless you wrap them in a `<template>` tag, then they run when added to the DOM.

Stylesheets, markup, and other resources need to be added to the main page explicitly. 

An import can access its own DOM and/or the DOM of the page that's importing it.

Javascript in the import shares the window of the importing document – window.document refers to the main page.

Imports do not block parsing of the main page, the browser can parallelise HTML parsing.

Scripts inside imports are processed in order, giving defer-like behavior with proper script order.

Imports do block rendering of the main page, similar to `<link rel="stylesheet">` to minimize FOUC.

To be completely asynchronous (not block the parser or rendering) use `async` attribute:
```sh
<link rel="import" href="/path/to/large/import.html" async>
```

[Vulcanize](https://github.com/Polymer/vulcanize) is an npm build tool that recursively flattens a set of HTML Imports into a single file - a concatenation build step for Web Components.

To load content from another domain the import location needs to be CORS-enabled.

Notes taken from [HTML5 Rocks](http://www.html5rocks.com/en/tutorials/webcomponents/imports/)







## HTML `<template>` tag

Browser support test for HTML templates:
```sh
function supportsTemplate() {
	return 'content' in document.createElement('template');
}
```

Content is inert until activated - scripts don't run, images don't load & audio doesn't play until the template is cloned into the main document:
```sh
var temp = document.querySelector('#mytemplate');
var clone = document.importNode(temp.content, true); // a deep clone
document.body.appendChild(clone);
```

Best practice to append template content to a shadow root.

There's no way to 'preload' assets of a template for both server and client.

Outer templates will not activate inner templates – each must be manually activated individially.

Taken from [HTML5 Rocks](http://www.html5rocks.com/en/tutorials/webcomponents/template/)







## Shadow DOM
Browser support test for Shadow DOM:
```sh
function supportsShadowDom() {
	return !!HTMLElement.prototype.createShadowRoot;
}
```
To give a document element a shadow root:
```sh
var shadow = document.querySelector('.myElement').createShadowRoot();
```
That element's DOM subtree is then encapsulated/hidden.

If a `<template>` is cloned into the shadow root element the encapsulated subtree can be projected through the `<template>`'s `<content>` element:

```sh
<template id="template">
	<style>
		…
	</style>
	Hi, my name is: 
	<div class="name">
		<content><!-- .myElement subtree appears here --></content>
	</div>
</template>
```

Markup, CSS & JS outside of the `<content>` tags are an ideal place to store presentation details.

The `select` attribute can control what a `content` element projects, there can be multiple.

If you have a document containing:
```sh
<div class="first">Max</div>
<div>Barrett</div>
<div class="email">max@max.com</div>
```

and a shadow root using CSS selectors to select specific content:
```sh
<div style="color: red;">
	<content select=".first"></content>
</div>
<div style="color: yellow;">
	<content select="div"></content>
</div>
<div style="color: blue;">
	<content select=".email"></content>
</div>
```

`Max` will be red but both `Barrett` & `max@max.com` will be yellow – Elements are projected on first match, not specificity (like CSS). As `div` is matched before `.email` the email address is yellow, not blue.

`select` can only select elements which are immediate children of the host node. Descendants cannot be selected e.g.select="table tr".

If there is no match, the content will be hidden/encapsulated inside the shadow root.


`:host` allows you to select and style the element hosting a shadow tree.

Rules in the parent page have higher specificity than `:host` rules defined in the element, but lower specificity than a style attribute defined on the host element.

```sh
:host(<selector>) {
	... 
}

:host(:hover) {
	...
}

// matches the host element if it or any of its ancestors matches <selector>
:host-context(<selector>) {
	...
}
```


Use multiple shadow on one shadow host
Use nested shadows for encapsulation
Architect your page using Model-Driven Views (MDV) and Shadow DOM



Taken from HTML5 Rocks [[1](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/)], [[2](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom-201/)] & [[3](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom-301/)]









## Custom elements

Auto register custom-elements.

createdCallback - fires when an instance of the element is created
attachedCallback - fires when injected into the document
detachedCallback - fires when removed from the document
attributeChangedCallback - fires when an attribute is added, removed, or changed

Taken from [HTML5 Rocks](http://www.html5rocks.com/en/tutorials/webcomponents/customelements/)

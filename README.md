#Native web components - no polyfills

Get an HTTP server running at `http://localhost:8000`
```sh
python -m SimpleHTTPServer
```


##HTML Imports
```javascript
function supportsImports() {
  return 'import' in document.createElement('link');
}
```

An import link tells the browser to fetch the document for use later. Recommended to put in `<head>`:
```html
<link rel="import" href="/path/to/import.html">
```

To access the content of an import, use the link element's `.import` property:
```javascript
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
```html
<link rel="import" href="/path/to/large/import.html" async>
```

[Vulcanize](https://github.com/Polymer/vulcanize) is an npm build tool that recursively flattens a set of HTML Imports into a single file - a concatenation build step for Web Components.

To load content from another domain the import location needs to be CORS-enabled.

Notes taken from [HTML5 Rocks](http://www.html5rocks.com/en/tutorials/webcomponents/imports/)







## HTML `<template>` tag
```javascript
function supportsTemplate() {
	return 'content' in document.createElement('template');
}
```

Content is inert until activated - scripts don't run, images don't load & audio doesn't play until the template is cloned into the main document:
```javascript
var temp = document.querySelector('#mytemplate');
var clone = document.importNode(temp.content, true); // a deep clone
document.body.appendChild(clone);
```

(Best practice to append template content to a shadow root) TODO?
Markup, CSS & JS outside of the `<content>` tags are an ideal place to store presentation details.

There's no way to 'preload' assets of a template for both server and client.

Outer templates will not activate inner templates – each must be manually activated individially.

Taken from [HTML5 Rocks](http://www.html5rocks.com/en/tutorials/webcomponents/template/)







## Shadow DOM
```javascript
function supportsShadowDom() {
	return !!HTMLElement.prototype.createShadowRoot;
}
```

Give an element a shadow root to encapsulate it's DOM subtree:
```javascript
var shadow = document.querySelector('.myElement').createShadowRoot();
```

Cloning a `<template>` (method above) into a shadow root element projects that element's DOM subtree through the `<content>` tag:

```html
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

The subtree retains styles from the main document but can also be styled from inside the shadow tree using the ::content pseudo element.

The `content` element has a `select` attribute that can control what is projected:

Document:
```html
<div class="first">Max</div>
<div>Barrett</div>
<div class="email">max@max.com</div>
```

Shadow root using CSS selectors:
```html
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

`Max` = red
`Barrett` & `max@max.com` = yellow
Elements are projected on first match, not by specificity like CSS.

`select` can only select immediate children of the host node. Descendants cannot be selected eg: select="table tr".

If there is no match, the content will be hidden/encapsulated inside the shadow root.

Rules in the parent page have higher specificity than `:host` rules defined in the element, but lower specificity than a style attribute defined on the host element.

```sass
// style the element hosting a shadow tree.
:host(<selector>) {
	... 
}

// matches the host element if it or any of its ancestors matches <selector>
:host-context(<selector>) {
	...
}

// styling shadow DOM from outside:
#myId::shadow span { color: red; }

// /deep/ ignores all shadow boundaries – when nesting custom elements:
tab-group /deep/ tab-panel {
  ...
}
```

Accessing deep shadow DOM elements with JavaScript:
```javascript
// One way:
document.querySelector('x-tabs').shadowRoot
        .querySelector('x-panel').shadowRoot
        .querySelector('#foo');

// Better way:
document.querySelector('tab-group::shadow tab-panel::shadow .elem');
```


Taken from HTML5 Rocks [[1](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/)], [[2](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom-201/)] & [[3](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom-301/)]









## Custom elements

Auto register custom-elements.

createdCallback - fires when an instance of the element is created
attachedCallback - fires when injected into the document
detachedCallback - fires when removed from the document
attributeChangedCallback - fires when an attribute is added, removed, or changed

Taken from [HTML5 Rocks](http://www.html5rocks.com/en/tutorials/webcomponents/customelements/)

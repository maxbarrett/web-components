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

An import link tells the browser to fetch the document for use later – put in `<head>`
```html
<link rel="import" href="/path/to/import.html">
```

Use `.import` property to access content:
```javascript
var content = document.querySelector('link[rel="import"]').import;
```

Imports from the same URL are retrieved and parsed once.

Scripts execute at import time, unless they are wrapped in `<template>`, then they run when added to the DOM.

Stylesheets, markup, and other resources need to be added to the main page explicitly. 

An import can access its own DOM and the DOM of the importing page.

`window.document` refers to the importing page.

Imports do not block parsing of the main page – browser parallelises HTML parsing.

Scripts inside imports are processed in order – defer-like behavior with proper script order.

Imports block rendering of the main page – similar to `<link rel="stylesheet">` to minimize FOUC.

Use `async` attribute to not block rendering:
```html
<link rel="import" href="/path/to/large/import.html" async>
```

[Vulcanize](https://github.com/Polymer/vulcanize) recursively flattens imports into a single file.

Imports from another domain need to be CORS-enabled.

Notes taken from [HTML5 Rocks](http://www.html5rocks.com/en/tutorials/webcomponents/imports/)







## HTML `<template>` tag
```javascript
function supportsTemplate() {
	return 'content' in document.createElement('template');
}
```

Content – scripts/images/audio – are inert until template is cloned into main document:
```javascript
var temp = document.querySelector('#mytemplate');
var clone = document.importNode(temp.content, true); // a deep clone
document.body.appendChild(clone);
```

No way to 'preload' `<template>` assets – for both server and client.

Outer templates do not activate inner templates – each must be manually activated individially.

A template's content is often cloned into a shadow root, see below.

Taken from [HTML5 Rocks](http://www.html5rocks.com/en/tutorials/webcomponents/template/)







## Shadow DOM
```javascript
function supportsShadowDom() {
	return !!HTMLElement.prototype.createShadowRoot;
}
```

Give an element a shadow root and encapsulate it's DOM subtree:
```javascript
var shadow = document.querySelector('.myElement').createShadowRoot();
```

Cloning a `<template>` (as above) into an element's shadow root projects that element's DOM subtree through the `<content>` tag:

```html
<template id="template">
	<!-- Store presentation details outside of the `<content>` tags. -->
	<style>
		…
	</style>
	Hi, my name is: 
	<div class="name">
		<content><!-- .myElement subtree appears here --></content>
	</div>
</template>
```

`<template>` has a `select` attribute that can select immediate children of the host node to control what is projected:

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

`Max` = red `Barrett` & `max@max.com` = yellow.

Elements are projected on first match, not specificity like CSS.

Descendants cannot be selected eg: `select="table tr"`

Projected subtree retains styles from the main document and can also be styled from inside the shadow tree using the `::content` pseudo element.

If there is no `<content>` or `select` match, the subtree will be hidden inside the shadow root.

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

// styling shadow tree from outside:
#myId::shadow span { color: red; }

// /deep/ ignores all shadow boundaries – used when nesting custom elements:
tab-group /deep/ tab-panel {
  ...
}
```

Accessing deep shadow tree elements with JavaScript:
```javascript
// One way:
document.querySelector('tab-group').shadowRoot
        .querySelector('tab-panel').shadowRoot
        .querySelector('.elem');

// Better way:
document.querySelector('tab-group::shadow tab-panel::shadow .elem');
```


Taken from HTML5 Rocks [[1](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/)], [[2](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom-201/)] & [[3](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom-301/)]









##Custom elements
```javascript
function supportsCustomElements() {
	return 'registerElement' in document;
}
```

Register a new element:
```javascript
var MyCustomElement = document.registerElement('my-element');
```

Instantiating custom elements:
```html
<my-element></my-element>
```

```javascript
// Create DOM in JS:
var myElem = document.createElement('my-element');

// Or use the new operator:
document.body.appendChild(new MyCustomElement());
```

Register a custom element, set it's prototype, give it a shadow, insert template & establish lifecycle callback methods:
```javascript
(function(){
	var importDoc = document.currentScript.ownerDocument;
	var slideShow = document.registerElement('slide-show', {

		// use default prototype
		prototype: Object.create(HTMLElement.prototype, {

			createdCallback: {
				value: function(){
					var template = importDoc.querySelector('#slideShow'); // grab <template> above
					var clone = document.importNode(template.content, true); // clone it
					this.createShadowRoot().appendChild(clone); // Give <slide-show> a shadow root & insert <template>
				}
			},

			attachedCallback: {
				value: function(){
					// fires when added to document
				}
			},

			detachedCallback: {
				value: function(){
					// fires when removed from document
				}
			},

			attributeChangedCallback: {
				value: function(attrName, oldVal, newVal){
					// fires when an attribute is added, removed, or changed.
				}
			}
		})
	});
})();
```


Instantiating type extension custom elements:
```html
<button is="mega-button">
```

```javascript
document.createElement('button', 'mega-button');
```



```sass
// Styling unresolved custom elements, before createdCallback() is called:
:unresolved {}
```


Auto register custom-elements.


Taken from [HTML5 Rocks](http://www.html5rocks.com/en/tutorials/webcomponents/customelements/)

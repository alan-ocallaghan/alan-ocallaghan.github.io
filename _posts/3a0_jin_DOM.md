---
title: "esirgeyen ve bağışlayan❤️ Allah'ın (c.c) adıyla - 3a_start_0DOMAPI"
date: 2016-09-01
categories:
  - blog
tags:
  - 3-techstack_webinterface_frontend_fundamentals
  - 3a_start_0DOMAPI
---
# KUNYE
## CODES:
/Users/serbulentocal/IdeaProjects/techstack/3/techstack_webinterface_frontend_fundamentals/3a_start/3a_start_0DOMAPI/3a0_jin/DOM
## REF:
- [jin]
    - [Document] https://javascript.info/document
        - https://javascript.info/dom-nodes (+)
        - https://javascript.info/dom-navigation (+)

    - [Document_and_resource_loading]   https://javascript.info/loading 
        - https://javascript.info/onload-ondomcontentloaded (+)
<!--###############################-->

# Table of Contents        
- [1_DOM_tree](#1_DOM_tree)
- [3_Page_DOMContentLoaded_load_beforeunload_unload](#3_Page_DOMContentLoaded_load_beforeunload_unload)
    - [3.1_DOMContentLoaded_event](#31_DOMContentLoaded_event)
        - [3.1.1](#311)
        - [3.1.2_using_with_scripts](#312_using_with_scripts)
        - [3.1.3_using_with_styles](#313_using_with_styles)
    - [3.2_load_event](#32_load_event)
        - [3.2.1_window_onload](#321_window_onload)
    - [3.3_beforeunload_event](#33_beforeunload_event)
    - [3.4_unload_event](#34_unload_event)
    - [4_readyState](#4_readyState)
<!--###############################-->






# 1_DOM_tree
The backbone of an HTML document is `tags`. According to the `Document Object Model (DOM)`,
every HTML tag is an object. For example, `document.body` is the object representing the <body> tag.

The document object that represents the whole document is, formally, a DOM node as well.

There are 12 node types. In practice we usually work with `4` of them:

- document – the “entry point” into DOM.
- element nodes – HTML-tags, the tree building blocks.
- text nodes – contain text.
- comments – sometimes we can put information there, it won’t be shown, but JS can read it from the DOM.

Tags are element nodes (or just elements) and form the tree structure: <html> is at the root, then <head> and <body> are its children, etc.

All these objects are accessible using JavaScript, and we can use them to modify the page.

The DOM allows us to do anything with elements and their contents, but first we need to reach the corresponding DOM object.

All operations on the DOM start with the document object. That’s the main “entry point” to DOM. From it we can access any node.

Here’s a picture of links that allow for travel between DOM nodes:
<img width="532" height="383" alt="1_DOM_tree" src="https://github.com/user-attachments/assets/be31d6a2-3c26-4193-b8eb-adfa612b948d" />

The topmost tree nodes are : 

- 1 _Siblings and the parent
Siblings are nodes that are children of the same parent.

For instance, here <head> and <body> are siblings:
```
<html> = document.documentElement
<body> = document.body
<head> = document.head
```

# 3_Page_DOMContentLoaded_load_beforeunload_unload
The lifecycle of an HTML page has three important `events`:

## 3.1_DOMContentLoaded_event
- the browser fully loaded HTML, and the DOM tree is built, but external resources like pictures <img> and stylesheets may not yet have loaded.
- DOM is ready, so the handler can lookup DOM nodes, initialize the interface.

The DOMContentLoaded event happens on the document object. We must use `addEventListener` to catch it:
- [syntax]: `document.addEventListener("DOMContentLoaded", ready);`

### 3.1.1
In this example;  DOMContentLoaded handler runs when the document is loaded,
so it can see all the elements, including <img> below.
But it doesn’t wait for the image to load. So alert shows zero sizes.

### 3.1.2_using_with_scripts
When the browser processes an HTML-document and comes across a <script> tag, it needs to execute it before continuing building the DOM.
so DOMContentLoaded has to wait.

The  exceptions from this rule can be scripts with the `async` attribute, they don’t `block DOMContentLoaded`.


## 3.2_load_event
- not only HTML is loaded, but also all the external resources: images, styles etc are loaded, so styles are applied, image sizes are known etc.
- this event is available via the `onload property`

- [syntax]: `window.onload = function() { };`

### 3.2.1_window_onload
The example below correctly shows image sizes, because `window.onload waits for all images`:
[example]
<!--steps
 -1 - image loaded
-2 - alert (page loaded)
-3 - alert (image size)
-->

## 3.3_beforeunload_event
- the user is leaving: we can check if the user saved the changes and ask them whether they really want to leave.
[example]
<!--
window.onbeforeunload = function (event) {
  return "There are unsaved changes. Leave now?";
}
-->
## 3.4_unload_event
- [syntax]: 
`window.onunload`
- When a visitor leaves the page, the unload event triggers on window
- the user almost left, but we still can initiate some operations, such as sending out statistics.
- Let’s say we gather data about how the page is used: mouse clicks, scrolls, viewed page areas, and so on.

[example]
<!-- let analyticsData = { /* object with gathered data */ };

window.addEventListener("unload", function() {
  navigator.sendBeacon("/analytics", JSON.stringify(analyticsData));
}); -->

# 4_readyState
There are cases when we are not sure whether the document is ready or not. We’d like our function to execute when the DOM is loaded,
The `document.readyState property` tells us about the current loading state.

There are 3 possible values:

- "loading" – the document is loading.
- "interactive" – the document was fully read.
- "complete" – the document was fully read and all resources (like images) are loaded too.




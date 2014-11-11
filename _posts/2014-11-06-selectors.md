---
layout: post
title: Selecting Elements
---

How many times have you seen a web app or library that uses jQuery simply
to perform trivial element selection?  How many times have _you_ written this:
`$(#myElement')`?  Or this: `$('.myElement')`?  Psst... you don't need jQuery
to select elements!  It's pretty easy with the plain 'ole DOM API.

1. [IDs](#by-id)
2. [CSS Classes](#by-css-class)
3. [Tag Names](#by-tag-name)
4. [Attributes](#by-attribute)
5. [Pseudo-classes](#by-pseudo-class)
6. [Children](#children)
7. [Descendants](#descendants)
8. [Exclusion Selectors](#excluding-elements)
9. [Multiple Selectors](#multiple-selectors)
10. [See a Pattern?](#see-a-pattern?)
11. [Filling in the Gaps](#but-i-want-more!)
12. [Next in this Series](#next)


### By ID

#### jQuery

```javascript
// returns a jQuery obj w/ 0-1 elements
$('#myElement');
```

#### DOM API
```javascript
// IE 5.5+
document.getElementById('myElement');
```

...or...

```javascript
// IE 8+
document.querySelector('#myElement');
```

Both methods return a single `Element`.  Note that [`getElementById` is more efficient
than using `querySelector`](http://jsperf.com/getelementbyid-vs-queryselector/11).

Does jQuery's syntax provide any advantage here?  I don't see one.  Do you?


### By CSS Class

#### jQuery
```javascript
// returns a jQuery obj w/ all matching elements
$('.myElement');
```

#### DOM API
```javascript
// IE 9+
document.getElementsByClassName('myElement');
```

...or...

```javascript
// IE 8+
document.querySelectorAll('.myElement');
```

The first method returns an [`HTMLCollection`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCollection)
and [is the most efficient of the two choices](http://jsperf.com/getelementsbyclassname-vs-queryselectorall/25).
`querySelectorAll` always returns a [`NodeList`](https://developer.mozilla.org/en-US/docs/Web/API/NodeList).

Again, really simple stuff here.  Why bother with jQuery?


### By Tag Name

For example, to select all `<div>` elements on a page:

#### jQuery
```javascript
$('div');
```

#### DOM API
```javascript
// IE 5.5+
document.getElementsByTagName('div');
```

...or...

```javascript
// IE 8+
document.querySelectorAll('div');
```

As expected, `querySelectorAll` (which returns a `NodeList`) is less efficient than
`getElementsByTagName` (which returns an `HTMLCollection`).


### By Attribute

Select all elements with a "data-foo-bar" attribute that contains a value of
"someval":

#### jQuery
```javascript
$('[data-foo-bar="someval"]');
```

#### DOM API
```javascript
// IE 8+
document.querySelectorAll('[data-foo-bar="someval"]');
```

Again, the DOM API and jQuery syntax is very similar.


### By Pseudo-class

Select all fields in a specific form that are currently invalid.  Let's assume
our form has an ID of "myForm".

#### jQuery
```javascript
$('#myForm :invalid');
```

#### DOM API
```javascript
// IE 8+
document.querySelectorAll('#myForm :invalid');
```


### Children

Select all children of a specific element.  Let's assume our specific has an ID
of "myParent".

#### jQuery
```javascript
$('#myParent').children();
```

#### DOM API
```javascript
// IE 5.5+
// NOTE: This will include comment and text nodes as well.
document.getElementById('myParent').childNodes;
```

...or...


```javascript
// IE 9+ (ignores comment & text nodes).
document.getElementById('myParent').children;
```


But what if we want to only find specific children?  Say, only children with an
attribute of "ng-click"?

#### jQuery
```javascript
$('#myParent').children('[ng-click]');
```

...or...

```javascript
$('#myParent > [ng-click]');
```


#### DOM API
```javascript
// IE 8+
document.querySelector('#myParent > [ng-click]');
```


### Descendants

Find all anchor elements under #myParent.

#### jQuery
```javascript
$('#myParent A');
```

#### DOM API
```javascript
// IE 8+
document.querySelectorAll('#myParent A');
```


### Excluding Elements

Select all `<div>` elements, except those with a CSS class of "ignore".

#### jQuery
```javascript
$('DIV').not('.ignore');
```

...or...

```javascript
$('DIV:not(.ignore)');
```

#### DOM API
```javascript
// IE 8+
document.querySelectorAll('DIV:not(.ignore)');
```


### Multiple Selectors

Select all `<div>`, `<a>` and `<script>` elements.

#### jQuery
```javascript
$('DIV, A, SCRIPT');
```

#### DOM API
```javascript
// IE 8+
document.querySelectorAll('DIV, A, SCRIPT');
```

## See a Pattern?

If we focus exclusively on selector support, and don't need to handle anything
older than IE8, seems like we could get pretty far simply by replacing
jQuery with this:

```javascript
window.$ = function(selector) {
    var selectorType = 'querySelectorAll';

    if (selector.indexOf('#') === 0) {
        selectorType = 'getElementById';
        selector = selector.substr(1, selector.length);
    }

    return document[selectorType](selector);
};
```

## But I Want More!
For the _vast_ majority of projects, the selectors support baked into the Web
API is sufficient.  But what if you are unfortunate enough to require IE7
support?  In that case, you'll probably need a little help from some third party
code.  

Sure, you could just pull in jQuery, but why use such a large
codebase when you only need advanced selector support for now?  Instead, try a
micro-library that focuses exclusively on element selection. Consider [Sizzle](http://sizzlejs.com/),
which happens to be the selector library that jQuery uses.  [Selectivizr](http://selectivizr.com/)
is another very small selector library that brings CSS3 selector support to very old browsers.


## Next

I'll talk about manipulating DOM elements in the next post.
---
layout: post
title: DOM Manipulation
---

We previously [learned how to easily select elements without relying on jQuery]({{ site.baseurl }}/selectors).
but what about changing elements?  What about creating new elements?  How about
relocating elements elsewhere on the page?  You may be glad to know that all of this,
and more, is also possible without pulling in jQuery.  The Web API provides us with
all of the tools we need, once again.

You will see that some DOM manipulation is trivial without the aid of jQuery or 
any other library.  However, others may be a bit tricky.  This is the time to stress
(again) that I am not trying to bash jQuery, nor will I assert that jQuery is
useless or completely unnecessary.  The intent of these posts is to inform you
how to work with the brower without jQuery, if you so choose.  You may find that,
in many cases, a large library like jQuery goes mostly unused, and can be ommitted.


1. [Creating Elements](#creating-elements)
2. [Inserting Elements Before & After](#inserting-elements-before-&-after)
3. [Inserting Elements As Children](#inserting-elements-as-children)
4. [Moving Elements](#moving-elements)
5. [Removing Elements](#removing-elements)
6. [Adding & Removing CSS Classes](#adding-&-removing-css-classes)
7. [Adding/Removing/Changing Attributes](#adding/removing/changing-attributes)
8. [Adding & Changing Text Content](#adding-&-changing-text-content)
9. [Adding/Updating Element Styles](#adding/updating-element-styles)
11. [Micro-libraries For More Help](#micro-libraries-for-more-help) 
12. [Next](#next)


## Creating Elements

#### jQuery
```javascript
$('<div></div>');
```

#### DOM API
```javascript
// IE 5.5+
document.createElement('div');
```

Wow, that was pretty easy.  jQuery saved us a few keystrokes, but that's hardly
worth the bytes.


## Inserting Elements Before & After

Let's create an element and insert it after another specific element.

So, we start with:

```html
<div id="1"></div>
<div id="2"></div>
<div id="3"></div>
```

... and we'd like to create a new element with an ID of '1.1' and insert it 
between the first two DIVs, giving us this:

```html
<div id="1"></div>
<div id="1.1"></div>
<div id="2"></div>
<div id="3"></div>
```

#### jQuery
```javascript
$('#1').after('<div id="1.1"></div>');
```

#### DOM API
```javascript
// IE 4+
document.getElementById('1')
    .insertAdjacentHTML('afterend', '<div id="1.1"></div>');
```

Ha!  Take THAT jQuery!  Pretty easy in every browser just relying on the tools
built into the browser. 


Ok, what if we want to insert a new element BEFORE the first div, giving us this:

```html
<div id="0.9"></div>
<div id="1"></div>
<div id="2"></div>
<div id="3"></div>
```

#### jQuery
```javascript
$('#1').before('<div id="0.9"></div>');
```

#### DOM API
```javascript
// IE 4+
document.getElementById('1')
    .insertAdjacentHTML('beforebegin', '<div id="0.9"></div>');
```

Pretty much the same as the last, with the exception of a different method call 
for jQuery and a different parameter for the plain 'ole JavaScript approach.


## Inserting Elements As Children
Let's say we have this:

```html
<div id="parent">
    <div id="oldChild"></div>
</div>
```

...and we want to create a new element and make it the first child of the parent,
like so:

```html
<div id="parent">
    <div id="newChild"></div>
    <div id="oldChild"></div>
</div>
```

#### jQuery
```javascript
$('#parent').prepend('<div id="newChild"></div>');
```

#### DOM API
```javascript
// IE 4+
document.getElementById('parent')
    .insertAdjacentHTML('afterbegin', '<div id="newChild"></div>');
```

...or create a new element and make it the last child of #parent:

```html
<div id="parent">
    <div id="oldChild"></div>
    <div id="newChild"></div>
</div>
```

#### jQuery
```javascript
$('#parent').append('<div id="newChild"></div>');
```

#### DOM API
```javascript
// IE 4+
document.getElementById('parent')
    .insertAdjacentHTML('beforeend', '<div id="newChild"></div>');
```

All of this looks a lot like the previous section that dealt with insertion of
new elements.  Again, it's pretty simple to do all of this, cross-browser, without
any help from jQuery (or any other library).  


## Moving Elements
Consider the following markup:

```html
<div id="parent">
    <div id="c1"></div>
    <div id="c2"></div>
    <div id="c3"></div>
</div>
<div id="orphan"></div>
```

What if we want to relocate the #orphan as the last child of the #parent?  
That would give us this:

```html
<div id="parent">
    <div id="c1"></div>
    <div id="c2"></div>
    <div id="c3"></div>
    <div id="orphan"></div>
</div>
```

#### jQuery
```javascript
$('#parent').append($('#orphan'));
```

#### DOM API
```javascript
// IE 5.5+
document.getElementById('parent')
    .appendChild(document.getElementById('orphan'));
```

Simple enough without jQuery.  But what if we want to make #orphan the first child 
of #parent, giving us this:

```html
<div id="parent">
    <div id="orphan"></div>
    <div id="c1"></div>
    <div id="c2"></div>
    <div id="c3"></div>
</div>
```

#### jQuery
```javascript
$('#parent').prepend($('#orphan'));
```

#### DOM API
```javascript
// IE 5.5+
document.getElementById('parent')
    .insertBefore(document.getElementById('orphan'), document.getElementById('c1'));
```

We can still complete this in one line, but it's a bit less intuitive & verbose
without jQuery.  Still, not too bad.



## Removing Elements
How can we remove an element from the DOM?  Let's say we know an element
with an ID of 'foobar' exists.  Let's kill it!

### jQuery
```javascript
$('#foobar').remove();
```

### DOM API
```javascript
// IE 5.5+
document.getElementById('foobar').parentNode
    .removeChild(document.getElementById('foobar'));
```

The DOM API approach is certainly a bit verbose and ugly, but it works!  Note
that we don't have to be directly aware of the parent element, which is nice.


## Adding & Removing CSS Classes
We have a simple element:

```html
<div id="foo"></div>
```

...let's add a CSS class of "bold" to this element, giving us:

```html
<div id="foo" class="bold"></div>
```

#### jQuery
```javascript
$('#foo').addClass('bold');
```

#### DOM API
```javascript
document.getElementById('foo').className += 'bold';
```

Let's remove that class now:

#### jQuery
```javascript
$('#foo').removeClass('bold');
```

#### DOM API
```javascript
// IE 5.5+
document.getElementById('foo').className = 
    document.getElementById('foo').className.replace(/^bold$/, '');
```

As usual, more characters, but still easy without jQuery.


## Adding/Removing/Changing Attributes
Let's start out with a simple element, like this:

```html
<div id="foo"></div>
```

Now, let's say this `<div>` actually functions as a button.  We should attach 
the approriate `role` attribute to make this element more accessible.  

#### jQuery
```javascript
$('#foo').attr('role', 'button');
```

#### DOM API
```javascript
// IE 5.5+
document.getElementById('foo').setAttribute('role', 'button');
```

In both cases, a new attribute can be created, or an existing attribute can
be updated using the same code.  

What if the behavior of our `<div>` changes, and it no longer functions as a
button.  In fact, it's just a plain 'ole `<div>` now with some trivial text or
markup.  Let's remove that `role`...

#### jQuery
```javascript
$('#foo').removeAttr('role');
```

#### DOM API
```javascript
// IE 5.5+
document.getElementById('foo').removeAttribute('role');
```



## Adding & Changing Text Content
Now, we have the following markup:

```html
<div id="foo">Hi there!</div>
```

...but we wan't to update the text to say "Goodbye!".

#### jQuery
```javascript
$('#foo').text('Goodbye!');
```

Note that you can also easily retrieve the current text of the element by calling
`text` with no parameters.

#### DOM API
```javascript
// IE 5.5+
document.getElementById('foo').innerHTML = 'Goodbye!';

// IE 5.5+ but NOT Firefox
document.getElementById('foo').innerText = 'GoodBye!';

// IE 9+
document.getElementById('foo').textContent = 'Goodbye!';
```

Both properties above will return the current element's HTML/text as well.

The advantage to using `innerText` or `textContent` is that any HTML is escaped, 
which is a great feature if the content is user-supplied and you only ever want 
to include text as content for the selected element.


## Adding/Updating Element Styles
Generally, adding styles inline or with JavaScript is a ["code smell"](http://en.wikipedia.org/wiki/Code_smell),
but it may be needed in some unique instances.  For those cases, I'll show you
how that can be done with jQuery and the DOM API.

Given a simple element with some text:

```html
<span id="note">Attention!</span>
```

...we'd like to make it stand out a bit more, so let's make it appear bold.

#### jQuery
```javascript
$('#note').css('fontWeight', 'bold');
```

#### DOM API
```javascript
// IE 5.5+
document.getElementById('note').style.fontWeight = 'bold';
```

I actually prefer the way the DOM API approach looks in this case.  It seems
much more intuitive than jQuery's `css` method.


## Micro-Libraries For More Help
So, guess what?  You don't really need jQuery for cross-browser DOM manipulation
either!  I realize that jQuery may make some more complex manipulation a bit easier,
bit if you're only really concerned with complex DOM manipulation tasks, then
consider pulling in a smaller library that focuses mostly on this.  A couple 
choices are [jBone](https://github.com/kupriyanenko/jbone) and [dom.js](https://github.com/dkraczkowski/dom.js).
There's nothing wrong with "rolling your own" library either.  DOM manipulation
isn't as difficult as you may think.



## Next
For me: I'll talk about making ajax requests.  

For you: if I've left out any important DOM manipulation examples, let me know 
in the comments so I can update the post.
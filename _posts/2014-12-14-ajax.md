---
layout: post
title: Ajax Requests
---

Many developers who learned web development through a jQuery lens probably think
that jQuery is doing something magical when you invoke the `$.ajax` method.  That
couldn't be further from the truth.  All of the heavy lifting is done by the browser
via the `XMLHttpRequest` object.  jQuery's `ajax` is just a wrapper around
`XMLHttpRequest`.  Using the browser's built-in support for ajax requests isn't very
difficult, as you'll see in a moment.  Even cross-origin requests are simple.

1. [GETting](#getting)
2. [POSTing](#posting)
3. [URL Encoding](#url-encoding)
4. [JSON](#sending-and-receiving-json)
5. [Uploading](#uploading-files)
6. [CORS](#cors)
7. [JSONP](#jsonp)
8. [Libraries to Consider](#libraries-to-consider)
9. [Next in this Series](#next)

## GETting
Let's start with a simple but common request.  We need to ask the server for the name of
a person, given that person's unique ID.  The unique ID string should be included
as a query parameter in the URI, with an empty payload, as is common for GET
requests.  Invoke an alert with the user's name, or an error if the request fails.

#### jQuery
There are a couple ways to initiate a GET ajax request using jQuery's API.  One
involves the `get` method, which is shorthand for `ajax` with a `type` of
'get'.  We'll just use the `ajax` method going forward for consistency.

```javascript
$.ajax('myservice/username', {
    data: {
        id: 'some-unique-id'
    }
})
.then(
    function success(name) {
        alert('User\'s name is ' + name);
    },

    function fail(data, status) {
        alert('Request failed.  Returned status of ' + status);
    }
);
```

#### Native XMLHttpRequest Object
```javascript
var xhr = new XMLHttpRequest();
xhr.open('GET', 'myservice/username?id=some-unique-id');
xhr.onload = function() {
    if (xhr.status === 200) {
        alert('User\'s name is ' + xhr.responseText);
    }
    else {
        alert('Request failed.  Returned status of ' + xhr.status);
    }
};
xhr.send();
```

The above native JS example will work in IE7 and up.  Even IE6 is trivial
to support, just by swapping out `new XMLHttpRequest()` with `new ActiveXObject("MSXML2.XMLHTTP.3.0")`.
Our native example seems easy to follow and fairly intuitive to write.  So, why
use jQuery here?  What is gained?


## POSTing
Let's extend our last example a bit.  Now that we have the user's full name, let's
go ahead and change it.  We will again address this user by ID.  We'll need to
POST a message to our server for that particular user, and include the user's new
name inside the request body as a URL encoded string.  The server will return the
updated name in its response, so we should check that to make sure all is well.

The correct method to use for this case is actually PATCH, but there are some
issues with PATCH and other non-traditional methods in older browsers (such as IE8),
So, we'll just use POST here, but the code is identical in either case, with the exception
of the differing method name.

Also note that the following approach is pretty much the same, regardless of the
request method.

#### jQuery
```javascript
var newName = 'John Smith';

$.ajax('myservice/username?' + $.param({id: 'some-unique-id'}), {
    method: 'POST',
    data: {
        name: newName
    }
})
.then(
    function success(name) {
        if (name !== newName) {
            alert('Something went wrong.  Name is now ' + name);
        }
    },

    function fail(data, status) {
        alert('Request failed.  Returned status of ' + status);
    }
);
```

#### Native XMLHttpRequest Object
```javascript
var newName = 'John Smith',
    xhr = new XMLHttpRequest();

xhr.open('POST', 'myservice/username?id=some-unique-id');
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
xhr.onload = function() {
    if (xhr.status === 200 && xhr.responseText !== newName) {
        alert('Something went wrong.  Name is now ' + xhr.responseText);
    }
    else if (xhr.status !== 200) {
        alert('Request failed.  Returned status of ' + xhr.status);
    }
};
xhr.send(encodeURI('name=' + newName));
```

It seems pretty clear here that the jQuery way to send this request is much more
elegant.  It does a portion of the work for you.  But, is this elegance worth
pulling in the dependency?  If you are comfortable enough with the relatively
simple `XMLHttpRequest` object, the answer is probably "no".


## URL Encoding
jQuery provides a function that takes an object and turns it into a URL encoded
string:

```javascript
$.param({
    key1: 'some value',
    'key 2': 'another value'
});
```

This is nice, but we can do something similar with a little elbow grease, sans
jQuery.  The Web API provides two functions that URL encode strings:
[`encodeURI`][encodeuri] and [`encodeURIComponent`][encodeuricomponent].  A function that builds on this native
support to mirror the functionality of `$.param` isn't _terribly_ difficult:

```javascript
function param(object) {
    var encodedString = '';
    for (var prop in object) {
        if (object.hasOwnProperty(prop)) {
            if (encodedString.length > 0) {
                encodedString += '&';
            }
            encodedString += encodeURI(prop + '=' + object[prop]);
        }
    }
    return encodedString;
}
```

Yes, of course, the jQuery method here is _much_ more elegant.  But this is, in
my humble opinion, one of the few instances where jQuery noticably improves your
code.  You're certainly not going to pull in jQuery just for the `$.param` method,
are you?


## Sending and Receiving JSON
We now need to communicate with an API that expects JSON, and returns it in the
response.  Say we need to update some information for a specific user.  Once the
server processes our update, it will echo all current information (after the update)
about that user in the response.  The proper method for this request is PUT, so
let's use that.

#### jQuery
```javascript
$.ajax('myservice/user/1234', {
    method: 'PUT',
    contentType: 'application/json',
    processData: false,
    data: JSON.stringify({
        name: 'John Smith',
        age: 34
    })
})
.then(
    function success(userInfo) {
        // userInfo will be a JavaScript object containing properties such as
        // name, age, address, etc
    }
);
```
The above code is actually pretty awful.  jQuery is broken in the ajax department
on a number of levels.  It's actually quite confusing to send anything other than
a trivial ajax request using jQuery, in my experience.  jQuery's ajax module is
targeted at application/x-www-form-urlencoded requests.  Any other encoding type
will require you to do a bit more work.

First we need to tell jQuery to leave the `data` alone (i.e. don't URL encode it).
Then, we must turn the JavaScript object into JSON ourself.  Why can't jQuery do this
for us based on the `contentType`?  I'm not sure.

If the server returns an appropriate Content-Type in the response, the success
handler should be passed a JavaScript object representing the JSON returned by
the server.


#### Web API
```javascript
var xhr = new XMLHttpRequest();
xhr.open('PUT', 'myservice/user/1234');
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.onload = function() {
    if (xhr.status === 200) {
        var userInfo = JSON.parse(xhr.responseText);
    }
};
xhr.send(JSON.stringify({
    name: 'John Smith',
    age: 34
}));
```

The above code will work in IE8 and up.  But maybe you work at an awful company that
requires support for ancient browsers.  In that case, just drop in
[json.js](https://github.com/douglascrockford/JSON-js) to fill in for the lack of `JSON` support in IE7 and older.


## Uploading Files
For starters, you should know that the only way to upload files in IE9 and older
is by submitting a `<form>` that contains an `<input type="file">`.  jQuery isn't
going to help you out much with that, and frankly neither is the Web API.

So let's talk about uploading files in modern browsers.  This is made possible by
the [File API][file-api].  As you will see shortly, jQuery doesn't help you out at
all when it comes to uploading files.  If anything, uploading files is _more_
confusing with `$.ajax`.

With the aid of the File API, you can upload files two ways.  The first involves
sending the file as part of a multipart encoded request.  The request sent here is
identical to the one sent by the browser when a `<form enctype="multipart/form-data">`
is submitted.  The second involves sending a request with a body that consists entirely
of the file data.  In each case, you must have access to the underlying `File`
or `Blob`, as this is the entity you must send to the server.

Given the following markup:

```html
<input type="file" id="test-input">
```

#### jQuery
First, we'll upload a file as part of a multipart encoded request:

```javascript
var file = $('#test-input')[0].files[0],
    formData = new FormData();

formData.append('file', file);

$.ajax('myserver/uploads', {
    method: 'POST',
    contentType: false,
    processData: false,
    data: formData
});
```

How non-intuitive is that?  `contentType: false`?  What does that even mean?  Well,
this is required to ensure that jQuery doesn't insert its own Content-Type header,
since the browser MUST specify the Content-Type for you as it includes a calculated
multipart boundary ID used by the server to parse the request.

Now, let's send a POST where the entire payload of the request consists of the
file data:

```javascript
var file = $('#test-input')[0].files[0];

$.ajax('myserver/uploads', {
    method: 'POST',
    contentType: file.type,
    processData: false,
    data: file
});
```

That's a bit better, but we still need to include the non-sensical `processData: false`
option to prevent jQuery from attempting to URL-encode the payload.


#### XMLHttpRequest
First, multipart encoded:

```javascript
var formData = new FormData(),
    file = document.getElementById('test-input').files[0],
    xhr = new XMLHttpRequest();

formData.append('file', file);
xhr.open('POST', 'myserver/uploads');
xhr.send(formData);
```

And now, let's send the file as the payload of the request:

```javascript
var file = document.getElementById('test-input').files[0],
    xhr = new XMLHttpRequest();

xhr.open('POST', 'myserver/uploads');
xhr.setRequestHeader('Content-Type', file.type);
xhr.send(file);
```

Hey, that was really easy.  All the power in uploading files comes from the
File API and `XMLHttpRequest`.  jQuery just gets in the way.

## CORS
CORS, or **C**ross **O**rigin **R**esource **S**haring (sending cross-domain ajax requests)
is actually a fairly complex topic, and there is much to discuss here.  But,
we're really not concerned with all the details here.  This assumes you already
understand CORS and the Same Origin Policy.  If you don't, [MDN has a great explanation][cors].
Maybe I'll even take some time to write more on the topic.

Anyway, sending a cross-origin ajax request via JavaScript is pretty straightforward
in modern browsers.  The process is a bit hairy in IE8 and IE9 though.  In either
case, jQuery offers zero assistance.

For modern browsers, all of the work is delegated to the server.  The browser
does everything else for you.  Your code for a cross-origin ajax request in a
modern browser is identical to a same-origin ajax request.  So, I won't bother showing
that in jQuery or native JavaScript.

It's important to know that cookies are not sent by default with cross-origin ajax
requests.  You must set the `withCredentials` flag on the `XMLHttpRequest`
transport.  Let's take a look.

#### jQuery
```javascript
$.ajax('http://someotherdomain.com', {
    method: 'POST',
    contentType: 'text/plain',
    data: 'sometext',
    beforeSend: function(xmlHttpRequest) {
        xmlHttpRequest.withCredentials = true;
    }
});
```

#### XMLHttpRequest
```javascript
var xhr = new XMLHttpRequest();
xhr.open('POST', 'http://someotherdomain.com');
xhr.withCredentials = true;
xhr.setRequestHeader('Content-Type', 'text/plain');
xhr.send('sometext');
```

Clearly no benefit from jQuery here.

jQuery actually becomes a headache to deal with when we need to send a cross-domain
ajax request in IE8 or IE9.  If you're using jQuery for this purpose, you are truly
trying to fit a square peg into a round hole.

To understand why jQuery is a poor
fit for cross-origin requests in IE9 and IE8, it's important to understand a couple
low-level points:

1. Cross-origin ajax requests in IE8 and IE9 can only be sent using the IE-proprietary
`XDomainRequest` transport.  I'll save the rant for why this was such a huge mistake
by the IE development team for another blog post.  Regardless, `XDomainRequest` is
a stripped down version of `XMLHttpReqest`, and it **must** be used when making cross-origin
ajax requests in IE8 and IE9.  To read more about the (significant) restrictions imposed
on this transport, read [Eric Law's MSDN post on the subject][xdr].

2. jQuery's `ajax` method (and all associated aliases) are just wrappers for
`XMLHttpRequest`.  It has a hard dependency on `XMLHttpRequest`.

So, you need to use `XDomainRequest` to send the cross-origin request in IE8/9,
but `jQuery.ajax` is hard-coded to use `XMLHttpRequest`.  That's a problem, and
resolving it in the context of jQuery is not going to be pleasant.  In fact, it's
so unpleasant that no one in their right mind would do it.  Luckily, for those
dead-set on using jQuery for this type of call, there are a few plug-ins that will
"fix" jQuery in this regard.  Essentially, the plug-ins must override jQuery's
ajax request sending/handling logic via the `$.ajaxTransport` method.

But, sending ajax requests in IE8/9 is pretty simple without jQuery.  In fact,
even if you're a die-hard jQuery fan, you should do it this way:

```javascript
// For cross-origin requests, some simple logic
// to determine if XDomainReqeust is needed.
if (new XMLHttpRequest().withCredentials === undefined) {
    var xdr = new XDomainRequest();
    xdr.open('POST', 'http://someotherdomain.com');
    xdr.send('sometext');
}
```

Note that you cannot set **any** request headers when using `XDomainRequest`.  If
you can avoid making cross-origin ajax requests in IE8/9, you should.  But if you must,
[become familiar with its limitations][xdr].


## JSONP
I'll begin here by suggesting you avoid using JSONP, as it's proven to be [a potential
security issue][jsonpsecurity].  Also, in modern browsers, CORS is a much better route.

If you're not familiar with JSONP, the name may
be a bit misleading.  There is actually no JSON involved here at all.  It's a _very_
common misconception that JSON must be returned from the server when the client
initiates a JSONP call, but that's simply not true.  Instead, the server returns
a function invocation, which is not valid JSON.

JSONP stands for **J**ava**S**cript **O**bject **N**otation with **P**adding.  It's
essentially just an ugly hack that exploits the fact that `<script>` tags that
load content from a server are not bound by the same-origin policy.  There needs
to be cooperation and an understanding of the convention by both client and server
for this to work properly.  You simply need to point the `src` attribute of a `<script>`
tag at a JSONP-aware endpoint, including the name of an exisitng global function
as a query parameter.  The server will then construct a string representation that,
when executed by the browser, will invoke the global function, passing in the requested
data.

#### jQuery
```javascript
$.ajax('http://jsonp-aware-endpoint.com/user', {
    jsonp: 'callback',
    dataType: 'jsonp',
    data: {
        id: 123
    }
}).then(function(response) {
    // handle requested data from server
});
```

jQuery has entirely abstracted away the awfulness of JSONP.  +1 for jQuery here.
But, we can still accomplish all of this without jQuery, and it's not as complicated
as it might seem:

#### Without jQuery
```javascript
window.myJsonpCallback = function(data) {
    // handle requested data from server
};

var scriptEl = document.createElement('script');
scriptEl.setAttribute('src',
    'http://jsonp-aware-endpoint.com/user?callback=myJsonpCallback&id=123');
document.body.appendChild(scriptEl);
```


## Libraries to Consider
I beleive that the examples I provided above show that any ajax related code
can be done fairly easily without pulling in any dependencies.  But, if you're not
convinced and don't want to pull in jQuery just for some ajax help, there are a few
focused libraries you can check out.

1. [fetch][fetch]: a polyfill for the [emerging fetch standard][fetchspec], which
aims to make native ajax code more intuitive and modern.

2. [xdomain][xdomain]: A library that makes cross-origin requests in all browsers, back to
IE8, really easy.  It makes use of the [Web Messaging API][postmessage], and includes some of its own
conventions to make this work.  The server buy-in is must simpler than the requirements for
CORS as well due to some clever workarounds in this library.

3. [Lightweight-JSONP][lightweightjsonp]: As the name suggests, this is a small
library that aims to make JSONP a breeze in the browser.


## Next
For me: I'll talk about [dealing with events (both DOM/native and custom)]({{ site.baseurl }}/events).

For you: if I've left out any important ajax-related topics, let me know in the comments
so I can update the post.


[cors]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS
[encodeuri]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURI
[encodeuricomponent]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent
[fetch]: https://github.com/github/fetch
[fetchspec]: https://fetch.spec.whatwg.org/
[file-api]: http://www.w3.org/TR/FileAPI/
[jsonpsecurity]: http://security.stackexchange.com/a/23439
[lightweightjsonp]: https://github.com/IntoMethod/Lightweight-JSONP
[postmessage]: http://www.w3.org/TR/webmessaging/
[xdomain]: https://github.com/jpillora/xdomain
[xdr]: http://blogs.msdn.com/b/ieinternals/archive/2010/05/13/xdomainrequest-restrictions-limitations-and-workarounds.aspx

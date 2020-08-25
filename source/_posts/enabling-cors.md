---
title: Enabling CORS
categories:
    - Programming
    - System Administration
tags: 
    - cordova
    - wkwebview
    - javascript
    - ios
---

## CORS in a Nutshell

Now that Apple is enforcing `WKWebView` on all applications, there is a whole lot more talk on these strange problems with using <span class="tip" title="XMLHttpRequest">XHR</span> or ajax requests. They no longer work! Why? Because of <span class="tip" title="Cross-Origin Resource Sharing">CORS</span>.

With `WKWebView`, <span class="tip" title="Cross-Origin Resource Sharing">CORS</span> is enabled and enforced. This means that if you make a <span class="tip" title="XMLHttpRequest">XHR</span> request, the server must respond in a CORS-compliant way.

In the nutshell, CORS allows you to grant or restrict access to your backend APIs. On every <span class="tip" title="XMLHttpRequest">XHR</span> request, an `Origin` http header is added. The `Origin` header is always the domain of the executing script.

In Cordova however, the website is hosted on the device itself and not from a webserver. So what is it's origin?
Up until `cordova-ios@6`, apps were always loaded through the `file://` protocol, therefore the `Origin` is `null`. Starting with `cordova-ios@6`, where the `WKWebView` is built into the core, you can specify a custom URL scheme (e.g. `app://localhost`). In these cases, the `Origin` will be your chosen URL scheme.

Because of this, the intended purpose of <span class="tip" title="Cross-Origin Resource Sharing">CORS</span> is broken because this means you can't actually lock your API down to your app specifically with the protocol. Anybody can change their origin to anything. Unfortunately, we still need to follow the protocol.

So how do implement the protocol? This depends on your tech stack on your server. But most simply your server should respond to every request with the following response headers:

- `Access-Control-Allow-Origin`
- `Access-Control-Allow-Headers`
- `Access-Control-Allow-Methods`

## Knowledge Prerequisites

Before we continue, I assume you have proficient knowledge in the HTTP protocol, you know your way around your server stack, and you know how to configure your webserver. Some examples are shown using specific technologies and should be easy to adapt to other technologies.

### Access-Control-Allow-Origin

The `Access-Control-Allow-Origin` is the main header, which expects one value. According to some [documentations](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin), this value can be `*` or `null`, but in my experience, neither worked on `WKWebView`.

Instead, it is best to simply set the `Access-Control-Allow-Origin` header to the same value as the request `Origin` header. This achieves the same result as a `*` wildcard. Examples are shown below:

#### NGINX
``` nginx
add_header 'Access-Control-Allow-Origin' $http_origin always;

```

### Access-Control-Allow-Headers

This header defines a list of headers that are acceptable. Some common ones that you'll prbably want to include are:

- Accept
- Content-Type
- X-Requested-With

Add more as you see fit, including any custom headers you choose to use.

### Access-Control-Allow-Methods

This header defines a list of HTTP methods that are acceptable. Chances are you'll want the following methods:

- GET
- POST
- PUT
- DELETE
- OPTIONS

Special note on `OPTIONS`, this is a *required* method to be added for Preflight Requests. Feel free to add or remove others as you see fit.

## Preflight Requests

Preflight requests will happen in certain conditions. A preflight request is an additional request sent by the webview to the same URL endpoint, but with the `OPTIONS` http method. This happens behind-the-scenes.

To borrow a graphic from [Mozilla Contributors](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS), here is a flow of a `/doc` request.

![](/images/preflight_correct.png)

Therefore, it is important to respond to the `OPTIONS` http method, in addition to your normal API request. How do I respond to the preflight request? Glad you asked.

The preflight response should:

- return no content body
- return a status code `204`
- return a `Content-Type` header with a value of `text/plain charset=UTF-8`
- return a `Content-Length` with a value of `0`
- return the `Access-Control` headers that we talked about earlier.

Here are some examples:

NGINX

``` nginx
if ($request_method = 'OPTIONS') {
    add_header 'Access-Control-Allow-Origin' $http_origin always;
    add_header 'Access-Control-Allow-Methods' 'GET, POST, DELETE, PUT, OPTIONS, HEAD' always;
    add_header 'Access-Control-Allow-Headers' 'Accept, Content-Type, Access-Control-Allow-Origin' always;
    add_header 'Content-Type' 'text/plain charset=UTF-8' always;
    add_header 'Content-Length' 0 always;
    return 204;
}
```

Graphics in this article is used under the [CC-BY-SA 2.5](https://creativecommons.org/licenses/by-sa/2.5/) license.

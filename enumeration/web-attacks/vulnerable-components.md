# Vulnerable components

### What is it?

Pentesting vulnerable components in web applications refers to the process of identifying, analyzing, and testing potential weaknesses present in various components of a web application, such as libraries, frameworks, and other software modules.

**A simple example**

A web application may use an outdated version of jQuery, which is known to have vulnerabilities. If an attacker can exploit this vulnerability, they may be able to run arbitrary code on a user's browser, potentially leading to unauthorized access or data theft.

**Other learning resources:**

* OWASP Top 10: [https://owasp.org/www-project-top-ten/](https://owasp.org/www-project-top-ten/)
* CWE/SANS Top 25: [https://www.sans.org/top25-software-errors/](https://www.sans.org/top25-software-errors/)
* Web Application Security Testing Cheat Sheet: [https://cheatsheetseries.owasp.org/cheatsheets/Web\_Application\_Security\_Testing\_Cheat\_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/Web_Application_Security_Testing_Cheat_Sheet.html)

### Checklist

* [ ] What components exist?
  * [ ] What does the technology stack look like? (libraries, frameworks, etc.)
  * [ ] Identify all plugins and extensions used
    * [ ] Is it a CMS with plugins?
  * [ ] Identify the versions of all these components
* [ ] Are there known vulnerabilities for those components?
  * [ ] NVD, exploit-db, dependency checker, etc
* [ ] Do the exploits have PoCs or available exploits?
* [ ] Can we validate the vulnerability?
  * [ ] If not, what other protections need to be bypassed?

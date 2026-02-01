# Enumerating COM Objects and their Methods

This is a quick note to capture some of the commands for finding interesting COM objects and the methods they expose, based on the great article from Fireeye:

> The Microsoft Component Object Model (COM) is a platform-independent, distributed, object-oriented system for creating binary software components that can interact
>
> https://docs.microsoft.com/en-us/windows/win32/com/the-component-object-model

This is less of a post-exploitation technique, rather a method that allows one to look for interesting COM objects that could be leveraged by one's malware.

### Enumerating COM Objects

We can find all the COM objects registered on the Windows system with:

{% code title="PowerShell" %}
```powershell
gwmi Win32_COMSetting | ? {$_.progid } | sort | ft ProgId,Caption,InprocServer32
```
{% endcode %}

![](<../../.gitbook/assets/image (951)>)

### Enumerating COM Object Methods

Once we have the list of COM objects and have identified an interesting COM object, we can check the methods it exposes. In this example, use the COM object WScript.Shell.1 and check its methods like so:

{% code title="PowerShell" %}
```powershell
$o = [activator]::CreateInstance([type]::GetTypeFromProgID(("WScript.Shell.1"))) | gm
```
{% endcode %}

Below are the methods exposed by WScript.Shell.1 COM object, one of which is RegRead:

![](<../../.gitbook/assets/image (952)>)

RegRead accepts one string as an argument — a path to the registry value. Example:

{% code title="PowerShell" %}
```powershell
$o.RegRead("HKEY_CURRENT_USER\Volatile Environment\LOGONSERVER")
```
{% endcode %}

Below shows how a registry value was read successfully:

![](<../../.gitbook/assets/image (953)>)

### Exposing All COM Object Methods

You can iterate through all the COM objects, list their methods, and save everything to a text file for later inspection:

{% code title="PowerShell" %}
```powershell
$com = gwmi Win32_COMSetting | ? {$_.progid } | select ProgId,Caption,InprocServer32

$com | % {
    $_.progid | out-file -append methods.txt
    [activator]::CreateInstance([type]::GetTypeFromProgID(($_.progid))) | gm | out-file -append methods.txt
    "`n`n" | out-file -append methods.txt
}
```
{% endcode %}

The output file will contain all the methods of all COM objects exposed. In the example output, methods for Shell.Application.1 are in focus:

![](<../../.gitbook/assets/image (954)>)

### References

{% embed url="https://www.fireeye.com/blog/threat-research/2019/06/hunting-com-objects.html" %}

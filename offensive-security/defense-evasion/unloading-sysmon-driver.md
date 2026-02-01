# Unloading Sysmon Driver

## Unloading Sysmon Driver

### Execution

{% code title="attacker\@victim" %}
```
fltMC.exe unload SysmonDrv
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LMEMV4bZcWMil3q5V3S%2F-LMENYs7FgCCrjOy2R6B%2Fsysmon-cmd.png?alt=media\&token=8c265dfa-772a-4084-8a52-16e3959d0194)

### Observations

Windows event logs suggesting `SysmonDrv` was unloaded successfully:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LMEMV4bZcWMil3q5V3S%2F-LMEN_eOBU33IuE-WxMB%2Fsysmon-unload-log1.png?alt=media\&token=bdaba8b8-383d-49d6-8599-cfbfc546815d)

As well as processes requesting special privileges:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LMEMV4bZcWMil3q5V3S%2F-LMENcwj6SVLspxcrDiA%2Fsysmon-unload-log2.png?alt=media\&token=003788aa-60bc-4210-b33f-fa303e427316)

Note how in the last 35 minutes since the driver was unloaded, no further process creation events were recorded, although I spawned new processes during that time:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LMEMV4bZcWMil3q5V3S%2F-LMEOphHDD-ddK5t1gx2%2Fsysmon-last-event.png?alt=media\&token=821fc73f-6eb9-4ded-9115-1c9b524c7f0b)

Note how the system thinks that the sysmon is still running, which it is, but not doing anything useful:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LMEMV4bZcWMil3q5V3S%2F-LMEPZvScb3EE7uSCyF3%2Fsysmon-running.png?alt=media\&token=afeb21e5-cd91-48ce-a402-ef0a28aa8862)

### References

{% embed url="https://twitter.com/Moti_B/status/1019307375847723008?ref_src=twsrc%5Etfw%7Ctwcamp%5Etweetembed%7Ctwterm%5E1019307375847723008%7Ctwgr%5E5bb5d958e84fdb70bbde7503e4b275ed1742a8a1%7Ctwcon%5Es1_&ref_url=https%3A%2F%2Fcdn.iframe.ly%2FxvY2bCD%3Fapp%3D1" %}

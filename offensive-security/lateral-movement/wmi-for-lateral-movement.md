# WMI for Lateral Movement

## WMI for Lateral Movement

### Execution

Spawning a new process on the target system 10.0.0.6 from another compromised system 10.0.0.2:

{% code title="attacker\@victim" %}
```bash
wmic /node:10.0.0.6 /user:administrator process call create "cmd.exe /c calc"
```
{% endcode %}

### Observations

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI2YKqNExCarC6KPUw1%2F-LI2YnLFrMYHVXmrLb6E%2Fwmic-calc.png?alt=media\&token=3f43d328-3ca3-4b40-b454-812fdf19b6ac)

Inspecting sysmon and windows audit logs, we can see `4648` logon events being logged on the source machine as well as processes being spawned by `WmiPrvSe.exe` on the target host:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI2YKqNExCarC6KPUw1%2F-LI2cZGIbNEPteMiwa5J%2Fwmic-create-cmdline.png?alt=media\&token=1c0dd616-4556-4660-842f-4c9ff1558c61)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI2YKqNExCarC6KPUw1%2F-LI2ca0impieLYNRCoj6%2Fwmic-logon.png?alt=media\&token=259c9450-e2f6-4906-b19a-51784ddda8c8)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI2YKqNExCarC6KPUw1%2F-LI2cbynNlqXXCT96Iod%2Fwmic-spawn.png?alt=media\&token=13f6c18f-7d98-4b05-bc6c-c892e237904f)

Both on the host initiating the connection and on the host that is being logged on to, events `4624` and `4648` should be logged:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LII9iyQTp5nB0U3NPUs%2F-LII9yVSKAg6p1rQDI99%2Fwmi-logons.png?alt=media\&token=4a8a051c-81ac-4ad2-92e0-f2de649ced3f)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1047" %}

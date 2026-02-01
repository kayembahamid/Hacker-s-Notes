# Application Window Discovery

## Application Window Discovery

Retrieving running application window titles:

{% code title="attacker\@victim" %}
```csharp
get-process | where-object {$_.mainwindowtitle -ne ""} | Select-Object mainwindowtitle
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKbpoVQDfki0K4XI0eT%2F-LKbpr09VvDvfZNCrcHK%2Fwindow-titles.png?alt=media\&token=713f1fe3-6f32-4b6c-9d10-e6bc3d96236b)

A COM method that also includes the process path and window location coordinates:

{% code title="attacker\@victim" %}
```csharp
[activator]::CreateInstance([type]::GetTypeFromCLSID("13709620-C279-11CE-A49E-444553540000")).windows()
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LhgXfuwnDlEJeBHbdBa%2F-LhgYUQ4VckbAYvFKANi%2FAnnotation%202019-06-18%20224603.png?alt=media\&token=0eaf6608-3d18-40b9-86f8-08aab70bedfa)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1010" %}

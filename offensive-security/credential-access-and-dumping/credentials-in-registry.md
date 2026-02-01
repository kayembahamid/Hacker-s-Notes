# Credentials in Registry

## Credentials in Registry

### Execution

Scanning registry hives for the value `password`:

{% code title="attacker\@victim" %}
```csharp
reg query HKLM /f password /t REG_SZ /s
# or
reg query HKCU /f password /t REG_SZ /s
```
{% endcode %}

### Observations

As a defender, you may want to monitor commandline argument logs and look for any that include `req query` and `password`strings:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LK6yBehyvtDEgxZBGDA%2F-LK6zKzHNSh8NEFrM0wW%2Fpasswords-registry.png?alt=media\&token=ee7267f5-eae0-47a1-9963-a5ae00bd70c4)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1214" %}

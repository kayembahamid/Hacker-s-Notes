# Alternate Data Streams

## Alternate Data Streams

### Execution

Creating a benign text file:

{% code title="attacker\@victim" %}
```csharp
echo "this is benign" > benign.txt
Get-ChildItem
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJzMlyj0kHmii0Z4nnh%2F-LJzMoqdhLcsCZg5RYja%2Fads-benign.png?alt=media\&token=e0a11877-a806-4fce-ba1a-9d3f07a0a9c6)

Hiding an `evil.txt` file inside the `benign.txt`

{% code title="attacker\@victim" %}
```csharp
cmd '/c echo "this is evil" > benign.txt:evil.txt'
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJzMlyj0kHmii0Z4nnh%2F-LJzMr4IhUYWff8drVRU%2Fads-evil.png?alt=media\&token=ec8681ae-634e-4a49-aef5-0c1491979148)

Note how the evil.txt file is not visible through the explorer - that is because it is in the alternate data stream now. Opening the benign.txt shows no signs of evil.txt. However, the data from evil.txt can still be accessed as shown below in the commandline - `type benign.txt:evil.txt`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJzMlyj0kHmii0Z4nnh%2F-LJzMwYrgjXaANJnds12%2Fads-evil-2.png?alt=media\&token=d73f472b-a5bc-4896-ba5a-79834b4be6e3)

Additionally, we can view the data in the notepad as well by issuing:

{% code title="attacker\@victim" %}
```csharp
notepad .\benign.txt:evil.txt
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJzMlyj0kHmii0Z4nnh%2F-LJzMythJAAqKdLKSwB4%2Fads-evil3.png?alt=media\&token=53183cf2-d09c-4a6b-9db6-ebcec6c62e63)

### Observations

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJzNzQvWBqFNANXAQd0%2F-LJzNwb0DZdbCU39L2t9%2Fads-commandline.png?alt=media\&token=781ae94a-1ab1-4852-b191-79f73edcd68b)

Note that powershell can also help finding alternate data streams:

```csharp
Get-Item c:\experiment\evil.txt -Stream *
Get-Content .\benign.txt -Stream evil.txt
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJzPg_4cKLsDaId6Hns%2F-LJzPdRam1MNcCP92G0J%2Fads-powershell.png?alt=media\&token=1773f259-eebc-44e1-91f7-b54b1361e8bb)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1096" %}

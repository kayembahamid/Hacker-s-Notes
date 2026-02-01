# Powershell Profile Persistence

## Powershell Profile Persistence

It's possible to use powershell profiles for persistence and/or privilege escalation.

### Execution

There are four places you can abuse the powershell profile, depending on the privileges you have:

```csharp
$PROFILE | select *
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LsxaMjnuUA7zmXZp1D6%2F-LsxdF_BjpR3m7AqxhWj%2Fimage.png?alt=media\&token=a90c1df5-ed56-46d3-9b95-ea69e8c5b99b)

Let's add the code to a `$profile` variable (that expands to the current user's profile file) that will get executed the next time the compromised user launches a powershell console:

{% code title="attacker\@target" %}
```csharp
echo "whoami > c:\temp\whoami.txt" > $PROFILE
cat $PROFILE
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LsxaMjnuUA7zmXZp1D6%2F-Lsxb-yXAHbywrB71vFI%2Fimage.png?alt=media\&token=242f2c65-d024-4076-a264-5dc8303ef87c)

Once the compromised user launches powershell, our code gets executed:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LsxaMjnuUA7zmXZp1D6%2F-LsxbCPHx-CvgCFVo9WS%2Fimage.png?alt=media\&token=bb3c4d6a-fad3-4032-baeb-52d015eff54b)

{% hint style="warning" %}
If the user is not using profiles, the technique will stick out immediately due to the "loading personal and system profiles..." message at the top.
{% endhint %}

### References

{% embed url="https://attack.mitre.org/techniques/T1504/" %}

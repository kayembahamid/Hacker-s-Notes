---
description: >-
  This lab explores a security impact of unrestricted kerberos delegation
  enabled on a domain computer.
---

# Kerberos Unconstrained Delegation

Overview

* Unrestricted kerberos delegation is a privilege that can be assigned to a domain computer or a user;
* Usually, this privilege is given to computers (in this lab, it is assigned to a computer IIS01) running services like IIS, MSSQL, etc.;
* Those services usually require access to some back-end database (or some other server), so it can read/modify the database on the authenticated user's behalf;
* When a user authenticates to a computer that has unrestricted kerberos delegation privilege turned on, authenticated user's TGT ticket gets saved to that computer's memory;
* The reason TGTs get cached in memory is so the computer (with delegation rights) can impersonate the authenticated user as and when required for accessing any other services on that user's behalf.

Essentially this looks like so: `User` --- authenticates to ---> `IIS server` ---> authenticates on behalf of the user ---> `DB server`

Any user authentication (i.e CIFS) to the computer with unconstrained delegation enabled on it will cache that user's TGT in memory, which can later be dumped and reused by an adversary.

Setup

Let's give one of our domain computers/our victim computer `IIS01` unrestricted kerberos delegation privilege:

![](<../../.gitbook/assets/image (184)>)

To confirm/find computers on a domain that have unrestricted kerberos delegation property set:

```powershell
Get-ADComputer -Filter {TrustedForDelegation -eq $true -and primarygroupid -eq 515} -Properties trustedfordelegation,serviceprincipalname,description
```

We can see our victim computer `IIS01` with `TrustedForDelegation` field set to `$true` — we are good to attack:

![](<../../.gitbook/assets/image (185)>)

Execution

{% stepper %}
{% step %}
### Inspect IIS01 memory for Kerberos tickets

On the computer IIS01 with kerberos delegation rights, run mimikatz to list tickets:

```
sekurlsa::tickets
```

Example output:

![](<../../.gitbook/assets/image (186)>)

Note: At this point we do not have a TGT for `offense\administrator`.
{% endstep %}

{% step %}
### Trigger authentication from a privileged user

From DC01 (as offense\administrator) send an HTTP request to IIS01 so the privileged user's TGT is cached on IIS01:

```powershell
Invoke-WebRequest http://iis01.offense.local -UseDefaultCredentials -UseBasicParsing
```

You should see an HTTP 200 OK response:

![](<../../.gitbook/assets/image (187)>)
{% endstep %}

{% step %}
### Re-inspect IIS01 memory for the new TGT

Check IIS01 again for kerberos tickets:

```
mimikatz # sekurlsa::tickets
```

You should now see a TGT for `offense\administrator` in IIS01 memory:

![](<../../.gitbook/assets/image (188)>)

This indicates the domain has effectively been compromised.
{% endstep %}

{% step %}
### Export Kerberos tickets from IIS01

Export all kerberos tickets from IIS01 memory so the administrator TGT can be reused:

```
mimikatz::tickets /export
```

Example UI:

![](<../../.gitbook/assets/image (189)>)
{% endstep %}

{% step %}
### Validate current session and import the TGT (Pass-the-Ticket)

Before importing, you can attempt a PSRemoting to DC01 from IIS01 to confirm you currently lack DA rights.

Import the dumped offense\administrator TGT into the current session on IIS01:

```
mimikatz # kerberos::ptt C:\Users\Administrator\Desktop\mimikatz\[0;3c785]-2-0-40e10000-Administrator@krbtgt-OFFENSE.LOCAL.kirbi
```

After importing, check available tickets and connect to DC01 (C$ or PSSession). The session should now contain a `krbtgt` for offense\administrator and enable access with Domain Admin privileges:

![](<../../.gitbook/assets/image (190)>)
{% endstep %}
{% endstepper %}

Reminder

Note that successful authentication to ANY service on the IIS01 will cache the authenticated user's TGT. Below is an example of a user `offense\delegate` accessing a share on `IIS01` — the TGT gets cached:

![](<../../.gitbook/assets/image (191)>)

Mitigation

Some of the available mitigations:

* Disable kerberos delegation where possible.
* Be cautious of whom you give privilege "Enable computer and user accounts to be trusted for delegation" — these are users who can enable unrestricted kerberos delegation.
* Enable "Account is sensitive and cannot be delegated" for high privileged accounts.

References

* [Active Directory Security Risk #101: Kerberos Unconstrained Delegation (or How Compromise of a Single Server Can Compromise the Domain)Active Directory & Azure AD/Entra ID Security](https://adsecurity.org/?p=1667)
* [![Logo](https://www.ired.team/~gitbook/image?url=https%3A%2F%2Fblog.xpnsec.com%2Fimages%2Ffavicon.ico\&width=20\&dpr=4\&quality=100\&sign=5472aee9\&sv=2)@\_xpn\_ - Kerberos AD Attacks - KerberoastingXPN InfoSec Blog](https://blog.xpnsec.com/kerberos-attacks-part-1/)
* [![Logo](https://www.ired.team/~gitbook/image?url=https%3A%2F%2Flearn.microsoft.com%2Ffavicon.ico\&width=20\&dpr=4\&quality=100\&sign=d8acf9f\&sv=2)  Kerberos for the Busy AdminMicrosoftLearn](https://blogs.technet.microsoft.com/askds/2008/03/06/kerberos-for-the-busy-admin/)

Privacy

This site uses cookies to deliver its service and to analyze traffic. By browsing this site, you accept the privacy policy: https://policies.google.com/privacy?hl=en-US

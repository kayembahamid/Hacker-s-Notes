---
description: >-
  It is possible to kerberoast a user account with SPN even if the account
  supports Kerberos AES encryption by requesting an RC4 ecnrypted (instead of
  AES) TGS which easier to crack.
---

# Kerberoasting: Requesting RC4 Encrypted TGS when AES is Enabled

## Execution

{% stepper %}
{% step %}
### Confirm there is a user with an SPN set

Run:

{% code title="PowerShell" %}
```
```
{% endcode %}

![Get-NetUser output](<../../.gitbook/assets/image (33)>)
{% endstep %}

{% step %}
### Request a TGS for a user that does not support AES (RC4 returned)

If the user account does not support Kerberos AES encryption, requesting a TGS for kerberoasting (with Rubeus) will return an RC4-encrypted ticket.

Run:

{% code title="Rubeus" %}
```
```
{% endcode %}

![RC4 TGS returned](<../../.gitbook/assets/image (34)>)
{% endstep %}

{% step %}
### Request a TGS when the user supports AES (AES returned by default)

If the user is configured to support AES encryption, the KDC will by default return tickets encrypted with the highest supported algorithm (AES):

Run:

{% code title="Rubeus" %}
```
```
{% endcode %}

![AES TGS returned](<../../.gitbook/assets/image (35)>)
{% endstep %}
{% endstepper %}

## Requesting RC4 Encrypted Ticket

It's possible to request an RC4-encrypted TGS even when AES is supported by both parties (provided RC4 is not disabled in the environment).

Run:

{% code title="Rubeus" %}
```
```
{% endcode %}

Even though AES is supported, a TGS encrypted with RC4 (enctype 0x17 / 23) can be returned.

{% hint style="warning" %}
Security operations may monitor for RC4-encrypted tickets — RC4 usage can be a detection indicator.
{% endhint %}

![RC4 TGS observed](<../../.gitbook/assets/image (36)>)

## References

*

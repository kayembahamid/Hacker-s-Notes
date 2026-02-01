# AS-REP Roasting

AS-REP roasting is a technique that allows retrieving password hashes for users that have the `Do not require Kerberos preauthentication` property selected:

![](<../../.gitbook/assets/image (66)>)

Those hashes can then be cracked offline, similarly to how it's done in T1208: Kerberoasting.

Execution and cracking process

{% stepper %}
{% step %}
### Gather AS-REP hashes using Rubeus

Run Rubeus to request AS-REP hashes for accounts without Kerberos preauthentication:

{% code title="Rubeus (Windows)" %}
```
```
{% endcode %}

Example output screenshot:

![](<../../.gitbook/assets/image (67)>)
{% endstep %}

{% step %}
### Prepare the hash for Hashcat and crack it

Example AS-REP hash returned:

{% code title="raw AS-REP hash" %}
```
```
{% endcode %}

Insert `23` after `$krb5asrep$` to match Hashcat's expected format (for AES-256-CTS-HMAC-SHA1-96):

{% code title="formatted for Hashcat" %}
```
```
{% endcode %}

Crack with Hashcat (example using mask/wordlist attack):

{% code title="Hashcat (Kali/Linux)" %}
```
```
{% endcode %}

Example screenshots of successful cracking:

![](<../../.gitbook/assets/image (68)>)

![](<../../.gitbook/assets/image (69)>)
{% endstep %}
{% endstepper %}

References

* [![xpnsec logo](https://www.ired.team/~gitbook/image?url=https%3A%2F%2Fblog.xpnsec.com%2Fimages%2Ffavicon.ico\&width=20\&dpr=4\&quality=100\&sign=5472aee9\&sv=2) @_xpn_ - Kerberos AD Attacks - More Roasting with AS-REP](https://blog.xpnsec.com/kerberos-attacks-part-2/)

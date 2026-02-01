# Process Doppelganging

This lab is simply a run of the tool written by @hasherezade that was inspired by the BlackHat talk by Tal Liberman and Eugene Kogan where they presented their research on Process Doppelganging - see [references ](process-doppelganging.md)for their slides.

Process doppelganing is a code injection technique that leverages NTFS transacations related Windows API calls which are (used to be?) less used with malicious intent and hence "less known" to AV vendors, hence this code injection technique is (was?) more likely to go undetected.

Mostly, I wanted to do this lab and see if Windows Defender caught up with this technique or not since the technique has been introduced almost a year ago from the time of this writing.

### Execution

First of, download hasherezade's PoC for doppleganging here [https://github.com/hasherezade/process\_doppelganging](https://github.com/hasherezade/process_doppelganging) and compile it.

Then test the technique like so:

{% code title="attacker\@victim" %}
```csharp
.\process-doppelganger.exe C:\tools\mimikatz\x64\mimikatz.exe c:\zone.txt
```
{% endcode %}

Note in the below screenshot how mimikatz is launched, but the Process Explorer actually represents the mimikatz process as zone.txt - this is because multiple Process Environment Block's (PEB) memory structures of the newly created process were modified during the new process creation:

{% hint style="info" %}
This test was done on Windows 7
{% endhint %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LV3uW8vxoItkQVyeExC%2F-LV3uv6N-_zKHv7FuoSK%2FScreenshot%20from%202018-12-31%2015-37-35.png?alt=media\&token=30616cfc-3707-4a4f-b7f9-c65090c6f68d)

Below are two links where we explore the PEB in a bit more depth:

### Windows 10

Going back to my original motivation as to why I wanted to try this technique out, which was to see if Windows 10 would detect this type of code injection - below is the answer:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LV428E2CuB3ejTjdiN3%2F-LV42B3xmr4HQlVDo1TT%2FScreenshot%20from%202018-12-31%2016-15-21.png?alt=media\&token=3e925f03-f282-438c-a8b1-bdf6bcf91a54)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LV3uW8vxoItkQVyeExC%2F-LV3uwlJthsKMJUhEnsx%2FScreenshot%20from%202018-12-31%2015-35-14.png?alt=media\&token=3ca9e923-a11c-47a0-8b02-c9cb62a46730)

### References

{% embed url="https://hshrzd.wordpress.com/2017/12/18/process-doppelganging-a-new-way-to-impersonate-a-process/" %}

{% embed url="https://github.com/hasherezade/process_doppelganging" %}

{% embed url="https://docs.google.com/viewerng/viewer?url=https://blackhat.com/docs/eu-17/materials/eu-17-Liberman-Lost-In-Transaction-Process-Doppelganging.pdf" %}

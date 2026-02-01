# SIP & Trust Provider Hijacking

## SIP & Trust Provider Hijacking

In this lab, I will try to sign a simple "rogue" powershell script `test-forged.ps1` that only has one line of code, with **Microsoft's** certificate and bypass any whitelisting protections/policies the script may be subject to if it is not signed.

### Execution

The script that I will try to sign:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIvogZaVVwPdchwNqa7%2F-LIwE7eN9lnY4Dpj8jAX%2Ftrust-ps-file.png?alt=media\&token=ea0348a6-ef98-4234-be72-7b2b89df5bfa)

Just before I start, let's make sure that the script is not signed by using a `Get-AuthenticodeSignature` cmdlet and `sigcheck` by SysInternals:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIwJxEHSdxTe0Zgo-Kq%2F-LIwFYiVJlbx7OKNMzjt%2Ftrust-not-signed.png?alt=media\&token=8ff217f2-7535-4e21-b9bc-25f1544f7d55)

In order to sign the script with Microsoft's certificate, we need to first find a native Microsoft Signed PowerShell script. I used powershell for this:

```csharp
Get-ChildItem -Path C:\*.ps* -Recurse -ErrorAction SilentlyContinue | Select-String -Pattern "# SIG # Begin signature block"
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIvogZaVVwPdchwNqa7%2F-LIwBNvbNZVgxLTilLNr%2Ftrust-find-signed.png?alt=media\&token=a6ed806d-e55c-4af5-a41d-b1d0b3d26330)

I chose one script at random and simply checked if it was signed - luckily it was:

```bash
type C:\Windows\WinSxS\x86_microsoft-windows-m..ell-cmdlets-modules_31bf3856ad364e35_10.0.16299.15_none_c7c20f51cd336675\Wdac.psd1
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIvogZaVVwPdchwNqa7%2F-LIwBgYUNAV3fXG_BbwI%2Ftrust-check-if-signing-block-exists.png?alt=media\&token=bb37a289-3bbb-4db4-9360-1b9e37d31777)

Let's copy the Microsoft signature block to my script:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIvogZaVVwPdchwNqa7%2F-LIwEiVim5jHr0cNp2zy%2Ftrust-script-with-ms-signing-code.png?alt=media\&token=0751fd65-7cf9-4836-90cb-8e23b633f5ec)

Now let's modify registry at:

```
HKLM\SOFTWARE\Microsoft\Cryptography\OID\EncodingType 0\CryptSIPDllVerifyIndirectData\{603BCC1F-4B59-4E08-B724-D2C6297EF351}
```

From:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIwJxEHSdxTe0Zgo-Kq%2F-LIwGRdEtj9rGWOXewtL%2Ftrust-from.png?alt=media\&token=3b5d3532-a993-4de8-a75b-a0f009eb9611)

To:

{% code title="DLL" %}
```csharp
C:\Windows\System32\ntdll.dll
```
{% endcode %}

{% code title="FuncName" %}
```
DbgUIContinue
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIwJxEHSdxTe0Zgo-Kq%2F-LIwGRdMiyoMCjLnYiOI%2Ftrust-to.png?alt=media\&token=cb36e3f5-8ff4-4409-9a6d-052f84463465)

Now, let's launch a new powershell instance (for the registry changes to take effect) and check the signature of the forged script - note how it now shows as signed, verified and valid:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIwJxEHSdxTe0Zgo-Kq%2F-LIwJuas1sFFCQdmTElF%2Ftrust-signed.png?alt=media\&token=30dcd81a-de12-406a-a8fe-a795851f91c3)

### Observations

Monitoring the following registry keys/values helps discover this suspicious activity:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIwK6MM-uiPRjrpi5JW%2F-LIwNVODjW9jTKyYx0jg%2Ftrust-sysmon1.png?alt=media\&token=4b28c4f5-81e3-4860-87ba-34eb26ef5ef2)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIwK6MM-uiPRjrpi5JW%2F-LIwNVOHgHXnwKTzw11R%2Ftrust-sysmon2.png?alt=media\&token=24e5fdea-ee4b-40cf-8a2e-0d7cce3c6f88)

### References

For all the registry keys/values that should be used as a baseline, please refer to the original research whitepaper by Matt Graeber:\
[SpecterOps Subverting Trust inWindows](https://specterops.io/assets/resources/SpecterOps_Subverting_Trust_in_Windows.pdf)

{% embed url="https://attack.mitre.org/wiki/Technique/T1198" %}

{% embed url="https://youtu.be/wxmxxgL6Nz8" %}

{% embed url="https://pentestlab.blog/2017/11/06/hijacking-digital-signatures/" %}

{% embed url="http://ultimate-sysadmin-fanboy.blogspot.com/2015/06/unable-to-renew-certificate-via.html" %}

{% embed url="https://blogs.msdn.microsoft.com/sqlforum/2011/01/02/walkthrough-request-a-digital-certificate-from-certificate-server-or-create-a-testing-digital-certificate-to-sign-a-package/" %}

{% embed url="https://youtu.be/WrHTJQovDoY" %}

{% embed url="https://www.hanselman.com/blog/SigningPowerShellScripts.aspx" %}

{% embed url="https://github.com/netbiosX/Digital-Signature-Hijack" %}

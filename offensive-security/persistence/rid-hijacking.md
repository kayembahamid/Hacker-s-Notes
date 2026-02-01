# RID Hijacking

## RID Hijacking

RID (Relative ID, part of the SID (Security Identifier)) hijacking is a persistence technique, where an attacker with SYSTEM level privileges assigns an RID 500 (default Windows administrator account) to some low privileged user, effectively making the low privileged account assume administrator privileges on the next logon.

This techniques was originally researched by [Sebastian Castro](https://twitter.com/r4wd3r) - [https://r4wsecurity.blogspot.com/2017/12/rid-hijacking-maintaining-access-on.html](https://r4wsecurity.blogspot.com/2017/12/rid-hijacking-maintaining-access-on.html)

### Execution

This lab assumes that we've compromised the WS01 machine and have `NT SYSTEM` access to it.

Below shows that the user `hijacked` is a low privileged user and has an RID of 1006 or 0x3ee:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-M0D9G_Ep5sKpO0KCPHV%2F-M0DGJlxfn7mv9ZjMEhU%2Fimage.png?alt=media\&token=03f34684-42e4-4ccd-8503-94fbc57e468e)

If we try to write something to c:\windows\ with the user `hijacked`, as expected, we get `Access is Denied`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-M0D9G_Ep5sKpO0KCPHV%2F-M0DGYZgoeE3NSona3oi%2Fimage.png?alt=media\&token=3a102557-d3c5-4031-9b48-d6b4136fc29c)

HKEY\_LOCAL\_MACHINE\SAM\SAM\Domains\Account\Users\000003EE stores some information about the user`hijacked` that is used by LSASS during the user logon/authentication process. Specifically, at offset `0030` in the value `F` there are bytes that denote user's RID, which in our case are 03ee (1006) for the user `hijacked`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-M0D9G_Ep5sKpO0KCPHV%2F-M0DGeYuTBtkjJYilbhi%2Fimage.png?alt=media\&token=fe5c61e3-2aba-405a-8b55-09c571a939bf)

We can change those 2 bytes to 0x1f4 (500 - default administrator RID), which will effectively make the user `hijacked` assume administrator privileges:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-M0D9G_Ep5sKpO0KCPHV%2F-M0DGtzKXc-5kF_h7ypJ%2Fimage.png?alt=media\&token=35aa028f-be03-4b85-8cb2-b25a44dd78f5)

### Demo

After changing the `hijacked` RID from 3ee to 1f4 and creating a new logon session, we can see that the user `hijacked` is now allowed to write to c:\windows\\, suggesting it now has administrative privileges:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-M0D9G_Ep5sKpO0KCPHV%2F-M0DHZBbuYfbn1bcmp53%2Frid-hijacking.gif?alt=media\&token=bea53568-efb2-468e-bfde-65844609e78c)

Note, that the user `hijacked` still does not belong to local administrators group, but its RID is now 500:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-M0D9G_Ep5sKpO0KCPHV%2F-M0DHoH1TRurXKs55jPX%2Fimage.png?alt=media\&token=860a2595-3673-418c-a03f-b5572d781fd2)

### Detection

Monitor HKEY\_LOCAL\_MACHINE\SAM\SAM\Domains\Account\Users\\\*\F for modifications, especially if they originate from unusual binaries.

### References

{% embed url="https://r4wsecurity.blogspot.com/2017/12/rid-hijacking-maintaining-access-on.html" %}

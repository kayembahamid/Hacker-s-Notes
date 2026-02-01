# API Monitoring and Hooking for Offensive Tooling

## API Monitoring and Hooking for Offensive Tooling

[Rio](https://twitter.com/0x09al) recently posted about his tool [RdpThief](https://www.mdsec.co.uk/2019/11/rdpthief-extracting-clear-text-credentials-from-remote-desktop-clients/) which I thought was plain genius. It allows for offensive operators to steal RDP credentials by injecting RdpThief's DLL into the RDP client mstsc.exe.

Under the hood, RdpThief does the following:

* hooks mstsc.exe functions responsible for dealing with user supplied credentials
* intercepts the user supplied username, password, hostname during authentication
* writes out intercepted credentials and hostname to a file

These are some notes of me tinkering with [API Monitor](http://www.rohitab.com/apimonitor), WinDBG and Detours (Microsoft's library for hooking Windows APIs) and reproducing some of the steps Rio took during his research and development of [RdpThief](https://github.com/0x09AL/RdpThief).

These notes will serve me as a reference for future on how to identify and hook interesting functions that can be useful when writing offensive tooling.

### Walkthrough

If we launch mstsc.exe and attempt connecting to a remote host WS01:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Ltog_hHBfJPZHtUQA1D%2F-Ltoq-WSrDTL57-SWP2q%2Fimage.png?alt=media\&token=58fac8cf-de06-4f49-9b81-a7eed664c3e4)

..we are prompted to enter credentials:

![RDP authentication prompt](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LtoH8X3vBpKJ2HYVq_3%2F-LtoICWKhi6n4vLIoX9T%2Fimage.png?alt=media\&token=3cfbbebb-7853-459e-b5d4-6ee72a6f71b1)

If API monitor was attached to mstsc.exe when we tried to authenticate to the remote host WS01, we should now have a huge list of API calls invoked by mstsc.exe and its module logged.

#### Intercepting Username

If we search for a string `spotless`, we will find some functions that take `spotless` as a string argument and one of those functions is `CredIsMarshaledCredentialW` as shown below:

![CredIsMarshaledCredentialW contains the string spotless](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LtoH8X3vBpKJ2HYVq_3%2F-LtoIrQEpRC2vsoe-FnL%2Ffind-computername.gif?alt=media\&token=30109cdc-d71b-4a85-ae28-227a339c2270)

![CredIsMarshaledCredentialW contains the string spotless](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LtoH8X3vBpKJ2HYVq_3%2F-LtoJN1EEPWn49V5Rfvx%2Fimage.png?alt=media\&token=13007cf9-abff-4a70-874d-baf12ebfc7f8)

In WinDBG, if we put a breakpoint on `ADVAPI32!CredIsMarshaledCredentialW` and print out its first and only argument (stored in RCX register per x64 calling convention), we will see `DESKTOP-NU8QCIB\spotless` printed out:

```c
bp ADVAPI32!CredIsMarshaledCredentialW "du @rcx"
```

![ADVAPI32!CredIsMarshaledCredentialW breakpoint hit and username printed](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LtoH8X3vBpKJ2HYVq_3%2F-LtoNCPFlTFAf7mhllYZ%2Ffind-computername-windbg.gif?alt=media\&token=44c10771-0561-448e-bde2-a7ba1c6fc5f3)

![ADVAPI32!CredIsMarshaledCredentialW breakpoint hit and username printed - still](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LtoH8X3vBpKJ2HYVq_3%2F-LtocXXvju0xNlGPkIHx%2Fimage.png?alt=media\&token=44fbb915-7b15-4f19-8874-d0ad22a5f24b)

#### Intercepting Hostname

To find the hostname of the RDP connection, we find API calls that took `ws01` (our hostname) as a string argument. Although RdpThief hooks `SSPICLI!SspiPrepareForCredRead` (hostname supplied as a second argument), another function that could be considered for hooking is `CredReadW` (hostname a the first argument) as seen below:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Ltog_hHBfJPZHtUQA1D%2F-LtolUHuZarCHEzqLtzr%2Fimage.png?alt=media\&token=76d931fc-3796-4637-ba91-600e9f45507e)

If we jump back to WinDBG and set another breakpoint for `CredReadW` and attempt to RDP to our host `ws01`, we get a hit:

```cpp
bp ADVAPI32!CredReadW "du @rcx"
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Ltog_hHBfJPZHtUQA1D%2F-LtomvCgXk8N60c7fs1o%2Fimage.png?alt=media\&token=1e9e3c21-60d7-497f-b480-78a67730ffd1)

Out of curiosity, let's also put a breakpoint on `SSPICLI!SspiPrepareForCredRead` and once it's hit, print out the second argument supplied to the function, which is stored in the RDX register:

```
bp SSPICLI!SspiPrepareForCredRead
du @rdx
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Ltog_hHBfJPZHtUQA1D%2F-Ltop6GzwUiYniCqUIiW%2Fimage.png?alt=media\&token=8b139f4f-c6f4-4116-9ad8-bdb279dc2f80)

#### Intercepting Password

We now know the functions required to hook for intercepting the username and the hostname. What's left is hooking the function that deals in one way or another with the password and from Rio's article, we know it's the DPAPI `CryptProtectMemory`.

Weirdly, searching for my password in API Monitor resulted in no results although I could see it in plain text in `CryptUnprotectMemory`:

![Password not found in API Monitor when using search, although the password is clearly there](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LttBwl_J6aaMO0E93n8%2F-LttCAV62xP9OLwR8MiS%2Fimage.png?alt=media\&token=5569b8f2-c5f1-44e2-8aac-d06fb90f93c4)

![Plain text password visible in CryptUnprotectMemory](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Ltog_hHBfJPZHtUQA1D%2F-Ltp4roYB1MHFsyC8N8u%2Fimage.png?alt=media\&token=1b5dc870-6bba-4601-846f-5a28a29a4cbb)

Reviewing `CryptProtectMemory` calls manually in API Monitor showed no plaintext password either, although there were multiple calls to the function and I would see the password already encrypted:

![32 byte encrypted binary blob](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Ltog_hHBfJPZHtUQA1D%2F-Ltp4agKygYEqjsSBmk4%2Fimage.png?alt=media\&token=320c4838-c809-4ec5-bdc9-48d69430d20c)

{% hint style="info" %}
From the above screenshot, note the size of the encrypted blob is 32 bytes - we will come back to this in WinDBG
{% endhint %}

While having issues with API Monitor, let's put a breakpoint on `CryptProtectMemory` in WinDBG and print out a unicode string (this should be the plaintext password passed to the function for encryption) starting 4 bytes into the address (first 4 bytes indicate the size of the encrypted data) pointed by the RCX register:

```cpp
bp dpapi!cryptprotectmemory "du @rcx+4"
```

Below shows the plain text password on a second break:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Ltog_hHBfJPZHtUQA1D%2F-Ltp157dviyvshSdvgb-%2Fcapture-password.gif?alt=media\&token=cc55999e-120b-4ddd-9065-0da1acfe4840)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Ltog_hHBfJPZHtUQA1D%2F-Ltp1Bv0sY4tWfgkxTUE%2Fimage.png?alt=media\&token=a2643934-2cf1-423e-ac6f-6df72d5903e5)

Earlier, I noted the 32 bytes encrypted blob seen in `CryptProtectMemory` function call (in API Monitor) and also mentioned the 4 byte offset into RCX that holds the size of the encrypted blob - below shows that - first 4 bytes found at RCX (during the `CryptProtectMemory` break) are 0x20 or 32 in decimal:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Ltog_hHBfJPZHtUQA1D%2F-Ltp8mpgN8za_jYTKwkA%2Fimage.png?alt=media\&token=0477837a-c241-4835-bca7-b52f8ae0b5e1)

### RdpThief in Action

Compiling RdpThief provides us with 2 DLLs for 32 and 64 bit architectures. Let's inject the 64 bit DLL into mstsc.exe and attempt to RDP into `ws01` - we see the credentials getting intercepted and written to a file:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Ltt4ssWl8i3aZnVFiJ_%2F-Ltt5vPbSA2fvZJ-PJ7p%2Finject-rdp-thief.gif?alt=media\&token=3f60ffc0-f785-48de-b454-b3e7ebeb554b)

### Intercepting Hostname via CredReadW

I wanted to confirm if my previous hypothesis about hooking `CredReadW` for intercepting the hostname was possible, so I made some quick changes to the RdpThief's project to test it.

I commented out the `_SspiPrepareForCredRead` signature and hooked `CreadReadW` with a new function called `HookedCredReadW` which will pop a message box each time `CredReadW` is called and print its first argument as the message box text.

Also, it will update the `lpServer` variable which is later written to the file creds.txt together with the username and password.

Below screenshot shows the code changes:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LtplzeQ5Gf1sBIALzvI%2F-LtqGXARtq-2WoojdPoD%2Fimage.png?alt=media\&token=682cfd0b-9cad-46a6-8053-58215afa571b)

Of course, we need to register the new hook `HookedCredReadW` and unregister the old hook `_SspiPrepareForCredRead`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LtplzeQ5Gf1sBIALzvI%2F-LtqHQWl0d3F1lQ5pKiL%2Fimage.png?alt=media\&token=04236f0e-caed-44b8-b5a2-d348300100a1)

Compiling and injecting the new RdpThief DLL confirms that the `CredReadW` can be used to intercept the the hostname:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Ltt4ssWl8i3aZnVFiJ_%2F-Ltt5PkI1nnKvAZRsXiH%2Finject-rdp-thief-credreadw.gif?alt=media\&token=0de25b76-c94e-43d1-8256-af7d020c7091)

### References

{% embed url="https://www.mdsec.co.uk/2019/11/rdpthief-extracting-clear-text-credentials-from-remote-desktop-clients/" %}

{% embed url="https://github.com/0x09AL/RdpThief" %}

{% embed url="https://docs.microsoft.com/en-us/cpp/build/x64-calling-convention?view=vs-2019" %}

{% embed url="https://docs.microsoft.com/en-us/windows/win32/api/wincred/nf-wincred-credismarshaledcredentialw" %}

{% embed url="https://docs.microsoft.com/en-us/dotnet/framework/tools/developer-command-prompt-for-vs#manually-locate-the-files-on-your-machine" %}




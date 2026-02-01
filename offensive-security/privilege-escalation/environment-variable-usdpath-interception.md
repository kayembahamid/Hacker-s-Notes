# Environment Variable $Path Interception

## Environment Variable $Path Interception

It's possible to abuse `$PATH` environment variable to elevate privileges if the variable:

* contains a folder that a malicious user can write to
* that folder precedes c:\windows\system32\\

Below is an example, showing how c:\temp precedes c:\windows\system32:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-M-5kbPi6WIoZENaFZ-g%2F-M-6g5S7OCf5zgbpgmAF%2Fimage.png?alt=media\&token=a22692ca-ec14-4eb7-bdba-47bbb05cae83)

Let's make sure c:\temp is (M)odifiable by low privileged users:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-M-5kbPi6WIoZENaFZ-g%2F-M-6gtfYvAlaGui6WqmM%2Fimage.png?alt=media\&token=de7dc975-b6ef-4575-8832-a6a3df22a4f8)

Let's now drop our malicious file (calc.exe in this case) into c:\temp and call it cmd.exe:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-M-5kbPi6WIoZENaFZ-g%2F-M-6g9EKrA-l-PulxAeQ%2Fimage.png?alt=media\&token=97348e52-fb91-454c-8db4-96ef3625fdc4)

Now, the next time a high privileged user invokes cmd.exe, our malicious cmd.exe will be invoked from the c:\temp:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-M-5kbPi6WIoZENaFZ-g%2F-M-6gLUaw1TB79moRZ3w%2Fimage.png?alt=media\&token=07a22d49-2718-4ac4-828b-f5de4489330a)

This can be very easily abused in environments where software deployment packages call powershell, cmd, cscript and other similar system binaries with `NT SYSTEM` privileges to carry out their tasks.

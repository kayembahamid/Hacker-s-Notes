# Executing Code as a Control Panel Item through an Exported Cplapplet Function

This is a quick note that shows how to execute code in a .cpl file, which is a regular DLL file representing a Control Panel item.

The .cpl file needs to export a function `CplApplet` in order to be recognized by Windows as a Control Panel item.

Once the DLL is compiled and renamed to .CPL, it can simply be double clicked and executed like a regular Windows .exe file.

## Code

item.cpl

```cpp
// dllmain.cpp : Defines the entry point for the DLL application.
#include "stdafx.h"
#include <Windows.h>

//Cplapplet
extern "C" __declspec(dllexport) LONG Cplapplet(
	HWND hwndCpl,
	UINT msg,
	LPARAM lParam1,
	LPARAM lParam2
)
{
	MessageBoxA(NULL, "Hey there, I am now your control panel item you know.", "Control Panel", 0);
	return 1;
}

BOOL APIENTRY DllMain( HMODULE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
                     )
{
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:
	{
		Cplapplet(NULL, NULL, NULL, NULL);
	}
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}
```

Once the DLL is compiled, we can see our exported function `Cplapplet`:

![](<../../.gitbook/assets/image (1359)>)

## Demo

Below shows that double-clicking the .cpl item is enough to launch it:

![](https://www.ired.team/~gitbook/image?url=https%3A%2F%2F386337598-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-legacy-files%2Fo%2Fassets%252F-LFEMnER3fywgFHoroYn%252F-LrkBJ0N_SxV9o7zZMKE%252Fcplexecution.gif%3Falt%3Dmedia%26token%3Dd28e3a80-c691-4311-ace0-81ff5a13fe41\&width=768\&dpr=4\&quality=100\&sign=9f98a82b\&sv=2)

![](<../../.gitbook/assets/image (1360)>)

CPL file can also be launched with:

* Using control.exe:
  * Example: control.exe \<pathtothe.cpl>

![](<../../.gitbook/assets/image (1361)>)

* Or with rundll32:
  * Example:

```
rundll32 shell32, Control_RunDLL \\VBOXSVR\Experiments\cpldoubleclick\cpldoubleclick\Debug\cpldoubleclick.cpl
```

![](<../../.gitbook/assets/image (1362)>)

## References

{% embed url="https://www.fireeye.com/blog/threat-research/2019/10/staying-hidden-on-the-endpoint-evading-detection-with-shellcode.html" %}

{% embed url="https://github.com/fireeye/DueDLLigence/blob/master/DueDLLigence/DueDLLigence.cs" %}

{% embed url="https://docs.microsoft.com/en-us/windows/win32/shell/using-cplapplet" %}

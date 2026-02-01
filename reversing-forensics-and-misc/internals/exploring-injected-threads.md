# Exploring Injected Threads

## Exploring Injected Threads

### Injecting Shellcode

Firstly, let's use an [injector](../../offensive-security/code-and-process-injection/createremotethread-shellcode-injection.md) program we wrote earlier to inject some shellcode into a process that will give us a reverse shell. In this case, we are injecting the shellcode into explorer.exe:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNHAyyVaVz1BZv5MhSW%2F-LNH8rtdFUuFn3RwRHfE%2Finjected-threads-explorer-injected.png?alt=media\&token=1d4d7c27-41d7-403f-8c87-111fd882f898)

### Detecting Injection

Now that we have injected the code into a new thread of the explorer.exe process, let's scan all the running processes for any injected threads using [Get-InjectedThreads.ps1](https://gist.github.com/jaredcatkinson/23905d34537ce4b5b1818c3e6405c1d2):

```csharp
$a = Get-InjectedThread; $a
```

Looks like the injected thread was successfully detected:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNHAyyVaVz1BZv5MhSW%2F-LNH941sA6haZhT98zWv%2Finjected-threads-get-injected-thread.png?alt=media\&token=c2e14831-5d1d-4a5b-ac62-390d70bdea88)

### Cross-checking Shellcode

Lets check the payload found in the injected thread:

```csharp
($a.Bytes | ForEach-Object tostring x2) -join "\x"
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNHAyyVaVz1BZv5MhSW%2F-LNHAMlWP8o8ep2Bu1J7%2Finjected-threads-shellcode2.png?alt=media\&token=aded04c9-1910-4668-8752-072111a7653f)

and cross-verify it with the shellcode specified in our injector binary. We see they match as expected:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNHAyyVaVz1BZv5MhSW%2F-LNHAvZXUf0wxyCwYzC1%2Finjected-threads-shellcode.png?alt=media\&token=6d0000e7-3e33-4253-b3b4-2c3cb874f878)

### Inspecting with WinDBG

In order to inspect the newly created thread that executes the above shellcode with WinDBG, we need to know the injected thread id. For this, we use Process Explorer and note the newly created thread's ID which is `2112`. Note the `ThreadId` is also shown in the output of Get-InjectedThread powershell script:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNHAyyVaVz1BZv5MhSW%2F-LNHPTxzc_i9IFhRlmYC%2Finjected-threads-threadid.png?alt=media\&token=13a4dbdb-7570-4b02-b92a-25b93d533187)

We can get all the threads for a process being debugged in WinDBG with `~` command:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNHAyyVaVz1BZv5MhSW%2F-LNHQrwc2shPT0gHpwWQ%2Finjected-threads-threadid-windbg.png?alt=media\&token=6ba51833-380c-44b4-9316-f20ab48f5b26)

Additionally, in order to inspect the bytes stored/executed in the injected thread, we need to get the thread's `StartAddress` which can be retrieved with `~.` command when in the context of the thread of interest.

Below graphic shows the injected thread's contents with WinDBG:

![Injected thread id + StartAddress + content bytes](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNHAyyVaVz1BZv5MhSW%2F-LNHSL9WypWnMoz4lbEg%2Finjected-threads-inspection.png?alt=media\&token=1f875c49-c365-4f20-ab2e-3bef9f119842)

The above also highlights the thread `0x1494 = 5268` ID. That thread is then inspected for its `StartAddress`, which happened to be `0x03730000 = 57868288`.

For reference, the original shellcode bytes are displayed in the upper right corner. Bottom right corner shows the output of the `Get-InjectedThreads` indicating `ThreadId` and `StartAddress` in decimal.

### How Get-InjectedThreads detects code injection?

One of the things Get-InjectedThreads does in order to detect code injection is:

* it enumerates all the threads in each running process on the system
* performs the following checks on memory regions holding those threads: `MemoryType == MEM_IMAGE && MemoryState == MEM_COMMIT`
* If the condition is not met, it means that the code, running from the thread being inspected, does not have a corresponding image file on the disk, suggesting the code may be injected directly to memory.

Below graphic shows details of the memory region containing the injected thread using WinDBG and Get-InjectedThreads. Note the Type/MemoryType and State/MemoryState in WinDBG/Get-InjectedThreads outputs respectively:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNHU-cCcTUbUDllaoqZ%2F-LNHVFMaSP5Ko3Gekrfi%2Finjected-threads-address.png?alt=media\&token=6e662f45-a1b6-4a2f-aac3-6d47b2e307c1)

### References

{% embed url="https://posts.specterops.io/defenders-think-in-graphs-too-part-1-572524c71e91" %}

{% embed url="https://blog.xpnsec.com/undersanding-and-evading-get-injectedthread/" %}


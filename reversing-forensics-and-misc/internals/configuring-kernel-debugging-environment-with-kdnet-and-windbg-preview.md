# Configuring Kernel Debugging Environment with kdnet and WinDBG Preview

## Configuring Kernel Debugging Environment with kdnet and WinDBG Preview

This is a quick note showing how to start debugging Windows kernel using [kdnet.exe](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools) and WinDBG Preview (the new WinDBG you can get from the Windows Store).

### Terms

* Debugger - local host on which WinDBG will run. In my case a host with IP `192.168.2.79`
* Debuggee - remote host which will be debugged by the host running the debugger. In my case - a host with IP `192.168.2.68`

### On the Debuggee

Copy over kdnet.exe and VerifiedNICList.xml to the debugee host. Get these files from a host that has Windows Development Kit installed, in C:\Program Files (x86)\Windows Kits\10\Debuggers\x64:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LuCn_MLcBelo1snfXEQ%2F-LuNdhSLVr7U34xj_FMC%2Fimage.png?alt=media\&token=2512cfb8-16ab-451d-978f-37cefe3f5046)

Then in an elevated prompt:

```
kdnet 192.168.2.79 50001
```

The bewlow shows how kdnet prints out the command that needs to be run on the debugger host:

```
windbg -k net:port=50001,key=1dk3k2bprui6m.26vzkoub4jmjl.3v6rvfqjys3ek.6kyxal1u1w6s
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LuCn_MLcBelo1snfXEQ%2F-LuNaX5_x7XbtOW6ZxJR%2Fimage.png?alt=media\&token=85862b9a-9b40-499b-a41f-251b5d080bb8)

Copy and paste to a notepad and reboot the debugee.

### On the Debugger

In WinDBG Preview, navigate to: start debugging > attach to kernel and enter the port and the key you got from running the kdnet on the debugge host:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LuCn_MLcBelo1snfXEQ%2F-LuNagr0RH-csTsDZ41C%2Fimage.png?alt=media\&token=1db25260-0922-4caf-a641-e65cacd63c4c)

Click OK and you should now be ready to start debugging the host `192.168.2.68`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LuCn_MLcBelo1snfXEQ%2F-LuNbM9Om9DSLeuUrPU8%2Fkerneldebuggingconnect.gif?alt=media\&token=fe99e7f6-7b58-49a5-9d42-b35870f6249f)

### References

{% embed url="https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/setting-up-a-network-debugging-connection-automatically" %}


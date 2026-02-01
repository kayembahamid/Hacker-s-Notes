# WebShells

## WebShells

This demo assumes a server compromise and that the attacker has already uploaded a webshell to the compromised host for persistence.

### Execution

Below illustrates the existence of a simple webshell on a compromised Windows 2008R at 10.0.0.6 running IIS web service. It also shows output of the classic system enumeration commands - `net`, `whoami`, `ipconfig`, etc:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIlnaagE46XJkGt_-MR%2F-LIlormD3ElNfCK64DXd%2Fwebshell-attacker.png?alt=media\&token=e3a4243a-74e2-4e98-a6e2-6e8777e5a6de)

### Observations

Note that this particular webshell's HTTP requests are sent to the webserver via POST method which means that looking at the IIS web logs will not allow you to see what commands were executed using the webshell. The only things you will just will be a bunch of POST requests to the `c.aspx` file:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIlnaagE46XJkGt_-MR%2F-LIlorn6ehx0vOHTisXX%2Fwebshell-iis-logs.png?alt=media\&token=eee18047-c233-4072-a669-60f9fbc58ab8)

However, if you are collecting network traffic data, you can see the attacker's commands and their outputs:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIlnaagE46XJkGt_-MR%2F-LIlormZDTYrEyjlL0fh%2Fwebshell-pcap.png?alt=media\&token=cf45f4f0-5ab5-453f-850d-edd08a8fe9f1)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIlshpT98F20tlO_-qS%2F-LIlseogsMAGXqsx7_QX%2Fwebshell-stream.png?alt=media\&token=95902b76-2f0c-41e5-8092-97c104ae41d6)

Looking at sysmon process creation logs, we can immediately identify nefarious behaviour - we can see multiple enumeration commands being invoked from `c:\windows\system\inetsrv` working directory under a `ISS\APPOOL\DefaultAppPool` user - this should not happen under normal circumstances and should raise your suspicion:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIlnaagE46XJkGt_-MR%2F-LIlormx2iVGvOhoXyNG%2Fwebshell-sysmon.png?alt=media\&token=8d104682-8c20-4099-b265-c6d99f5415b1)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1108" %}

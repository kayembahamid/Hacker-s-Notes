---
description: Sticky keys backdoor.
---

# Sticky Keys

## Sticky Keys

### Execution

Replace the originali sethc.exe with a cmd.exe and rename it. You may need to change sethc.exe owner to yourself first as TrustedIntaller may be giving you a hard time:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI2jjlA_Tm1Gh8wfndr%2F-LI2keQv_N9EOG3hhqnY%2Fsethc-trustedinstaller.png?alt=media\&token=051c1a10-4ccc-4d2d-8ac1-434ac0d6a917)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI2jjlA_Tm1Gh8wfndr%2F-LI2kIQx0zYFEUarD5nO%2Fsethc-backdoor.png?alt=media\&token=4485d49b-6cc1-449d-b8c6-4064ffb3a899)

Hit shift 5 times while on the logon screen to invoke the backdoor:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI2lcRc5H6xZS3qi_Bd%2F-LI2liUhh3mtk4gAw6lD%2Fsethc-logon.png?alt=media\&token=24abeec6-4dcb-49fb-8d77-e2d92524d0ce)

### Observations

If you notice sethc.exe spawning well known windows processes, you may want to investigate the endpoint further:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LI2jjlA_Tm1Gh8wfndr%2F-LI2juUGYZvXZn5Au1un%2Fsethc-enumeration.png?alt=media\&token=3e795c64-ccb6-4591-8bbe-da2376f21745)

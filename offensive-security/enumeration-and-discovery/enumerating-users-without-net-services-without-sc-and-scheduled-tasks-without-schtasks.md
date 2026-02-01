# Enumerating Users without net, Services without sc and Scheduled Tasks without schtasks

## Enumerating Users without net, Services without sc and Scheduled Tasks without schtasks

It is possible to use MMC snap-ins to enumerate local users and local groups, services, scheduled tasks, SMB shares and sessions on a system if you have an interactive desktop session on the compromised system either via RDP or if you are simulating an insider threat during a pentest and you are given a company's laptop.

### Why would you do it?

The use of well known lolbins like net, sc and schtasks on a host where an EDR solution is running is risky and may get you caught. Using snap-ins may help evade commandline detections SOC may be relying on.

Of course, marketing department is unlikely to run mmc snap-ins either, so beware :)

### Enumerating Users and Local Groups

Launch mmc.exe, click File > Add\remove snap-in > Local users and Groups:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lf1MiFENTsu8-m11hRx%2F-Lf1Op9_0cnfJxnro7x5%2Fsnapin.gif?alt=media\&token=114de5b5-b597-41fe-b8d1-91089093d488)

### Enumerating Services

Same could be done for enumerating services running on the system:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lf1MiFENTsu8-m11hRx%2F-Lf1QnvmvBwseLc7yPwH%2Fsnapins.PNG?alt=media\&token=0a8397cf-f2d8-41d8-ad98-05fe8f526476)

Note that `services.msc` could give you the same view.

### Enumerating Scheduled Tasks

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lf1MiFENTsu8-m11hRx%2F-Lf1R9I_NshCEr5P11Ae%2Ftasksch.PNG?alt=media\&token=c350a87d-19b3-4914-a688-73df63f777ce)

Persistence anyone? Note that `taskschd.msc` could give you the same view:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lf1MiFENTsu8-m11hRx%2F-Lf1dOpeQPouvStO9hpN%2Fscheduler-new-task.PNG?alt=media\&token=e9533ee3-cf71-4ed6-824f-4d840c2561fb)

### Shares and Sessions

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lf1MiFENTsu8-m11hRx%2F-Lf1ghA2LFoTaIrnlwAJ%2Fsessions%2Bshares.PNG?alt=media\&token=20aad1a7-9a34-4fbb-8d0e-c1f06934c599)

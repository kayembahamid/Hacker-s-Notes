---
description: Code execution, privilege escalation, lateral movement and persitence.
---

# Schtask

## Schtask

### Execution

Creating a new scheduled task that will launch shell.cmd every minute:

{% code title="attacker\@victim" %}
```bash
schtasks /create /sc minute /mo 1 /tn "eviltask" /tr C:\tools\shell.cmd /ru "SYSTEM"
```
{% endcode %}

### Observations

Note that processes spawned as scheduled tasks have `taskeng.exe` process as their parent:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHy6CND-BeNpqKGubrx%2F-LHy68PyzZEhWDrkWOlD%2Fschtask-ancestry.png?alt=media\&token=bc74d939-a1a4-4604-b1ae-91b54e2e0c1a)

Monitoring and inspecting commandline arguments and established network connections by processes can help uncover suspicious activity:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHyEll55_v99wBksst0%2F-LHz01Kwl65cvw5hpiqi%2Fschtasks-created.png?alt=media\&token=32add7ba-243a-4456-82af-c902831bf99a)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHyEll55_v99wBksst0%2F-LHyzlcSRavyqDnRP4qt%2Fschtask-connection.png?alt=media\&token=0c355274-8594-4468-8f55-23781d5fc185)

Also, look for events 4698 indicating new scheduled task creation:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHz95WkxrtC27q5M76t%2F-LHz92WU-MFxxlBGMnRi%2Fschtasks-created-new-task.png?alt=media\&token=b8515270-e708-4318-b0cc-f962b4d71355)

#### Lateral Movement

Note that when using schtasks for lateral movement, the processes spawned do not have taskeng.exe as their parent, rather - svchost:

{% code title="attacker\@victim" %}
```bash
schtasks /create /sc minute /mo 1 /tn "eviltask" /tr calc /ru "SYSTEM" /s dc-mantvydas /u user /p password
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHz3KC9abTXNhyE2hiX%2F-LHz3Z0gLOeaUDeDoNqv%2Fschtasks-remote.png?alt=media\&token=89edb7e6-b88a-452b-a318-7456a74ba801)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1053" %}

{% embed url="https://docs.microsoft.com/en-us/windows/desktop/taskschd/schtasks" %}

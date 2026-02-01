# Lateral Movement over headless RDP with SharpRDP

## Lateral Movement over headless RDP with SharpRDP

Executing commands on a remote host is possible by using a headless (non-GUI) RDP lateral movement technique brought by a tool called [SharpRDP](https://posts.specterops.io/revisiting-remote-desktop-lateral-movement-8fb905cb46c3?gi=fe80458d82a5).

### Execution

Executing a binary on a remote machine dc01 from a compromised system with offense\administrator credentials:

```
SharpRDP.exe computername=dc01 command=calc username=offense\administrator password=123456
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LzsHuH78mpg7jbWKO2N%2F-LzsKSQef9AhBoU8PjVd%2Fimage.png?alt=media\&token=d65bc5dc-8ce3-4ffa-a8fe-d032acc3084f)

### Observations

Defenders may want to look for mstscax.dll module being loaded by suspicious binaries on a compromised host from which SharpRDP is being executed:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LzsHuH78mpg7jbWKO2N%2F-LzsLPGoHpbVX3GWqreo%2Fimage.png?alt=media\&token=c0638669-c8d7-4fc9-a2f7-85d222bbf90e)

Also, weird binaries making connections to port 3389:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LzsHuH78mpg7jbWKO2N%2F-LzsLbkMUJKyV6dibir4%2Fimage.png?alt=media\&token=62b7b16a-dc34-4188-a499-551fb67ef26e)

### References

{% embed url="https://posts.specterops.io/revisiting-remote-desktop-lateral-movement-8fb905cb46c3?gi=fe80458d82a5" %}

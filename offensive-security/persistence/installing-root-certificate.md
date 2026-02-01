# Installing Root Certificate

## Installing Root Certificate

### Execution

Adding a certificate with a native windows binary:

{% code title="attacker\@victim" %}
```csharp
certutil.exe -addstore -f -user Root C:\Users\spot\Downloads\certnew.cer
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJynKsn7J7D5s7mLbwz%2F-LJyqMckKQO2kEqqc1UV%2Fcerts-certutil.png?alt=media\&token=6c098d3c-c3f0-49f1-a284-248db726315d)

Checking to see the certificate got installed:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJynKsn7J7D5s7mLbwz%2F-LJyqQ2VqLB4JYJDrAnZ%2Fcerts-installed.png?alt=media\&token=6f6451bb-08c7-42fe-8bfd-d396a77e8a92)

Adding the certificate with powershell:

{% code title="attacker\@victim" %}
```csharp
Import-Certificate -FilePath C:\Users\spot\Downloads\certnew.cer -CertStoreLocation Cert:\CurrentUser\Root\
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJynKsn7J7D5s7mLbwz%2F-LJyqbycoKnz6uhQe5WO%2Fcerts-add-with-ps.png?alt=media\&token=977a73c0-9160-4753-88d1-6e22cd426b89)

### Observations

Advanced poweshell logging to the rescue:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJynKsn7J7D5s7mLbwz%2F-LJyqg-QzrDfu7RLZY3p%2Fcerts-ps-logging.png?alt=media\&token=f97b0c60-a6f4-4c5a-b6f8-0bbd4d167908)

Commandline logging:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJynKsn7J7D5s7mLbwz%2F-LJyr4S1zYBs87IneLGr%2Fcerts-logs.png?alt=media\&token=adeef754-0099-4b94-bb12-d96ec058d34f)

The CAs get installed to:

```csharp
Computer\HKEY_CURRENT_USER\Software\Microsoft\SystemCertificates\Root\Certificates\C6B22A75B0633E76C9F21A81F2EE6E991F5C94AE
```

..so it is worth monitoring registry changes there:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJz3ScYEAr42Csq9UcK%2F-LJz3Pa7Z4DINdAebLKM%2Fcerts-registry.png?alt=media\&token=31ca5d98-42de-4bef-9a9b-03c8c55d7d06)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1130" %}

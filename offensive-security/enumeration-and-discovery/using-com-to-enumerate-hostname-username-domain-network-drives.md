# Using COM to Enumerate Hostname, Username, Domain, Network Drives

## Using COM to Enumerate Hostname, Username, Domain, Network Drives

At `Computer\HKEY_CLASSES_ROOT\CLSID\{093FF999-1EA0-4079-9525-9614C3504B74}` we have a **Windows Script Host Network Object** COM object which allows us to get details such as computer name, logged on user, etc:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LhgRZqMtBMIucwA41cH%2F-LhgSmE3vR77MHTLYghW%2FAnnotation%202019-06-18%20222057.png?alt=media\&token=9b2dac58-9e82-414d-b24a-3e3defd5eb5b)

```csharp
$o = [activator]::CreateInstance([type]::GetTypeFromCLSID("093FF999-1EA0-4079-9525-9614C3504B74"))
```

Below are all the properties and methods exposed by the object:

```csharp
$o | gm
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LhgRZqMtBMIucwA41cH%2F-LhgSGX5CTpm_QAb69p7%2FAnnotation%202019-06-18%20221846.png?alt=media\&token=28ed5720-8396-4c7e-8709-55bb11e031ed)

Viewing username, domain, machine name, etc:

```
$o
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LhgRZqMtBMIucwA41cH%2F-LhgSZXYB91veMtmtLoh%2FAnnotation%202019-06-18%20221927.png?alt=media\&token=c8491b35-42f8-49a8-a4dd-888bc5e768ea)

We can also see any network connected drives:

```
$o.EnumNetworkDrives()
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LhgRZqMtBMIucwA41cH%2F-LhgSXKYZ8Y0bh9Vghmz%2FAnnotation%202019-06-18%20221949.png?alt=media\&token=38f54690-52f8-4cfe-b10c-4c1d0ed779dd)

### Observations

Below shows what additional modules Powershell loads once the COM object is instantiated:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LhgjdHSpFBhijqq8Szo%2F-Lhgji5h6t4Grez-J3oK%2Floaded-dlls.gif?alt=media\&token=b696a1df-9864-4b65-a350-91fb549aee25)

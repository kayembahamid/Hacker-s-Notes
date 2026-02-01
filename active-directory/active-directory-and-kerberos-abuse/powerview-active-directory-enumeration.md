# PowerView: Active Directory Enumeration

This lab explores a couple of common cmdlets of PowerView that allows for Active Directory/Domain enumeration.

### Get-NetDomain

Get current user's domain:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLzcu9yUBckhGDnpEL1%2F-LLzd9N9rZOj1zqIhSBF%2Fpowerview-getnetdomain.png?alt=media\&token=2d96e2bd-a65e-4eca-a823-475392712b2d)

### Get-NetForest

Get information about the forest the current user's domain is in:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLzcu9yUBckhGDnpEL1%2F-LLzddnEPdmu7Q8SfouR%2Fpowerview-forestinfo.png?alt=media\&token=39331786-3ea3-4879-a3a1-809c9d1cf349)

### Get-NetForestDomain

Get all domains of the forest the current user is in:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLzcu9yUBckhGDnpEL1%2F-LLzf0S10E1OSY7T7k9H%2Fpowerview-forest-domains.png?alt=media\&token=1c918630-aa6f-4b2e-8ec0-f15db28d59a5)

### Get-NetDomainController

Get info about the DC of the domain the current user belongs to:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLzcu9yUBckhGDnpEL1%2F-LLzfOFLgrOxW6Y-bR52%2Fpowerview-getdc.png?alt=media\&token=a1cf8a65-5cd7-46ff-9bda-07805a09aa4e)

### Get-NetGroupMember

Get a list of domain members that belong to a given group:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLzg0oOoEp1IZFTfNWi%2F-LLzgA_HPmbcYClpCNOt%2Fpowerview-groups.png?alt=media\&token=afd61ad7-fd32-41fd-b55f-4570941a9135)

### Get-NetLoggedon

Get users that are logged on to a given computer:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLzg0oOoEp1IZFTfNWi%2F-LLzhPeRsZfpet96kWuT%2Fpowerview-connected-users.png?alt=media\&token=dc17d021-11e7-4958-a18f-0dc81f3c657b)

### Get-NetDomainTrust

Enumerate domain trust relationships of the current user's domain:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLzleS4g0dxZT8fVUSL%2F-LLzhpw5yYwcsbZ5Arzk%2Fpowerview-domain-trusts.png?alt=media\&token=2629195f-b4f1-4a5e-b288-838eefea6131)

### Get-NetForestTrust

Enumerate forest trusts from the current domain's perspective:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLzleS4g0dxZT8fVUSL%2F-LLzi97c12Py-wn6iGz1%2Fpowerview-foresttrusts.png?alt=media\&token=8b4de389-39f0-4b29-b5e6-2769465f6a3e)

### Get-NetProcess

Get running processes for a given remote machine:

```csharp
Get-NetProcess -ComputerName dc01 -RemoteUserName offense\administrator -RemotePassword 123456 | ft
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQItdzMOGqHK9h9YNfz%2F-LQIu2VkugNWZBJbC43M%2FScreenshot%20from%202018-11-02%2010-11-17.png?alt=media\&token=8953902b-db5a-443a-8a38-33851553efb5)

### Invoke-MapDomainTrust

Enumerate and map all domain trusts:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLzleS4g0dxZT8fVUSL%2F-LLzjb4pR0R0QZFWnSEL%2Fpowerview-all-domain-trusts.png?alt=media\&token=01bff020-d0a6-4619-be8c-a21c13d81f18)

### Invoke-ShareFinder

Enumerate shares on a given PC - could be easily combines with other scripts to enumerate all machines in the domain:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLzleS4g0dxZT8fVUSL%2F-LLzkAMHWApp9EX94TzE%2Fpowerview-enumerate-shares.png?alt=media\&token=42f07d44-8096-438f-8f95-ceaae6258883)

### Invoke-UserHunter

Find machines on a domain or users on a given machine that are logged on:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLzleS4g0dxZT8fVUSL%2F-LLzlbfMrGPxEcvbX6E1%2Fpowerview-invoke-user-hunter.png?alt=media\&token=edec5965-02af-43f1-9303-4e0de0bb58c7)

### References

{% embed url="https://github.com/PowerShellMafia/PowerSploit" %}

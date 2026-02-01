# Windows Event IDs and Others for Situational Awareness

Below is a living list of Windows event IDs and miscellaneous PowerShell snippets that may be useful for situational awareness once you are on a box.

## Lock / screensaver

Workstation was locked

{% code title="Get-WinEvent - Workstation locked" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='security'; Id='4800' }
```
{% endcode %}

Workstation was unlocked

{% code title="Get-WinEvent - Workstation unlocked" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='security'; Id='4801' }
```
{% endcode %}

Screensaver invoked

{% code title="Get-WinEvent - Screensaver invoked" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='security'; Id='4802' }
```
{% endcode %}

Screensaver dismissed

{% code title="Get-WinEvent - Screensaver dismissed" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='security'; Id='4803' }
```
{% endcode %}

## System ON / OFF

Windows is starting up

{% code title="Get-WinEvent - Windows starting up" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='security'; Id='4608' }
```
{% endcode %}

System uptime

{% code title="Get-WinEvent - System uptime" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='system'; Id='6013' }
```
{% endcode %}

Windows is shutting down

{% code title="Get-WinEvent - Windows shutting down" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='security'; Id='4609' }
```
{% endcode %}

System has been shut down

{% code title="Get-WinEvent - System has been shut down" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='system'; Id='1074' }
```
{% endcode %}

## System sleep / awake

System entering sleep mode

{% code title="Get-WinEvent - Sleep mode" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='system'; Id=42 }
```
{% endcode %}

System returning from sleep

{% code title="Get-WinEvent - Return from sleep" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='system'; Id='1'; ProviderName = "Microsoft-Windows-Power-Troubleshooter" }
```
{% endcode %}

## Logons

Successful logons

{% code title="Get-WinEvent - Successful logons" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='Security'; Id='4624' }
```
{% endcode %}

Logons with explicit credentials

{% code title="Get-WinEvent - Explicit credential logons" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='Security'; Id='4648' }
```
{% endcode %}

Account logoffs

{% code title="Get-WinEvent - Account logoffs" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='security'; Id='4634' }
```
{% endcode %}

## Access

Outbound RDP (client initiated)

{% code title="Get-WinEvent - Outbound RDP (client)" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-TerminalServices-RDPClient/Operational'; id='1024' } | select timecreated, message | ft -AutoSize -Wrap
```
{% endcode %}

Inbound RDP (session creation)

{% code title="Get-WinEvent - Inbound RDP (LocalSessionManager)" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-TerminalServices-LocalSessionManager/Operational'; id='21' } | select timecreated, message | ft -AutoSize -Wrap
```
{% endcode %}

Inbound RDP (RdpCoreTS)

{% code title="Get-WinEvent - Inbound RDP (RdpCoreTS)" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-RemoteDesktopServices-RdpCoreTS/Operational'; id=131 } | select timecreated, message | ft -AutoSize -Wrap
```
{% endcode %}

RemoteConnectionManager (RDP reconnections / connection attempts)

{% code title="Get-WinEvent - RemoteConnectionManager (1149)" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-TerminalServices-RemoteConnectionManager/Operational'; id='1149' } | ft -AutoSize -Wrap
```
{% endcode %}

Outbound WinRM

{% code title="Get-WinEvent - WinRM outbound (6)" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-WinRM/Operational'; id=6 }
```
{% endcode %}

{% code title="Get-WinEvent - WinRM outbound (80)" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-WinRM/Operational'; id=80 }
```
{% endcode %}

Inbound WinRM

{% code title="Get-WinEvent - WinRM inbound (91)" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-WinRM/Operational'; id=91 }
```
{% endcode %}

WMI activity related to Terminal Services

{% code title="Get-WinEvent - WMI Activity (5857) filter for terminal service providers" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='Microsoft-Windows-WMI-Activity/Operational'; id=5857 } | ? {$_.message -match 'Win32\_WIN32\_TERMINALSERVICE\_Prov|CIMWin32'}
```
{% endcode %}

### Inbound Network and Interactive Logons (custom parsing)

This collects 4624 events over a recent timeframe and filters by logon type and source.

{% code title="PowerShell - Inbound Network & Interactive Logons (parsed)" %}
```powershell
$events = New-Object System.Collections.ArrayList

Get-WinEvent -FilterHashtable @{ LogName='Security'; id=(4624); starttime=(get-date).AddMinutes(-60*24*2) } | % {

    $event = New-Object psobject
    $subjectUser = $_.properties[2].value + "\" + $_.properties[1].value
    $targetUser = $_.properties[6].value + "\" + $_.properties[5].value
    $logonType = $_.properties[8].value
    $subjectComputer = $_.properties[18].value

    if ($logonType -in 3,7,8,9,10,11 -and $subjectComputer -notmatch "::1|-|^127.0.0.1") {

        switch ($logonType) {
            3 { $logonType = "Network" }
            7 { $logonType = "Screen Unlock" }
            8 { $logonType = "Network Cleartext" }
            9 { $logonType = "New Credentials" }
            10 { $logonType = "Remote Interactive" }
            11 { $logonType = "Cached Interactive" }
        }

        $event | Add-Member "Time" $_.TimeCreated
        $event | Add-Member "Subject" $subjectUser
        $event | Add-Member "LogonFrom" $subjectComputer
        $event | Add-Member "LoggedAs" $targetUser
        $event | Add-Member "Type" $logonType
        $events.Add($event) | out-null
    }

}

$events
```
{% endcode %}

### Outbound Network Logons (custom parsing)

This parses 4648 events and filters out localhost targets.

{% code title="PowerShell - Outbound Network Logons (parsed)" %}
```powershell
$events = New-Object System.Collections.ArrayList

Get-WinEvent -FilterHashtable @{ LogName='Security'; id=(4648); starttime=(get-date).AddMinutes(-60*24*2) } | % {

    $event = New-Object psobject
    $subjecUser = $_.Properties[2].Value + "\" + $_.Properties[1].Value
    $targetUser = $_.Properties[6].Value + "\" + $_.Properties[5].Value
    $targetInfo = $_.Properties[9].Value
    $process = $_.Properties[11].Value

    $event | Add-Member "Time" $_.timecreated
    $event | Add-Member "SubjectUser" $subjecUser
    $event | Add-Member "TargetUser" $targetUser
    $event | Add-Member "Target" $targetInfo
    $event | Add-Member "Process" $process

    if ($targetInfo -notmatch 'localhost') {
        $events.add($event) | out-null
    }

}

$events
```
{% endcode %}

## Activity

Attempt to install a service

{% code title="Get-WinEvent - Service installation attempted" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='Security'; Id='4697' }
```
{% endcode %}

Scheduled task created

{% code title="Get-WinEvent - Scheduled task created" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='security'; Id='4698' }
```
{% endcode %}

Scheduled task updated

{% code title="Get-WinEvent - Scheduled task updated" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='security'; Id='4702' }
```
{% endcode %}

Sysinternals usage?

{% code title="Registry - Sysinternals usage" %}
```powershell
Get-ItemProperty 'HKCU:\SOFTWARE\Sysinternals\*' | select PSChildName, EulaAccepted
```
{% endcode %}

## Security

LSASS started as a protected process

{% code title="Get-WinEvent - LSASS protected process started" %}
```powershell
Get-WinEvent -FilterHashtable @{ LogName='system'; Id='12' ; ProviderName='Microsoft-Windows-Wininit' }
```
{% endcode %}

***

(End of content)

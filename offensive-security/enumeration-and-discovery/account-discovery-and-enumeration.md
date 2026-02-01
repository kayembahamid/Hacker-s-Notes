# Account Discovery & Enumeration

## Account Discovery & Enumeration

### Execution

Let's run some of the popular enumeration commands on the victim system:

{% code title="attacker\@victim" %}
```csharp
net user
net user administrator
whoami /user
whoami /all
...
```
{% endcode %}

### Hunting and Observations

Having command line logging can help in identifying a cluster of enumeration commands executed in a relatively short span of time on a compromised host .

For this lab, I exported 8600+ command lines from various processes and wrote a dirty powershell script that ingests those command lines and inspects them for a couple of classic windows enumeration commands that are executed in the span of 2 minutes and spits them out:

{% code title="hunt.ps1" %}
```csharp
function hunt() {
    [CmdletBinding()]Param()
    $commandlines = Import-Csv C:\Users\mantvydas\Downloads\cmd-test.csv
    $watch = 'whoami|net1 user|hostname|netstat|net localgroup|cmd /c'
    $matchedCommandlines = $commandlines| where-object {  $_."event_data.CommandLine" -match $watch}

    $matchedCommandlines| foreach-Object {
        [datetime]$eventTime = $_."@timestamp"
        [datetime]$low = $eventTime.AddSeconds(-60)
        [datetime]$high = $eventTime.AddSeconds(60)
        $clusteredCommandlines = $commandlines | Where-Object { [datetime]$_."@timestamp" -ge $low -and [datetime]$_."@timestamp" -le $high -and  $_."event_data.CommandLine" -match $watch}
        
        if ($clusteredCommandlines.length -ge 4) {
            Write-Verbose "Possible enumeration around time: $low - $high ($eventTime)"
            $clusteredCommandlines
        }
    }
}
```
{% endcode %}

Invoking the script to start the hunt:

```csharp
. \hunt.ps1; hunt -verbose
```

Below are some of the findings which may warrant further investigation of the suspect host:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKcVgLiliP-vL499Mrc%2F-LKcZN_Pwa8TFXRTZdVU%2Fenumeration-hunt-5.png?alt=media\&token=39a34c09-098d-44ee-8adc-8f710ef7ce88)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKcVgLiliP-vL499Mrc%2F-LKcZN_EEBRT9kw8HtCw%2Fenumeration-hunt-4.png?alt=media\&token=315566f4-cec3-40bb-8915-d54667399fd0)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKcVgLiliP-vL499Mrc%2F-LKcZN_9tJ205c4_EGVW%2Fenumeration-hunt-3.png?alt=media\&token=f555adcb-5007-472f-bb7f-ce4c9978bc87)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKcVgLiliP-vL499Mrc%2F-LKcZN_7TmfdOZZWk9TZ%2Fenumeration-hunt-2.png?alt=media\&token=fa72f210-9407-4845-ac17-04b0cb20fb9e)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKcVgLiliP-vL499Mrc%2F-LKcZN_6Ytp3LQcyb1Xu%2Fenumeration-hunt-1.png?alt=media\&token=9e42b0e3-7a0d-4056-96f4-7f17766ce31c)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1087" %}

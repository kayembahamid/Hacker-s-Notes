# Detecting Sysmon on the Victim Host

## Detecting Sysmon on the Victim Host

### Processes

{% code title="attacker\@victim" %}
```csharp
PS C:\> Get-Process | Where-Object { $_.ProcessName -eq "Sysmon" }
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOOeCqOiPWbiOLxmV1x%2F-LOOgheZa0IieeWu1os4%2FScreenshot%20from%202018-10-09%2017-39-28.png?alt=media\&token=9ad846c1-38d6-41eb-8974-e9ee1aa7cf85)

{% hint style="warning" %}
Note: process name can be changed during installation
{% endhint %}

### Services

{% code title="attacker\@victim" %}
```csharp
Get-CimInstance win32_service -Filter "Description = 'System Monitor service'"
# or
Get-Service | where-object {$_.DisplayName -like "*sysm*"}
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOOeCqOiPWbiOLxmV1x%2F-LOOighsvbnGtFYmGnBi%2FScreenshot%20from%202018-10-09%2017-48-11.png?alt=media\&token=5d469b47-efae-4ceb-b29d-7aff229d0b06)

{% hint style="warning" %}
Note: display names and descriptions can be changed
{% endhint %}

### Windows Events

{% code title="attacker\@victim" %}
```csharp
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\WINEVT\Channels\Microsoft-Windows-Sysmon/Operational
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOOeCqOiPWbiOLxmV1x%2F-LOOjHPWGDSHkbU5Pwvj%2FScreenshot%20from%202018-10-09%2017-50-47.png?alt=media\&token=780e1598-1d00-4869-ac06-c9542ff99b64)

### Filters

{% code title="attacker\@victim" %}
```
PS C:\> fltMC.exe
```
{% endcode %}

Note how even though you can change the sysmon service and driver names, the sysmon altitude is always the same - `385201`

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOOjIxI9l80Uzkydwsw%2F-LOOjlnxkHRyqT6nLFzU%2FScreenshot%20from%202018-10-09%2017-51-45.png?alt=media\&token=ada64200-5603-469f-a82a-d2dbfaf5d89b)

### Sysmon Tools + Accepted Eula

{% code title="attacker\@victim" %}
```
ls HKCU:\Software\Sysinternals
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOOjIxI9l80Uzkydwsw%2F-LOOkcleCCvSzv-kmWDX%2FScreenshot%20from%202018-10-09%2017-56-33.png?alt=media\&token=374124e0-4874-4bbc-93a5-6408614f8918)

### Sysmon -c

Once symon executable is found, the config file can be checked like so:

```
sysmon -c
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOOufC4FbrKXa1-M0o2%2F-LOOvPUPPG3UZl4cGM6z%2FScreenshot%20from%202018-10-09%2018-43-39.png?alt=media\&token=74191a9f-680c-4347-b2e3-2c4e852ba394)

### Config File on the Disk

If you are lucky enough, you may be able to find the config file itself on the disk by using native windows utility findstr:

{% code title="attcker\@victim" %}
```csharp
findstr /si '<ProcessCreate onmatch="exclude">' C:\tools\*
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOOufC4FbrKXa1-M0o2%2F-LOOyZ8B1S66xdWiTWar%2FScreenshot%20from%202018-10-09%2018-57-32.png?alt=media\&token=5f6ea661-a849-41a1-bf88-af6ea9d190f1)

### Get-SysmonConfiguration

A powershell tool by @mattifestation that extracts sysmon rules from the registry:

{% code title="attacker\@victim" %}
```csharp
PS C:\tools> (Get-SysmonConfiguration).Rules
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOOnUMn6LrY1vGhaPIo%2F-LOOoAGwjhaEnNsXljLG%2FScreenshot%20from%202018-10-09%2018-12-09.png?alt=media\&token=4294a30e-2154-4439-98f2-990876d37089)

As an example, looking a bit deeper into the `ProcessCreate` rules:

{% code title="attacker\@victim" %}
```csharp
(Get-SysmonConfiguration).Rules[0].Rules
```
{% endcode %}

We can see the rules almost as they were presented in the sysmon configuration XML file:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOOnUMn6LrY1vGhaPIo%2F-LOOoWdDkFIxPk31gLk9%2FScreenshot%20from%202018-10-09%2018-13-37.png?alt=media\&token=1ebf5dfc-1fca-4cc7-b7c8-54a1eb1e439e)

A snippet from the actual sysmonconfig-export.xml file:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOOnUMn6LrY1vGhaPIo%2F-LOOot8jxGW0TSXz4g3b%2FScreenshot%20from%202018-10-09%2018-14-57.png?alt=media\&token=a959dde0-5f7a-439c-a214-7aed6a6e03c1)

### Bypassing Sysmon

Since Get-SysmonConfiguration gives you the ability to see the rules sysmon is monitoring on, you can play around those.

Another way to bypass the sysmon altogether is explored here:

### References

{% embed url="https://www.darkoperator.com/blog/2018/10/5/operating-offensively-against-sysmon" %}

{% embed url="https://github.com/mattifestation/PSSysmonTools/blob/master/PSSysmonTools/Code/SysmonRuleParser.ps1" %}

{% embed url="https://docs.microsoft.com/en-us/windows-hardware/drivers/ifs/allocated-altitudes" %}

{% embed url="https://github.com/GhostPack/Seatbelt" %}

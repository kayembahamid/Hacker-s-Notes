# Timestomping

## Timestomping

### Execution

Checking original timestamps of the `nc.exe`:

```csharp
.\timestomp.exe .\nc.exe -v
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LK6UyTRwVzf7Wl-0Z0N%2F-LK6WG-TCbQr8Hbwwovw%2Ftimestomp-original.png?alt=media\&token=30515153-9ded-422f-b80f-2555d0c4ade6)

Forging the file creation date:

```csharp
.\timestomp.exe .\nc.exe -c "Monday 7/25/2005 5:15:55 AM"
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LK6UyTRwVzf7Wl-0Z0N%2F-LK6WG-YHi2hXvPUJ5pr%2Ftimestomp-forged.png?alt=media\&token=4a845159-de50-4c1d-b368-f9c9aa4d3eaa)

Checking the `$MFT` for changes - first of, dumping the `$MFT`:

```csharp
.\RawCopy64.exe /FileNamePath:C:\$MFT /OutputName:c:\experiments\mft.dat
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LK6UyTRwVzf7Wl-0Z0N%2F-LK6WG-abYuMn-zIKjK2%2Ftimestomp-dump-parse-mft.png?alt=media\&token=2b28ab27-6d37-4ab5-9759-da5118363c7d)

Let's find the `nc.exe` record and check its timestamps:

```csharp
Import-Csv .\mft.csv -Delimiter "`t" | Where-Object {$_.Filename -eq "nc.exe"}
```

Note how `fnCreateTime` did not get updated:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LK6UyTRwVzf7Wl-0Z0N%2F-LK6WG-cVY_oG0pvC3Oy%2Ftimestomp-mft-timestamps.png?alt=media\&token=fc847a0a-b3df-463d-a515-fa6c0fbaea7d)

For this reason, it is always a good idea to check both `$STANDARD_INFO` and `$FILE_NAME` times during the investigation to have a better chance at detecting timestomping.

Note that if we moved the nc.exe file to any other folder on the system and re-parsed the $MFT again, the `fnCreateTime` timestamp would inherit the timestamp from `siCreateTime`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LK6hZ2h2bUqPo0K0T0k%2F-LK6jKRdTtf1x38GbFCB%2Ftimestomp-moved.png?alt=media\&token=d276db1a-476a-4d36-917d-ec0238aa8e4e)

### References

{% embed url="https://digital-forensics.sans.org/blog/2010/11/02/digital-forensics-time-stamp-manipulation" %}

{% embed url="https://attack.mitre.org/wiki/Technique/T1099" %}

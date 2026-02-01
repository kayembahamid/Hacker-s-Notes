# Word Library Add-Ins

## Word Library Add-Ins

It' possible to persist in the userland by abusing word library add-ins by putting your malicious DLL into a Word's trusted location. Once the DLL is there, the Word will load it next time it is run.

### Execution

Get Word's trusted locations where library add-ins can be dropped:

{% tabs %}
{% tab title="attacker\@target" %}
```csharp
 Get-ChildItem "hkcu:\Software\Microsoft\Office\16.0\Word\Security\Trusted Locations"
```
{% endtab %}
{% endtabs %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LhyoRa1tCT-gCs5l2zM%2F-LhytB9tWdIEMv5lUNLp%2FAnnotation%202019-06-22%20121402.png?alt=media\&token=6ea87100-ad57-4618-8c19-d6abb7e5f598)

Those trusted locations are actually defined in Word's Security Center if you have access to the GUI:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LhyoRa1tCT-gCs5l2zM%2F-LhytH-t9VeQiWWp2mBG%2FAnnotation%202019-06-22%20121426.png?alt=media\&token=9655e4b6-d89c-4f9f-acff-bc7e3ec7167e)

Let's create a simple DLL that will launch a notepad.exe once the DLL addin is loaded:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LhzOOPW175IQ2cX3v-V%2F-LhzOmsSztaeNIRcSWpj%2FAnnotation%202019-06-22%20143558.png?alt=media\&token=92887b82-7b21-413f-83e3-49f2121ea469)

Compile the DLL and copy it over to `Startup` folder and rename it to `evilm64.wll`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LhyoRa1tCT-gCs5l2zM%2F-LhytYW4_8OpAlmfrAqk%2FAnnotation%202019-06-22%20121537.png?alt=media\&token=07584061-ac7a-441c-b272-c662b96fd494)

```
mv .\evilm64.dll .\evilm64.wll
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LhzOOPW175IQ2cX3v-V%2F-LhzPgNrjVRZH2fMoXh-%2FAnnotation%202019-06-22%20144024.png?alt=media\&token=af8514fb-4a82-40d9-9edf-bc33bc2096f8)

Next time the victim opens up Word, `evilm64.wll` will be loaded and executed:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LhzOOPW175IQ2cX3v-V%2F-LhzOSZOxRWdwxLn0rPY%2FAnnotation%202019-06-22%20143432.png?alt=media\&token=8acabd54-3cb2-4ec5-ab4c-5e937ca09501)

Interesting to note that Process Explorer does not see the evilm64.wll loaded in any of the currently running processes:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LhzOOPW175IQ2cX3v-V%2F-LhzPwCMi_9dZ3vP9MDk%2FAnnotation%202019-06-22%20144128.png?alt=media\&token=1e15ea4e-0701-47c6-937b-b88ed0952740)

...although we can definitely see that the add-in is now recognized by Word:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LhzOOPW175IQ2cX3v-V%2F-LhzQC88OEHPwAWNcubW%2FAnnotation%202019-06-22%20144219.png?alt=media\&token=7d0be23a-d053-49fe-a19b-781604adc435)

{% hint style="info" %}
This technique did not work for me on Office 365 version, but worked on Office Professional. Not sure if there's a bug in the 365 version or it's just a limitation of that version.
{% endhint %}

### References

{% embed url="https://www.mdsec.co.uk/2019/05/persistence-the-continued-or-prolonged-existence-of-something-part-1-microsoft-office/" %}


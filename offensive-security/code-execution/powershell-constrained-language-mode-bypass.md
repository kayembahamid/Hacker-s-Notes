# Powershell Constrained Language Mode Bypass

Constrained Language Mode in short locks down the nice features of Powershell usually required for complex attacks to be carried out.

### Powershell Inside Powershell

For fun - creating another powershell instance inside powershell without actually spawning a new `powershell.exe` process:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LM933gU6QIBeSF4St1D%2F-LM93J31GDKWmZl5XU1Y%2Fps-invoke.gif?alt=media\&token=c94043bc-822f-4d6d-8ed0-5188e53f89a9)

### Constrained Language Mode

Enabling constrained language mode, that does not allow powershell execute complex attacks (i.e. mimikatz):

```csharp
[Environment]::SetEnvironmentVariable(‘__PSLockdownPolicy‘, ‘4’, ‘Machine‘)
```

Checking constrained language mode is enabled:

```csharp
PS C:\Users\mantvydas> $ExecutionContext.SessionState.LanguageMode
ConstrainedLanguage
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LM9PSZpLGwJbcXVvYHE%2F-LM9PVvArwDX8-xSuyFS%2Fps-constrained.png?alt=media\&token=34ce6025-f3fb-42b0-8c02-81b0ef4f2018)

With `ConstrainedLanguage`, trying to download a file from remote machine, we get `Access Denied`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LM9QXG0J_8rV03UwQLG%2F-LM9Qd-R1KaTueujpj9C%2Fps-constrained-download-denied.png?alt=media\&token=6f8bd579-6acc-496c-8f50-9d8814c6f44b)

However, if you have access to the system and enough privileges to change environment variables, the lock can be lifted by removing the variable `__PSLockdownPolicy` and re-spawning another powershell instance.

#### Powershell Downgrade

If you have the ability to downgrade to Powershell 2.0, this can allow you to bypass the `ConstrainedLanguage`mode. Note how `$ExecutionContext.SessionState.LanguageMode` keeps returning `ConstrainedLangue` in powershell instances that were not launched with `-version Powershell 2` until it does not:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LM9ifDRPS0RB9NvTGd5%2F-LM9juaZ8VJkPyFsl1AJ%2Fps-downgrade.png?alt=media\&token=1f05b4cc-03a8-43d2-8118-5a4163e877df)

### System32 Bypass

[Carrie Roberts](https://twitter.com/OrOneEqualsOne) discovered and wrote in her post [https://www.blackhillsinfosec.com/constrained-language-mode-bypass-when-pslockdownpolicy-is-used/](https://www.blackhillsinfosec.com/constrained-language-mode-bypass-when-pslockdownpolicy-is-used/) that there's another way to bypass the contrained language mode and it's super easy - the path from where your script is being executed, needs to contain the string `system32`, meaning even if you rename the script to `system32.ps1`, it should work, so let's try it and confirm it works:

```
PS>.\test.ps1; mv .\test.ps1 system32.ps1; .\system32.ps1
ConstrainedLanguage
FullLanguage

PS>cat .\system32.ps1
$ExecutionContext.SessionState.LanguageMode
```

### References

{% embed url="https://blogs.msdn.microsoft.com/powershell/2017/11/02/powershell-constrained-language-mode/" %}

{% embed url="https://www.blackhillsinfosec.com/powershell-without-powershell-how-to-bypass-application-whitelisting-environment-restrictions-av/" %}

{% embed url="https://adsecurity.org/?p=2604" %}

{% embed url="https://pentestn00b.wordpress.com/2017/03/20/simple-bypass-for-powershell-constrained-language-mode/" %}

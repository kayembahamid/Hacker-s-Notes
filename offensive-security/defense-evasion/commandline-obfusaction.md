# Commandline Obfusaction

## Commandline Obfusaction

This lab is based on the research done by Daniel Bohannon from FireEye.

### Environment variables

```csharp
C:\Users\mantvydas>set a=/c & set b=calc
C:\Users\mantvydas>cmd %a% %b%
```

Note though that the commandline logging (dynamic detection) still works as the commandline needs to be expanded before it can get executed, but static detection could be bypassed:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNUPDVYAH_3gMRgypxx%2F-LNUUUwUnlMVKTxm3C7G%2Fenvironment-variables.png?alt=media\&token=52da7222-cf6d-498d-bf37-2c6b26f81aee)

### Double quotes

```csharp
C:\Users\mantvydas>c""m"d"
```

Note how double quotes can actually make both static and dynamic detection a bit more difficult:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNUPDVYAH_3gMRgypxx%2F-LNUVyrUuYN0Tet36BV5%2Fdouble-quotes.png?alt=media\&token=c2eaea81-3de3-4eaf-950c-3c77e3107846)

### Carets

```csharp
C:\Users\mantvydas>n^e^t u^s^er
```

Commandline logging, same as with using environment variables, is not affected, however static detection could be affected:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNUPDVYAH_3gMRgypxx%2F-LNUWT0eIK8eFqY_tR4C%2Fcarets.png?alt=media\&token=32962232-f880-4bc4-a792-d26773d9be69)

### Garbage delimiters

A very interesting technique. Let's look at this first without garbage delimiters:

```csharp
PS C:\Users\mantvydas> cmd /c "set x=calc & echo %x% | cmd"
```

The above sets en environment variable x to `calc` and then prints it and pipes it to the standard input of the cmd:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNUYB6m3eEQpdOmHIID%2F-LNU_pK9GJpwl0bxs_yp%2Fgarbage1.png?alt=media\&token=467289e5-5369-4914-9b28-52353c14d805)

Introducing garbage delimiters `@` into the equation:

```csharp
PS C:\Users\mantvydas> cmd /c "set x=c@alc & echo %x:@=% | cmd"
```

The above does the same as the earlier example, except that it introduces more filth into the command (`c@lc`). You can see from the below screenshot that Windows does not recognize such a command `c@lc`, but the second attempt when the `%x:@=%` removes the extraneous `@` symbol from the string, gets executed successfully:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNUYB6m3eEQpdOmHIID%2F-LNUavyT85jw6C1wAWNQ%2Fgarbage2.png?alt=media\&token=3ca5a759-6870-4a7f-9f58-d680a91c8941)

If it is confusing, the below should help clear it up:

```
PS C:\Users\mantvydas> cmd /c "set x=c@alc & echo %x:@=mantvydas% | cmd"
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNUYB6m3eEQpdOmHIID%2F-LNUcxvu_9u7wS_HTFn_%2Fgarbage3.png?alt=media\&token=0f6a9dc2-63da-42c1-959c-db391f50bafd)

In the above, the value `mantvydas` got inserted in the `c@lc` in place of @, suggesting that `%x:@=%` (`:@=` to be precise) is just a string replacement capability in the cmd.exe utility.

With this knowledge, the original obfuscated command

```csharp
PS C:\Users\mantvydas> cmd /c "set x=c@alc & echo %x:@=% | cmd"
```

reads: replace the @ symbol with text that goes after the `=` sign, which is empty in this case, which effectively means - remove @ from the value stored in the variable x.

### Substring

Cmd.exe also has a substring capability. See below:

```csharp
# this will take the C character from %programdata% and will launch the cmd prompt
%programdata:~0,1%md
```

Note that this is only good for bypassing static detection:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNUyGHJEf6uaoz8ah4o%2F-LNUzASBPAF5ej-J3-jw%2Fsubstring1.png?alt=media\&token=b71f0dad-6686-466a-b41d-d4c7777362fd)

### Batch FOR, DELIMS + TOKENS

We can use a builtin batch looping to extract the Powershell string from environment variables in order to launch it and bypass static detection that looks for a string "powershell" in program invocations:

{% code title="@cmd" %}
```csharp
set pSM 
PSModulePath=C:\Users\mantvydas\Documents\WindowsPowerShell\Modules;....
```
{% endcode %}

Note how the `WindowsPowerShell` string is present in the `PSModule` environment variable - this mean we can extract it like so:

```csharp
FOR /F "tokens=7 delims=s\" %g IN ('set^|findstr PSM') do %g
```

What the above command does:

1. Executes `set^|findstr PSM` to get the PSModulePath variable value
2. Splits the string using delimiters `s` & `\`
3. Prints out the 7th token, which happens to be the `PowerShell`
4. Which effectively launches PowerShell

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNVg9lPA4TbTrcXOd6g%2F-LNVF2gcOuSy-1xNdFlO%2Fbatch-powershell.png?alt=media\&token=4ea441a5-0f72-4f1d-a3ae-5ff3e5cb46a4)

### Comma, semicolon

This may be used for both static and dynamic detection bypasses:

```csharp
C:\Users\mantvydas>cmd,/c;hostname
PC-MANTVYDAS
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNVhqhFI0u8E2FyV_BT%2F-LNVi6T_0TagaAVmsRr7%2Fcomasemicoma.png?alt=media\&token=1a0a691f-fbd8-4659-9d3f-20ef4004165f)

### FORCoding

What happens below is essentially there is a loop that goes through the list of indexes (0 1 2 3 2 6 2 4 5 6 0 7) which are used to point to characters in the variable `unique` which acts like an alphabet. This allows for the FOR loop to cycle through the index, pick out characters from the alphabet pointed to by the index and concatenate them into a final string that eventually gets called with `CALL %final%` when the loop reaches the index 1337.

```csharp
PS C:\Users\mantvydas> cmd /V /C "set unique=nets /ao&&FOR %A IN (0 1 2 3 2 6 2 4 5 6 0 7 1337) DO set final=!final!!uni
que:~%A,1!&& IF %A==1337 CALL %final:~-12%"
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNWJPMKuW3zmn1ijWhy%2F-LNWDL-Y3-e_IBtGlcxn%2Fforcoding.png?alt=media\&token=861a4f3a-7af1-429d-b399-a6aa0b8b9d74)

In verbose python this could look something like this:

{% code title="forcoding.py" %}
```python
import os

dictionary = "nets -ao"
indexes = [0, 1, 2, 3, 2, 6, 2, 4, 5, 6, 0, 7, 1337]
final = ""

for index in indexes:
    if index == 1337:        
        break
    final += dictionary[index]
os.system(final)
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNWKqsGYnB0XYWlP_y9%2F-LNWKnIIv3wxsgb_B5Sw%2Fforcoding-python.png?alt=media\&token=cadadb79-6ffb-4503-b78d-a6d715a9633d)

### References

{% embed url="https://youtu.be/mej5L9PE1fs" %}

{% embed url="https://www.fireeye.com/blog/threat-research/2018/03/dosfuscation-exploring-obfuscation-and-detection-techniques.html" %}

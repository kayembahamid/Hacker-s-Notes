# Executing C# Assemblies from Jscript and wscript with DotNetToJscript

## Executing C# Assemblies from Jscript and wscript with DotNetToJscript

It's possible to load in to memory and execute C# compiled binaries from within javascript and vbscript by using a technique called [DotNetToJscript](https://github.com/tyranid/DotNetToJScript) by James Forshaw.

Since [SharpShooter](https://github.com/mdsecactivebreach/SharpShooter), [CactusTorch](https://github.com/mdsecactivebreach/CACTUSTORCH) and a couple of other offensive security tools are leveraging DotNetToJscript to execute the payloads in memory using C#, in this quick lab I wanted to simply use the DotNetToJscipt framework just to get a feel of the process and see if there are any easy to spot artefacts this technique leaves behind on the target system that could help defenders catch the attackers.

### Compilation

1. Download [DotNetToJscript](https://github.com/tyranid/DotNetToJScript)
2. Compile it (review the code before you do). It will spit out two binaries:
   1. DotNetToJscript.exe - responsible for bootrstrapping C# binaries (supplied as input) and converting them to JavaScript or VBScript
   2. ExampleAssembly.dll - the C# assembly that will be given to DotNetToJscript.exe. In default project configuration, the assembly just pops a message box with the text "test"
3. Execute DotNetToJscript.exe and supply it with the ExampleAssembly.dll, specify the output file and the output type:

```csharp
\\VBOXSVR\Experiments\DotNetToJScript\DotNetToJScript\bin\Debug\DotNetToJScript.exe \\VBOXSVR\Experiments\DotNetToJScript\ExampleAssembly\bin\Debug\ExampleAssembly.dll -l vbscript -o \\VBOXSVR\Experiments\DotNetToJScript\DotNetToJScript\test.vbs
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfF308uWWbIwntr7Dc6%2F-LfF8b4CX3cRkoC0216u%2FAnnotation%202019-05-19%20135204.png?alt=media\&token=d7f1f17a-b0df-481c-a2da-51e6c9b521ae)

We got a test.vbs created and if we look inside it, we can see that at a high level:

* the C# binary is now present as a base64 encoded data blob
* the data blobob will be deserialized and invoked using `DynamicInvoke`
* which will create a new instance of the `TestClass`
* which will kick off the `MessageBox` as defined in the `TestClass` constructor

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfF308uWWbIwntr7Dc6%2F-LfFCDP0SUR1HEa5FF4M%2FAnnotation%202019-05-19%20140645.png?alt=media\&token=60603ac0-6221-49b3-9b90-f4b9680d3fb9)

```javascript
entry_class = "TestClass"

Dim fmt, al, d, o
Set fmt = CreateObject("System.Runtime.Serialization.Formatters.Binary.BinaryFormatter")
Set al = CreateObject("System.Collections.ArrayList")
al.Add Empty

Set d = fmt.Deserialize_2(Base64ToStream(s))
Set o = d.DynamicInvoke(al.ToArray()).CreateInstance(entry_class)
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfFMU1LSdTVPTnoxdbi%2F-LfFMm5uRXJfayDQBLLc%2FAnnotation%202019-05-19%20145407.png?alt=media\&token=d14d7d83-09c8-4cbf-9126-18f388d78623)

### Execution & Observation

Let's now run the test.vbs - it pops the message box as expected:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfF308uWWbIwntr7Dc6%2F-LfFA5kxcPtDluiy_-ZI%2FAnnotation%202019-05-19%20135844.png?alt=media\&token=ffbe622a-7a47-4584-bb0c-46457f24340e)

Looking at the loaded modules of the wscript.exe, we can see a number of .NET assemblies in the process memory, which makes sense if you think about it:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfF308uWWbIwntr7Dc6%2F-LfFDlO3zDmMKl1bIg-j%2FAnnotation%202019-05-19%20141447.png?alt=media\&token=30b775e1-5efc-493a-adee-5ad68e504ffd)

Now, what happens if we try executing a simple vbscript that pops a message box and inspect the loaded modules of the wscript.exe again? Bingo, no .NET assemlies loaded:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LfF308uWWbIwntr7Dc6%2F-LfFFr593Eee1XHzbCum%2FAnnotation%202019-05-19%20142153.png?alt=media\&token=9693976e-c892-4e1a-b58e-ab04b0131e43)

Looking from the defensive point of view, it may be worth checking the environment for machines executing wscript (or jscript or cscript) which load .NET assemblies in their memory space and make sure the activity is benign.

Since .js or .vbs may be one of the payload delivery methods used in phishing using [file smuggling ](file-smuggling-with-html-and-javascript.md)through browsers, you may also want to check your environment for wscript (or cscript or jscript) launching files scripts from the user's download folder which is the default browser download location.

Know of any other hlepful artefacts? Let me know.

### References

{% embed url="https://github.com/tyranid/DotNetToJScript" %}

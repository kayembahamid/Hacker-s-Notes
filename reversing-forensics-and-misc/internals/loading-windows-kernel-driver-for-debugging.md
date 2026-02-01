# Loading Windows Kernel Driver for Debugging

## Loading Windows Kernel Driver for Debugging

### Loading a Driver with OSR Driver Loader

On the system where you want to load your driver (debugee), from an elevated command prompt, disable the driver integrity checks so that we can load our unsigned drivers onto Windows 10:

```
bcdedit /set nointegritychecks on; bcdedit /set testsigning on
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LuNfHkLVQ1z6lFM_d62%2F-LuNo3eU4ScoLrl3E-UV%2Fimage.png?alt=media\&token=a40ad637-4024-4ebb-877b-19d5ac734599)

Once you have rebooted the system, open up the [OSR Loader](https://www.osronline.com/article.cfm^article=157.htm) and load the driver as shown below:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LuNfHkLVQ1z6lFM_d62%2F-LuNxI6Uo6QEVmrA3FkX%2Floadkerneldriver.gif?alt=media\&token=a301260c-23ee-446a-8330-e67014d461c0)

Note that my driver name was `kmdfHelloDriver`. We can now confirm the driver loaded successfully by debugging the kernel:

```
0: kd> db kmdfHelloDriver
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LuNfHkLVQ1z6lFM_d62%2F-LuNyE1QfaoOH8vc06gf%2Fconfirmdriverloaded.gif?alt=media\&token=b2bcc05d-71a7-418b-91d6-0ac59f37147c)

Additionally, we can check it this way by showing some basic details about the loaded module:

```
0: kd> ln kmdfHelloDriver
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LuNyQV1knaDj0w6pe2L%2F-LuNyl3FSgAk1thv3YUG%2Fimage.png?alt=media\&token=15e1d951-84f5-4589-8aa1-66b70fabfba4)

If we check it via the service configuration manager, we also see that our driver is now loaded and running:

```
sc.exe query kmdfHelloDriver
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LuNyoGgoX4HO-H2lN2-%2F-LuNzGj5jhWsJrFucFec%2Fimage.png?alt=media\&token=14b68c79-03b5-44e9-a559-0b5d05c7c2b2)

### Loading a Driver via Command Prompt + WinDBG

The benefit of loading a kernel driver this way is that it does not rely on OSR Driver Loader or any other 3rd party tools and also is much more efficient.

{% hint style="info" %}
**Important**\
In order for this technique to work, the WinDBG debugger needs to be attached to the debugee.
{% endhint %}

#### Preparing Powershell Profile

On the debuggee, launch an elevated powershell console and do the following:

```
notepad $PROFILE.AllUsersAllHosts
```

in the powershell profile, add the following powershell function:

```csharp
function Install-Driver($name)
{
	$cleanName = $name -replace ".sys|.\\", ""

	sc.exe stop $cleanName
	sc.exe delete $cleanName

	cp $name c:\windows\system32\drivers\ -verbose -force
	sc.exe create $cleanName type= kernel start= demand error= normal binPath= c:\windows\System32\Drivers\$cleanName.sys DisplayName= $cleanName

	sc.exe start $cleanName
}
```

The above function `Install-Driver` takes one parameter `$name`, which signifies a driver name that we want to install.

The function `Install-Driver` will:

* Attempt to stop the service (unload the driver) if it's already running (no error checking)
* Attempt to delete the service (no error checking)
* Copy the driver from the current directory to c:\windows\system32\drivers
* Create a service for the driver
* Start the service (load the driver)

Below screenshot shows the two steps explained above:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-ME8wcIXb6rpBCpajByT%2F-ME8ysJKm5ognyndoCKQ%2Fimage.png?alt=media\&token=a7d38406-98b8-4e89-8cfe-34eec9e44a24)

{% hint style="info" %}
Once the powershell profile is saved, close the powershell console and open it again for the function `Install-Driver` to become usable.
{% endhint %}

#### Loading the Driver

Navigate to the folder that contains the .sys file of the driver you want to install, which in my case is `wdm-helloworld.sys` in Z:\wdm-helloworld\x64\Debug:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-ME8wcIXb6rpBCpajByT%2F-ME91YmZTD-AmfBmQOei%2Fimage.png?alt=media\&token=83457416-4f10-47be-8962-d664f43c012a)

Now, we can install the driver by simply invoking:

```csharp
Install-Driver wdm-helloworld.sys
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-ME8wcIXb6rpBCpajByT%2F-ME93SAgq7BOQYo468I3%2Fload-driver.gif?alt=media\&token=96250b43-d564-47f6-9cb4-2bac2aeb53d8)

#### Stepping through Source Code

If we have source code for the driver we want to debug, we can load its source code and step through it in WinDBG. Load the source code via the `Source > Open Source File` and re-load the driver again using `Install-Driver` function:

![Stepping through driver's C code](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-ME8wcIXb6rpBCpajByT%2F-ME9528M2ts5vjAeITq-%2Fdebugging-kernel-source-code.gif?alt=media\&token=b38a3aa4-bd84-4ac0-ba89-147d3b3db7c1)

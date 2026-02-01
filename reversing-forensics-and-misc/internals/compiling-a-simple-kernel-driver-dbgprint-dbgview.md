# Compiling a Simple Kernel Driver, DbgPrint, DbgView

## Compiling a Simple Kernel Driver, DbgPrint, DbgView

### Simple Windows Driver Framework (WDF) Kernel Driver

Select Kernel Mode Driver, Emtpy (KMDF) from templates:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-M12jvqwCP4zgHSwkDtu%2F-M12kHHK5ZM_7c2358C9%2Fimage.png?alt=media\&token=e28bff10-48c8-4039-92d8-a02345ba6aa8)

### Create a driver.c

Create a new `driver.c` file under `Source Files`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-M12jvqwCP4zgHSwkDtu%2F-M12kSVYTjYJAs5pMSed%2Fimage.png?alt=media\&token=fbed76fb-5835-4894-9fad-2b4ec10a2c99)

### Add Driver Code

{% code title="driver.c" %}
```c
#include <ntddk.h>
#include <wdf.h>

DRIVER_INITIALIZE DriverEntry;
EVT_WDF_DRIVER_DEVICE_ADD EvtDriverDeviceAdd;
EVT_WDF_DRIVER_UNLOAD UnloadDriver;

_Use_decl_annotations_
void UnloadDriver(IN WDFDRIVER driver)
{
    UNREFERENCED_PARAMETER(driver);
    DbgPrint("Driver unloaded");
}

NTSTATUS DriverEntry(_In_ PDRIVER_OBJECT DriverObject, _In_ PUNICODE_STRING RegistryPath)
{
    WDF_DRIVER_CONFIG config;
    WDF_DRIVER_CONFIG_INIT(&config, EvtDriverDeviceAdd);
    config.EvtDriverUnload = UnloadDriver;
    NTSTATUS status = WdfDriverCreate(DriverObject, RegistryPath, WDF_NO_OBJECT_ATTRIBUTES, &config, WDF_NO_HANDLE);
    
    DbgPrint("Driver loaded");

    return status;
}

NTSTATUS EvtDriverDeviceAdd(_In_ WDFDRIVER Driver,_Inout_ PWDFDEVICE_INIT DeviceInit)
{
    UNREFERENCED_PARAMETER(Driver);
    WDFDEVICE device;
    NTSTATUS status = WdfDeviceCreate(&DeviceInit, WDF_NO_OBJECT_ATTRIBUTES, &device);
    
    return status;
}
```
{% endcode %}

### Enable DbgPrint Monitoring for WinDBG

Change the debug output verbosity:

```
ed kd_default_mask 0xf
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-M1G42F6G7F1E0t-J-fo%2F-M1G7tWDFuO1nDUU1zpc%2Fimage.png?alt=media\&token=4da75386-fcfc-467a-8fab-6ed517a4f731)

[Starting the driver](https://www.ired.team/miscellaneous-reversing-forensics/windows-kernel-internals/loading-a-windows-kernel-driver-osr-driver-loader-debugging-with-source-code) allows us to see the debug output in WinDBG:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-M1G42F6G7F1E0t-J-fo%2F-M1G85ERdnQ5LlAXo4Eu%2Fimage.png?alt=media\&token=5535524d-7dc5-4f45-b3c7-b8e486e96967)

### Enable DbgPrint Monitoring for DbgView

Create a sub-key `Debug Print Filter` if it does not exist:

```
Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Debug Print Filter
```

Add a new DWORD value `DEFAULT` and set its Data field to `0xf`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-M1GbSWj__fCFGbyUZch%2F-M1H0pS6TcssCh6KhUK4%2Fimage.png?alt=media\&token=50cfaa9a-5398-4547-969f-eea6f81580ae)

If we load the driver now and start it, we can see the debug output in DbgView too:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-M1GbSWj__fCFGbyUZch%2F-M1H0iZY9RcOqz2k1oKA%2Fimage.png?alt=media\&token=1c46ec67-7dee-4065-b30b-2ccef75e122c)

### Requested Control is Not Valid for This Service

The below error message is seen if you attempt to stop the WDF driver via OSR Driver Loader or the native sc.exe, even if you have defined the driver unloading routine:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-M1H1aI3s2I2g3Xykd0k%2F-M1H5NBIA6sP1M607lrF%2Fimage.png?alt=media\&token=d49f2163-3094-427b-9076-8e8543e18a08)

I could not find a solution to this, but WDM driver has no such issue - see the code below.

### Simple Windows Driver Model (WDM) Kernel Driver Load and Unload

Below is a simple WDM driver that can be compiled and then loaded and stopped with OSR Driver Loader:

```c
#include <ntddk.h>

void DriverUnload(PDRIVER_OBJECT dob)
{
	UNREFERENCED_PARAMETER(dob);
	DbgPrint("Driver unloaded");
}

NTSTATUS DriverEntry(PDRIVER_OBJECT DriverObject, PUNICODE_STRING RegistryPath) {

	UNREFERENCED_PARAMETER(DriverObject);
	UNREFERENCED_PARAMETER(RegistryPath);

	DriverObject->DriverUnload = DriverUnload;
	DbgPrint("Driver loaded");

	return STATUS_SUCCESS;
}
```

Below shows how our driver is loaded and unloaded via OSR Loader while DbgView prints our DbgPrint output defined in the above `DriverEntry` and `DriverUnload` routines:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-M1H1aI3s2I2g3Xykd0k%2F-M1H4x5cgTU9LsEPFe4G%2Fimage.png?alt=media\&token=89d7cd9f-5519-4963-96e8-ae2997619961)

### References

{% embed url="https://docs.microsoft.com/en-us/windows-hardware/drivers/gettingstarted/writing-a-very-small-kmdf--driver" %}

{% embed url="http://www.osronline.com/article.cfm%5Earticle=295.htm" %}

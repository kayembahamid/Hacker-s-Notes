# Shellcode Reflective DLL Injection

Shellcode reflective DLL injection (sRDI) is a technique that allows converting a given DLL into a position independent shellcode that can then be injected using your favourite shellcode injection and execution technique. In this lab I wanted to try this technique as I think it is an amazing technique to have in your arsenal.

In this lab, I'm playing with the amazing [https://github.com/monoxgas/sRDI](https://github.com/monoxgas/sRDI) written by monoxgas from Silent Break Security.

### Execution

Let's compile a simple x86 DLL - in my case, an odd DLL that pops 2 notepad processes when executed:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LjSvRKitqquU9hLq-KD%2F-LjSvUBjVqzB_3nceoWi%2Fimage.png?alt=media\&token=8f0e05ca-c4a1-44f4-a1d4-c505770b27e2)

Convert the DLL into shellcode. We will get an array of shellcode bytes represented in decimal values:

```csharp
$sc = ConvertTo-Shellcode \\VBOXSVR\Experiments\messagebox\messagebox\Debug\messagebox.dll
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LjSc2tUq-UeoaSiDnaK%2F-LjScZ1IsecIta-eRGP9%2Fimage.png?alt=media\&token=7ee1e4ac-06fa-4b14-9b91-8918db89efbe)

Let's convert them to hex:

```csharp
$sc2 = $sc | % { write-output ([System.String]::Format('{0:X2}', $_)) }
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LjSc2tUq-UeoaSiDnaK%2F-LjSdspQB8-dVUn0PySt%2Fimage.png?alt=media\&token=8ed55723-cde3-4410-b421-be390ff0117b)

Join them all and print to a text file:

```
$sc2 -join "" > shell.txt
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LjSc2tUq-UeoaSiDnaK%2F-LjSpDUX4et_bGhXqEjU%2Fimage.png?alt=media\&token=566149af-3e0e-4ead-a602-9dd074ffb478)

Create a new binary file with the shellcode we got earlier - just copy the hex string (as seen in the above screenshot) and paste it to a new file using HxD hex editor:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LjSvRKitqquU9hLq-KD%2F-LjSw2nFPXwgHVVMttmd%2Fimage.png?alt=media\&token=09ff06f3-b85d-4484-a451-b1a30cf2a7be)

In order to load and execute the shellcode, we will place it in the binary as a resource as described in my other lab [Loading and Executing Shellcode From PE Resources](loading-and-executing-shellcode-from-pe-resources.md):

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LjSvRKitqquU9hLq-KD%2F-LjSw86-sEyrM6zkGbz3%2Fimage.png?alt=media\&token=f9d16a62-976e-4b3a-8f40-264ffdc93976)

Compile and run the binary. If the shellcode runs successfully, we should see two notepad.exe processes popup:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LjSvRKitqquU9hLq-KD%2F-LjSwX8r33rMWYenuqnq%2Fpop-2notepads.gif?alt=media\&token=58cb1eef-4710-46ab-b7be-bc3baa5a5cb6)

### References

{% embed url="https://github.com/monoxgas/sRDI/tree/master/PowerShell" %}

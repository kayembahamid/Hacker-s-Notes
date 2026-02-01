---
description: Defense Evasion, Code Obfuscation
---

# Packed Binaries

## Packed Binaries

For this exercise, I will pack a binary with a well known UPX packer.

### Execution

```csharp
.\upx.exe -9 -o .\nc-packed.exe .\nc.exe
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LK3G3Ez0w3zqw6EaRkP%2F-LK3GPB0aQxdyiaAzCFL%2Fupx-pack.png?alt=media\&token=1cb688ef-ea5e-43ae-8809-cd0a0a3e3fc8)

Note how the file size shrank by 50%!

### Observations

Some of the tell-tale signs of a UPX packed binary are the PE section headers - note the differences between `nc-packed.exe` and `nc.exe`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LK3G3Ez0w3zqw6EaRkP%2F-LK3GPB2aEYxUUE6SMvM%2Fupx-packed-vs-unpacked.png?alt=media\&token=d90ba43e-fae0-4b84-acb7-1578de79f8f1)

Another important observation should be made from the above screenshot - `nc-packed` binary's `Raw Size` (section's size on the disk) is 0 bytes for the UPX0 section (.text/.code section) and therefore much smaller than the `Virtual Size` (space allocated for this section in the process memory), whereas these values in a non-packed binary are of similar sizes. This is another good indicator suggesting the binary may be packed.

Yet another sign of a potentially packed binary is a low(-er) number of imported DLLs and their functions:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LK3G3Ez0w3zqw6EaRkP%2F-LK3GPB50CUh3daHZj1x%2Fupx-imports.png?alt=media\&token=8fe442ae-cf11-4790-bab2-38d307aa7693)

Note how the packed binary only imports one function from the `WSOCK32.dll` and many more are imported by a non-packed binary:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LK3G3Ez0w3zqw6EaRkP%2F-LK3GPB7FBq6n0YpfCMs%2Fupx-sockets.png?alt=media\&token=219e5b5b-9f96-4465-97e8-a5ffa64e0020)

Another classic sign of a packed binary is `KERNEL32.dll` **only** importing a couple of functions, including:`LoadLibraryA` and `GetProcAddress`. These are crucial for the binary as they are used to locate other important functions of the `KERNEL32.dll` located in the process memory, hence packed binaries will almost always have those functions exposed since they are required for the binary to work properly:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LK3G3Ez0w3zqw6EaRkP%2F-LK3GPB8ZqYPTrrBYdM4%2Fupx-kernel.png?alt=media\&token=faa20380-7d13-44a9-828d-a941c925f3ad)

If you have no fancy malware analysis tools to hand, but you have `strings.exe`, you can make a fairly good educated guess whether the binary is packed by just running strings against it and noting the DLL imports - if there's only a few of them (and more importantly - GetProcAddress and LoadLibrary) and they are from KERNEL32.dll - the binary is likely packed:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LK3HoqfUNctqEaIZ90l%2F-LK3H5gmgR92mP-Z27G-%2Fupx-strings.png?alt=media\&token=effb5b6f-fb3a-47c2-b842-e0a173d1daad)

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1045" %}

# Exploring Process Environment Block

## Exploring Process Environment Block

A very brief look into the PEB memory structure found, aiming to get a bit more comfortable with WinDBG and walking memory structures.

### Basics

First of, checking what members the `_PEB` structure actually entails:

```
dt _peb
```

There are many fields in the structure among which there are `ImageBaseAddresss` and `ProcessParameters` which are interesting to us for this lab:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LL65Cjy2bYGSydp-iUi%2F-LL6Lp5XpOLIr66r1WKi%2Fpeb-structure.png?alt=media\&token=d6f4a188-f41c-4496-8596-d4525d76e4f8)

Getting the PEB address of the process:

```bash
0:001> r $peb
$peb=000007fffffd5000
```

The `_PEB` structure can now be overlaid on the memory pointed to by the `$peb` to see what values the structure members are holding/pointing to:

```bash
0:001> dt _peb @$peb
```

`_PEB` structure is now populated with the actual data pulled from the process memory:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LL65Cjy2bYGSydp-iUi%2F-LL6NaFQxL6gvvwjid7b%2Fpeb-overlay.png?alt=media\&token=186ba355-ff58-450b-a92c-eb9f8faad2a9)

Let's check what's in memory at address `0000000049d40000` - pointed to by the `ImageBaseAddress` member of the `_peb` structure:

```cpp
0:001> db 0000000049d40000 L100
```

Exactly! This is the actual binary image of the running process:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LL6NxJTZvtcQsar2ToB%2F-LL6OZ9ELwy69ciz2EBF%2Fpeb-baseimage.png?alt=media\&token=082be540-69de-487b-be5d-d23a9d3d1e42)

Another way of finding the `ImageBaseAddress` is:

```csharp
0:001> dt _peb
ntdll!_PEB
//snip
      +0x010 ImageBaseAddress : Ptr64 Void
//snip

0:001> dd @$peb+0x010 L2
000007ff`fffd5010  49d40000 00000000

// 49d40000 00000000 is little-endian byte format - need to invert
0:001> db 0000000049d40000 L100
```

### Convenience

We can forget about all of the above and just use:

```
!peb
```

This gets us a nicely formatted PEB information of some of the key members of the structure:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LL65Cjy2bYGSydp-iUi%2F-LL6L-1Rg9Cxr5TIPGLF%2Fpeb.png?alt=media\&token=41bc2a38-db79-4911-ba76-731ea950856c)

### Finding Commandline Arguments

One of the interesting fields the PEB holds is the process commandline arguments. Let's find them:

```cpp
dt _peb @$peb processp*
ntdll!_PEB
   +0x020 ProcessParameters : 0x00000000`002a1f40 _RTL_USER_PROCESS_PARAMETERS

dt _RTL_USER_PROCESS_PARAMETERS 0x00000000`002a1f40
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LL6WlTP-5L3r-gm1CTK%2F-LL6UsWoLR5d9A-KQnv4%2Fpeb-cmdline.png?alt=media\&token=aef0e503-a87a-415e-b0e4-e31a7af24a09)

We can be more direct and ask the same question like so:

```cpp
0:001> dt _UNICODE_STRING 0x00000000`002a1f40+70
ntdll!_UNICODE_STRING
 ""C:\Windows\system32\cmd.exe" "
   +0x000 Length           : 0x3c
   +0x002 MaximumLength    : 0x3e
   +0x008 Buffer           : 0x00000000`002a283c  ""C:\Windows\system32\cmd.exe" "
```

or even this:

```cpp
0:001> dd 0x00000000`002a1f40+70+8 L2
00000000`002a1fb8  002a283c 00000000
0:001> du 00000000002a283c
00000000`002a283c  ""C:\Windows\system32\cmd.exe" "
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LL6WlTP-5L3r-gm1CTK%2F-LL6WjLdFXj9UQUJbOpF%2Fpeb-cmdline2.png?alt=media\&token=56bf1edc-cc51-49c5-b06b-97a203d95431)

Since we now know where the commandline arguments are stored - can we modify them? Of course.

### Forging Commandline Arguments

```cpp
0:001> eu 00000000002a283c "cmdline-logging? Are You Sure?"
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LL6eSB-2MiJztVCgE_W%2F-LL6eOnasTLRj-s3J5sF%2Fpeb-cmdline3.png?alt=media\&token=030ac31b-de1b-4e4d-804f-c3f63f4e0fd0)

### \_PEB\_LDR\_DATA <a href="#peb_ldr_data-structure" id="peb_ldr_data-structure"></a>

Getting a list of loaded modules (exe/dll) by the process:

```cpp
// get the first _LIST_ENTRY structure address
0:001> dt _peb @$peb ldr->InMemoryOrderModuleList*
ntdll!_PEB
   +0x018 Ldr                          : 
      +0x020 InMemoryOrderModuleList      : _LIST_ENTRY [ 0x00000000`002a2980 - 0x00000000`002a1e40 ]


// walking the list manually and getting loaded module info
dt _LIST_ENTRY 0x00000000`002a2980
// cmd module
dt _LDR_DATA_TABLE_ENTRY 0x00000000`002a2980

dt _LIST_ENTRY 0x00000000`002a2980 
// ntdll module
dt _LDR_DATA_TABLE_ENTRY 0x00000000`002a2a70

dt _LIST_ENTRY 0x00000000`002a2a70
// kernel32 module
dt _LDR_DATA_TABLE_ENTRY 0x00000000`002a2df0

...loop...
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LL6ug_7rz4dS8NkjoSO%2F-LL6ysCGWOvgaP6e6IY0%2Fpeb-modulelist.png?alt=media\&token=2d0f088a-0fb8-4cd9-b275-afab74490da1)

If we check the loaded modules with `!peb`, it shows we were walking the list correctly:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LL6ug_7rz4dS8NkjoSO%2F-LL6zwVaEEzpD1ZkG9gx%2Fpeb-modules2.png?alt=media\&token=6ef7c299-d6a0-4dd1-bd5b-3ae9b31af95a)

Here is another way to find the first `_LDR_DATA_TABLE_ENTRY`:

```cpp
dt _peb @$peb
dt _PEB_LDR_DATA 0x00000000`774ed640
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLBNnXk5T1ExIA-G6UT%2F-LLBNZS5n-N5YQHMuvZj%2Fpeb-manual1.png?alt=media\&token=3e37f8e4-6a7d-49ab-b1af-dd7cf5959eae)

```cpp
dt _LDR_DATA_TABLE_ENTRY 0x00000000`002a2980
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLBNnXk5T1ExIA-G6UT%2F-LLBN_uPpzxY9He_kIPZ%2Fpeb-manual2.png?alt=media\&token=a9f204bc-f104-4cca-98f5-1dfe39cbeea9)

A nice way of getting a list of linked-list structure addresses is by providing address of the first `list_entry` structure to the command `dl` and specifying how many list items it should print out:

```cpp
0:001> dl 0x00000000`002a2980 6
00000000`002a2980  00000000`002a2a70 00000000`774ed660
00000000`002a2990  00000000`00000000 00000000`00000000
00000000`002a2a70  00000000`002a2df0 00000000`002a2980
00000000`002a2a80  00000000`002a2f70 00000000`774ed670
00000000`002a2df0  00000000`002a2f60 00000000`002a2a70
00000000`002a2e00  00000000`002a3cb0 00000000`002a2f70
00000000`002a2f60  00000000`002a3ca0 00000000`002a2df0
00000000`002a2f70  00000000`002a2e00 00000000`002a2a80
00000000`002a3ca0  00000000`002a41f0 00000000`002a2f60
00000000`002a3cb0  00000000`002defc0 00000000`002a2e00
00000000`002a41f0  00000000`002a3ff0 00000000`002a3ca0
00000000`002a4200  00000000`002e1320 00000000`002a4000
```

Another way of achieving the same would be to use the !list command to list through the list items and dump the info:

```cpp
!list -x "dt _LDR_DATA_TABLE_ENTRY" 0x00000000`002a2980
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLBfgaBmYL1nlR0b2Q7%2F-LLBfVaSh1KZes_LT-QL%2Fpeb-dll-automated.gif?alt=media\&token=c4071cd9-f3d6-4d30-a019-66c860c1232c)

Continuing further:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLBh18T54DPU9eM0_fV%2F-LLBgzuTbr3zquZgRqcR%2Fpeb-dll-automated2.gif?alt=media\&token=dec74639-83f9-4ad0-9e4b-df2aadd80f63)

### Abusing PEB

It is possible to abuse the PEB structure and masquerade one windows processes with another process. See this lab for more:

{% content-ref url="../../offensive-security/defense-evasion/masquerading-processes-in-userland-via-_peb.md" %}
[masquerading-processes-in-userland-via-\_peb.md](../../offensive-security/defense-evasion/masquerading-processes-in-userland-via-_peb.md)
{% endcontent-ref %}

### References

{% embed url="http://windbg.info/doc/1-common-cmds.html#13_breakpoints" %}

{% embed url="https://www.aldeid.com/wiki/PEB_LDR_DATA" %}

{% embed url="https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/-list" %}

---
description: Code execution with embedded Internet Explorer Object
---

# Phishing: Embedded Internet Explorer

## Phishing: Embedded Internet Explorer

In this phishing lab I am just playing around with the POCs researched, coded and described by Yorick Koster in his blog post [Click me if you can, Office social engineering with embedded objects](https://securify.nl/blog/SFY20180801/click-me-if-you-can_-office-social-engineering-with-embedded-objects.html)

### Execution

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLJxiytqzVTtACKX5C5%2F-LLJyjh9CIrwT0n3Rvp5%2Fphishing-iex-video.gif?alt=media\&token=de339ff1-d27e-4397-99da-88a3d1e69ee3)

{% file src="../../../.gitbook/assets/WebBrowser (1).docx" %}

{% file src="../../../.gitbook/assets/poc (1).ps1" %}

### Observations

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLJxiytqzVTtACKX5C5%2F-LLJysTYVw-QHrgMvF2T%2Fphishing-iex-ancestry.png?alt=media\&token=f812e343-1e7e-48bb-9869-ff42a809b86c)

As with other phishing documents, we can unzip the .docx and do a simple hexdump/strings on the `oleObject1.bin` to look for any suspicious strings referring to some sort of file/code execution:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLK05DndfnIsuUfV705%2F-LLK0UeV_qWarNBkci5z%2Fphishing-iex-olebin.png?alt=media\&token=8b70b55d-0cbc-4ad6-a18a-59df52dadaab)

The CLSID object that makes this technique work is a `Shell.Explorer.1` object, as seen here:

```csharp
Get-ChildItem 'registry::HKEY_CLASSES_ROOT\CLSID\{EAB22AC3-30C1-11CF-A7EB-0000C05BAE0B}'
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLK2sE5XTQMWzZS7exM%2F-LLK2l7mDsspEUow5HBF%2Fphishing-explorer-obj.png?alt=media\&token=95036445-3044-4b29-81d5-b7839229d143)

As an analyst, one should inspect the .bin file and look for the {EAB22AC3-30C1-11CF-A7EB-0000C05BAE0B} bytes inside, signifying the `Shell.Explorer.1` object being embedded in the .bin file:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLLGrSLoowguTx6g3S0%2F-LLLHD4o6Fm9OQa8C0Hw%2Fphishing-clsid.png?alt=media\&token=b3603c36-4957-4053-9ba1-29365fa5248b)

### References

{% embed url="https://securify.nl/blog/SFY20180801/click-me-if-you-can_-office-social-engineering-with-embedded-objects.html" %}

<br>

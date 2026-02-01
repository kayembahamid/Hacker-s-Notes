# Phishing: Embedded HTML Forms

## Phishing: Embedded HTML Forms

In this phishing lab I am just playing around with the POCs researched, coded and described by Yorick Koster in his blog post [Click me if you can, Office social engineering with embedded objects](https://securify.nl/blog/SFY20180801/click-me-if-you-can_-office-social-engineering-with-embedded-objects.html)

### Execution

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLP-hT6LHOb2zpqU6DK%2F-LLP0QqjMQH0JmxAI59j%2Fphishing-forms-shell.gif?alt=media\&token=af22377b-053a-440f-a981-ea98cd52c8b0)

{% file src="../../../.gitbook/assets/Forms.HTML.ps1" %}

{% file src="../../../.gitbook/assets/Forms.HTML.docx" %}

### Observations

These types of phishing documents can be identified by looking for the CLSID 5512D112-5CC6-11CF-8D67-00AA00BDCE1D in the embedded `.bin` files:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLP106hwnfRyN8wya1R%2F-LLP11w7b5dwx2QN9I6C%2Fphishing-forms-clsid.png?alt=media\&token=f139d541-8ebe-4037-9e08-d7aa78ec31aa)

...as well as inside the activeX1.xml file:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLP-hT6LHOb2zpqU6DK%2F-LLP0Pamd6EwRl-1AWn4%2Fphishing-forms-xml.png?alt=media\&token=2559732a-d3c7-4aba-ab48-3285b2336b9c)

As usual, MS Office applications spawning cmd.exe or powershell.exe should be investigated:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLP1ShUd5SORlNtqYm0%2F-LLP1VLs6A-1Aq2xHze7%2Fphishing-forms-ancestry.png?alt=media\&token=674b535a-a606-48d2-b6bc-3d022750552f)

### References

{% embed url="https://securify.nl/blog/SFY20180801/click-me-if-you-can_-office-social-engineering-with-embedded-objects.html" %}

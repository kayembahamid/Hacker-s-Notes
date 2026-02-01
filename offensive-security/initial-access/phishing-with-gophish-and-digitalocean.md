---
description: >-
  This lab is dedicated to exploring one of the phishing frameworks GoPhish. I
  will be installing and configuring GoPhish on a DigitalOcean VPS running
  Ubuntu Linux distribution.
---

# Phishing with GoPhish and DigitalOcean

### Configuring Environment

#### DigitalOcean VPS

The dropled that I have created got assigned an IP address `68.183.113.176`

Let's login to the VPS and install the mail delivery agent:

{% code title="attacker\@kali" %}
```csharp
ssh root@68.183.113.176
apt-get install postfix
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVoQC4imLeTkkR3ODWT%2F-LVoS_OcY-ohW-hUdMGy%2FScreenshot%20from%202019-01-09%2021-12-51.png?alt=media\&token=76741890-668e-4ff8-ba5d-077de8419ad9)

Point `mynetworks` variable in postfix config to the IP we got assigned in DigitalOcean:

{% code title="attacker\@vps" %}
```csharp
nano /etc/postfix/main.cf
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVjb_jgJ3XJ0Y4PvbAX%2F-LVjhJgshVbLYvKNUWvs%2FScreenshot%20from%202019-01-08%2022-37-41.png?alt=media\&token=ee72b7e9-52e7-4c3f-bf53-0c6d97b16c57)

#### Configure DNS Zones

Create an `A` record `mail` that points to the VPS IP and an `MX` record that points to `mail.yourdomain`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVjb_jgJ3XJ0Y4PvbAX%2F-LVjh1v1ZZiqHRqukbtM%2FScreenshot%20from%202019-01-08%2022-56-12.png?alt=media\&token=6c9ea02b-8064-4c01-8670-df1e7736d182)

#### Install GoPhish

{% code title="attacker\@vps" %}
```csharp
wget https://github.com/gophish/gophish/releases/download/0.7.1/gophish-v0.7.1-linux-64bit.zip
apt install unzip
unzip gophish-v0.7.1-linux-64bit.zip 
chmod +x gophish
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVjb_jgJ3XJ0Y4PvbAX%2F-LVjhGWqEbjavf_0m4I1%2FScreenshot%20from%202019-01-08%2022-40-21.png?alt=media\&token=27ce8c7b-84d9-47ac-b410-4e47491b5aae)

### Execution

Launching GoPhish is simple:

{% code title="attacker\@vps" %}
```csharp
./gophish
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVjb_jgJ3XJ0Y4PvbAX%2F-LVjhF48ilLNAa6Xp4Ah%2FScreenshot%20from%202019-01-08%2022-41-09.png?alt=media\&token=3ad21ef2-a7d7-47cc-8284-23df5ee12567)

GoPhish admininistration panel is bound to 127.0.0.1:3333 by default, so we can either modify the config and change it to listen on 0.0.0.0 (all interfaces) if we want to access the admin panel from the Internet or create a local SSH tunnel if we want to restrict access to local network only. Let's do an SSH tunnel:

{% code title="attacker\@kali" %}
```csharp
ssh root@68.183.113.176 -L3333:localhost:3333 -N -f
```
{% endcode %}

We can now access the GoPhish admin panel via `https://127.0.0.1:3333` from our Kali box. After creating user groups (phish targets), landing pages (phishing pages victims will see if they click on our phishing links), etc, we can create an email template - the email that will be sent to the unsuspecting victims as part of a phishing campaign that we will create in the next step:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVjb_jgJ3XJ0Y4PvbAX%2F-LVjhDD6NGaiOe-gENde%2FScreenshot%20from%202019-01-08%2022-45-34.png?alt=media\&token=70df0ca3-8dec-43ce-b12a-80b7bfe62073)

Below is a quick demo of how a new campaign is put together once all the other pieces mentioned above are in place (users, templates, landing pages):

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVjb_jgJ3XJ0Y4PvbAX%2F-LVjhAvK5i_NB8AkT2WN%2FPeek%202019-01-08%2022-47.gif?alt=media\&token=606411e8-6e21-475c-aa9c-55a94210cbdd)

### Receiving the Phish

Below is the actual end result of our mock phish campaign:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVjb_jgJ3XJ0Y4PvbAX%2F-LVjh5tpohiTzCTxvPI5%2FScreenshot%20from%202019-01-08%2022-50-47.png?alt=media\&token=718d8682-91b8-456d-b951-a1f0ae9ea4ff)

The URL found in the above phish email takes the user to our mock phishing page:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVjb_jgJ3XJ0Y4PvbAX%2F-LVjh3rv8BVL-OaPUcZr%2FScreenshot%20from%202019-01-08%2022-51-21.png?alt=media\&token=f71ce484-d9b3-4644-940a-7643d6e21dd6)

### Campaign Results

Switching to `Campaigns` section of the admin panel, we can see how many emails were sent as part of the campaign, how many of them were opened and how many times the phishing URL was clicked:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVjj9cH8EEv_ZKQTtXx%2F-LVjjCGda2tGS6R2SWbo%2FScreenshot%20from%202019-01-08%2023-11-32.png?alt=media\&token=359dc4a1-200b-4698-9a1c-5b6f65b854b6)

### References

{% embed url="https://docs.getgophish.com/user-guide/building-your-first-campaign/creating-the-template" %}

{% embed url="http://www.postfix.org/BASIC_CONFIGURATION_README.html" %}

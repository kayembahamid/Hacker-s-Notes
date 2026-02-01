# Spiderfoot 101 with Kali using Docker

This lab walks through some simple steps required to get the OSINT tool Spiderfoot up and running on a Kali Linux using Docker.

Spiderfoot is an application that enables you as a pentester/red teamer to collect intelligence about a given subject - email address, username, domain or IP address that may help you in planning and advancing your attacks against them.

### Download Spiderfoot

Download the Spiderfoot linux package from [https://www.spiderfoot.net/download/](https://www.spiderfoot.net/download/) and extract it to a location of your choice on your file system.\
I extracted it to `/root/Downloads/spiderfoot-2.12.0-src/spiderfoot-2.12`

and made it my working directory:

```csharp
cd /root/Downloads/spiderfoot-2.12.0-src/spiderfoot-2.12
```

### Upgrade PIP

You may need to upgrade the pip before it starts giving you trouble:

```csharp
pip install --upgrade pip
```

### Build Docker Image

Build the spiderfoot docker image :

```
docker build -t spiderfoot .
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTwEKXVlfa7dFAOdcJx%2F-LTwIHAEEMAhTPym0UPX%2FScreenshot%20from%202018-12-17%2013-13-33.png?alt=media\&token=c9e1a1ef-6522-4db8-87fd-1e0a7dcbc4a6)

Check if the image got created successfully:

```
docker images
```

You should see the spiderfoot image creted seconds ago:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTwEKXVlfa7dFAOdcJx%2F-LTwJ7ksRwafMptTtGHO%2FScreenshot%20from%202018-12-17%2013-00-55.png?alt=media\&token=5db9bba8-8c04-4f3e-aa3a-deb84744a2f1)

### Run the Spiderfoot Docker

```csharp
docker run -p 5009:5001 -d spiderfoot
```

The above will run previously created spiderfoot image in the background and expose a TCP port 5009 on the host computer. Any traffic sent to `host:5009` will be forwarded to the port 5001 on the docker where spiderfoot is running and listening.

To check if the docker image is running, we can do:

```
docker ps
```

The below confirms the docker is indeed running the spiderfoot image and is listening on port 5001:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTwEKXVlfa7dFAOdcJx%2F-LTwJo09kLZqqr_NiY_K%2FScreenshot%20from%202018-12-17%2013-20-22.png?alt=media\&token=ae2fecdb-6dae-4c92-bd2f-aad7e145667b)

Below confirms that the host machine has now exposed the TCP port 5009 (which forwards traffic to the docker's port 5001):

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTwEKXVlfa7dFAOdcJx%2F-LTwJfLBr76AlSt7LMp3%2FScreenshot%20from%202018-12-17%2013-02-03.png?alt=media\&token=598f28c7-9336-46bf-94ae-5ae002476afa)

### Using Spiderfoot

Navigate to your host:5009 to access the spiderfoot UI and start a new scan:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTwEKXVlfa7dFAOdcJx%2F-LTwKG8JvgSv50GOlhwG%2FScreenshot%20from%202018-12-17%2012-57-59.png?alt=media\&token=5eabb28c-52f1-4055-8975-b906a2831acc)

During the scan, we can start observing various pieces of data being returned from the internet:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTwEKXVlfa7dFAOdcJx%2F-LTwKVsIRUq7gedGiZt_%2FScreenshot%20from%202018-12-17%2012-58-32.png?alt=media\&token=73519952-2fb2-40e3-8670-11bd79888dc9)

Drilling down to one of the above categories - DNS records:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LTwEKXVlfa7dFAOdcJx%2F-LTwKbO0pkqxMl38C7pE%2FScreenshot%20from%202018-12-17%2012-58-45.png?alt=media\&token=0d54eed8-0f99-4709-a08b-ea6b27e4f10c)

### References

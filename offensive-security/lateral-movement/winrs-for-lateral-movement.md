# WinRS for Lateral Movement

## WinRS for Lateral Movement

It's possible to use a native Windows binary `winrs` to connect to a remote endpoint via `WinRM` like so:

```
winrs -r:ws01 "cmd /c hostname & notepad"
```

Below shows how we connect from `DC01` to `WS01` and execute two processes `hostname`,`notepad` and the process partent/child relationship for processes spawned by the `winrshost.exe`:

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-MMpw9M1HiNhR-i7oBeD%2F-MMpwJuIlXGuCCD3N8VZ%2Fimage.png?alt=media\&token=93fc56d7-1f50-475c-91bc-0397f964488f)

### References

{% embed url="https://bohops.com/2020/05/12/ws-management-com-another-approach-for-winrm-lateral-movement/amp/" %}

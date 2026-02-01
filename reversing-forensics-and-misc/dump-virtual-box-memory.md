# Dump Virtual Box Memory

## Dump Virtual Box Memory

### List Available VMs

```erlang
cd "C:\Program Files\Oracle\VirtualBox\"
.\VBoxManage.exe list vms

...
"win1002 debugee" {5f176ebb-a0cc-4dc7-9c6f-988fcbcca867}
...
```

### Enable Debug Mode

{% code title="linux host" %}
```bash
mantvydas@~: virtualbox --startvm 'yourVMName or VM UUID' --dbg
```
{% endcode %}

### Dump VM Memory

Launch the VirtualBox debug console by navigating to "Debug" menu an select "Command Line":

![](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIa_oeZz2CvDYlbpmlQ%2F-LIaaB-m1ieDNZur2HfN%2Fvbox-menu.png?alt=media\&token=340e3f20-b22d-4051-84b0-7fc9e945e4ef)

Once you select "Command Line", you will be presented with a console that looks like this:

![memory dump will be a raw file dumped to /home/youruser directory](https://386337598-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LFEMnER3fywgFHoroYn%2F-LIaYJJkXcYXwFvJN0es%2F-LIaZmJjRIjVb7NaLhVr%2Fvbox-debug.png?alt=media\&token=c56ccd56-d87e-44d2-8fe8-f0763a5dae7d)

To create a memory dump, issue the below command (also highlighted in the above graphic):

{% code title="VM\@virtualbox" %}
```
VBoxDbg> .pgmphystofile 'w7-nc-shell.bin'
```
{% endcode %}

### Persistence

If you want the debug options to be always available, you can:

* export `VBOX_GUI_DBG_ENABLED=true` before launching the VM or
* put export `VBOX_GUI_DBG_ENABLED=true` in your `.bashrc` or `/etc/environment`

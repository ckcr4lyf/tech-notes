# Remote Wireshark

In many cases, it can be useful to view the traffic of a remote machine locally via Wireshark.

Although tcpdump via SSH in the terminal is ok for some initial poking around, the convenicne of Wireshark w/ a GUI, deeper protocol analysis etc. is quite nifty.

There are two ways to do so, depending on tools available. In both, the rudimentary filter `not tcp port 22` is used to not show SSH traffic, which otherwise would lead to a positive feedback loop.

## using dumpcap

If the remote machine has `dumpcap` installed, then it can be launched using via the SSH command with output redirected to `stdout`, which is then piped to Wireshark locally:

```
wireshark -k -i <(ssh HOSTNAME "dumpcap -P -w - -f 'not tcp port 22'")
```

##  using tcpdump

If the remote machine has `tcpdump` installed, then it can be launched via the following command:

```
wireshark -k -i <(ssh HOSTNAME "sudo tcpdump -w - -f 'not tcp port 22'")
```


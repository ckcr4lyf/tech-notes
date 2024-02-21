# Exposting a local webservice over the internet via Cloudflare Tunnel

Go to Cloudflare Dashboard -> Zero Trust -> Networks -> Tunnels

Create a tunnel via GUI, it will give command to run to install `cloudflared` and setup the tunnel on the host machine. This step can sometimes take 60-120 seconds! You'll get something like:

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 16.9M  100 16.9M    0     0  13.2M      0  0:00:01  0:00:01 --:--:-- 38.2M
Selecting previously unselected package cloudflared.
(Reading database ... 106916 files and directories currently installed.)
Preparing to unpack cloudflared.deb ...
Unpacking cloudflared (2024.2.0) ...
Setting up cloudflared (2024.2.0) ...
Processing triggers for man-db (2.10.2-1) ...
2024-02-19T12:13:16Z INF Using Systemd
2024-02-19T12:15:05Z INF Linux service for cloudflared installed successfully
```

Then, again in the Cloudflare Dashboard, just attach it to a subdomain and setup the local port, ezpz!

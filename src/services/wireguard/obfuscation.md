# Obfuscation

Certain firewalls might try and block outbound wireguard connections to a server.

Typically, wireguard as a protocol is quite easy to identify if some is implementing DPI (Deep Packet Inspection). This is harder to hide. However, if the firewall is somewhat naive, e.g. just looking for the default port or something, there are ways around this.

## Offering wireguard on additional ports

Some firewalls may block the known Wireguard port (51820). Even stricter firewalls may just block UDP traffic on all but very few ports.

Typically, port 53 will not be blocked (DNS). Some other more common UDP ports include:

* 123 (NTP - Network Time Protocol)
* 443 (HTTPS over QUIC)
* 1725 (Steam - Gaming software)
* 27015 (Common Counter Strike server port)

If any of these ports are free on your server, you can use IPtables to configure forwarding these ports to the wireguard port transparently, so clients can try connecting on these alternative ports if `51820` is blocked. 

No changes are needed on the server's wireguard config.

The iptables rules look like:

```
iptables -A PREROUTING -d 1.2.3.4/32 -i ens3 -p udp -m multiport --dports 53,123,1725,27015 -j REDIRECT --to-ports 51820 -t nat
```

Where:

* `1.2.3.4` is the public IP of the server (that wireguard is listening on)
* `ens3` is the corresponding 
* `53,123,1725,27015` is the list of UDP ports you want to use as backups for wireguard

# Client

## Creating a client config for routing all traffic "via VPN"

In a temporary directory:

```
wg genkey | tee private.key
cat private.key | wg pubkey | tee public.key
```

Then, make a config file e.g. `wireguard_config.conf` :

```
[Interface]
PrivateKey = [private key just generated]
Address = 10.10.0.3/24 (an IP on the subnet of the server's wireguard interface)

[Peer]
PublicKey = [server public key]
AllowedIPs = 0.0.0.0/0 (all traffic to be routed through this tunnel)
Endpoint = [server public IP address:port]
```

You then need to add the client as authorized to connect to the server. On the server, run:

```
wg set wg1 peer [public key just generated] allowed-ips 10.10.0.3
```

Replace `10.10.0.3` with whatever IP was chosen in the config file for the client above.

To add the config to something like a smartphone, you can qrencode it via:

```
qrencode -t ansiutf8 < wireguard_config.conf
```

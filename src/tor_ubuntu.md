# Installing tor on ubuntu

## Basic steps

All these should be done in a root terminal.

1. Install https transport for apt

```
apt install apt-transport-https
```

2. Add the tor distribution file in apt sources (i.e. `/etc/apt/sources.list.d/tor.list`)

First get your distribution name:

```
$ lsb_release -c
Codename:	jammy
```

Use this in the file:

```
deb     [signed-by=/usr/share/keyrings/tor-archive-keyring.gpg] https://deb.torproject.org/torproject.org jammy main
deb-src [signed-by=/usr/share/keyrings/tor-archive-keyring.gpg] https://deb.torproject.org/torproject.org jammy main
```

3. Get the tor GPG key

```
wget -qO- https://deb.torproject.org/torproject.org/A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89.asc | gpg --dearmor | tee /usr/share/keyrings/tor-archive-keyring.gpg >/dev/null
```

4. Install tor!

```
apt update
apt install tor deb.torproject.org-keyring
```


## References

Mostly everything is from https://support.torproject.org/apt/tor-deb-repo/

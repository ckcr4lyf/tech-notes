# Running a monero node (pruned) on a VPS

Note: The pruned node still needs ~50GB!

## Downloading & installing

Monero distributes a tard set of binaries directly, which we will save into `/opt`. 

In a root terminal:

```
mkdir -p /opt/monero
chown -R ubuntu:ubuntu /opt/monero
```

Then as regular user:


```
cd /tmp
wget https://downloads.getmonero.org/linux64
tar -xjvf linux64 --strip-components=1 -C /opt/monero
```

## Run the node in prune mode:

(`cd` into `/opt/monero` first...)


```
./monerod --prune-blockchain --detach
```

You can check progress by tailing the log:

```
tail -f ~/.bitmonero/bitmonero.log
```

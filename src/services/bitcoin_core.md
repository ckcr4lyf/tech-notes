# Running bitcoin-core on a VPS

We will install and run bitcoin-core on a VPS to support the bitcoin network.

Since a VPS would typically not have enough resources for the whole 300GB+ blockchain, we will use the pruned blockchain which is ~7GB.

## Downloading & installing

Find the download link for the latest bitcoin-core from https://bitcoin.org/en/download

Then, download it to a tmp directory, extract it, and install it:

```
cd /tmp
wget https://bitcoin.org/bin/bitcoin-core-25.0/bitcoin-25.0-x86_64-linux-gnu.tar.gz
tar xvf bitcoin-25.0-x86_64-linux-gnu.tar.gz
sudo install -m 0755 -o root -g root -t /usr/local/bin bitcoin-25.0/bin/*
```

## Pruning

Before running the bitcoin daemon, setup the bitcoin config to prune the chain:

```
mkdir -p ~/.bitcoin
echo "prune=550" >> ~/.bitcoin/bitcoin.conf
```

## Running the node

For simplicity, in just a `screen`, you can run `bitcoind`

TODO: Service or something

# Connecting a monero wallet to a node over tor

This assumes you've tor already setup on your machine.

To connect the wallet CLI to the monero network over tor, you can do:


```
./monero-wallet-cli --proxy 127.0.0.1:9050 --daemon-address lrtrju7tz72422sjmwakygfu7xgskaawiqmfulmssfzx7aofatfkmvid.onion:18089
```

Here `--proxy 127.0.0.1:9050` assumes tor is listening on the default port, and `--daemon-address` is the remote onion node.

If you're not running your own node, you can find a list at [monero.fail](https://monero.fail/?chain=monero&network=mainnet&onion=on&all=true)

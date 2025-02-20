# TLS Inspection

Most programs will use TLS to encrypt their HTTP requests to remote servers, so in Wireshark you'd be unable to view the actual data apart from some TLS metadata.

If the application supports dumping TLS secrets via [SSLKEYLOGFILE](https://datatracker.ietf.org/doc/draft-ietf-tls-keylogfile/), then you can instruct wireshark to use this file to decrypt the TLS communcation.

## Specifying the keylogfile in wireshark

Navigate to the TLS submenu via:

* Edit -> Preferences -> Protocols -> TLS

Then in the `(Pre)-Master-Secret log filename`, put the path to the keylogfile, e.g. `/tmp/sslkeylogfile.txt`

## Inspecting Node.JS traffic

Node.JS supports dumping TLS secrets via the [`--tls-keylog` parameter](https://nodejs.org/docs/latest/api/cli.html#--tls-keylogfile).

To set it as an environment variable, you can do:

```
export NODE_OPTIONS="--tls-keylog=/tmp/sslkeylogfile.txt"
```

If a node application then makes a TLS connection using the native TLS libraries in Node.JS, it will log the secrets to the specified file to be inspected via Wireshark.

## cURL

cURL respects the [`SSLKEYLOGFILE` environment variable](https://everything.curl.dev/usingcurl/tls/sslkeylogfile.html).

So you can just set it to something like

```
export SSLKEYLOGFILE=/tmp/sslkeylogfile.txt
```

and then run cURL commands.

# CA Signed TLS certificates

Sometimes it can be useful to get TLS certificates signed by a CA. To do this for free, you need a domain name.

Usually ACME clients like certbot will auto-detect a webserver and use that to "serve" the challenge.

However this required pointing the domain to an IP address, which can be persisted in DNS history records. Another way to do it, from ANY machine, is via the DNS challenge.

## ACME DNS Challenge

We will use certbot for this. Assuming the domain you want a certificate for is `example.com`, then the following command will suffice:

```
certbot -d example.com --manual --preferred-challenges dns certonly
```

This will ask you to create a DNS TXT record with a certain value, to prove you control the domain name. Once that is done, you will have the signed certificate and private key on your machine, to do whatever you want with.

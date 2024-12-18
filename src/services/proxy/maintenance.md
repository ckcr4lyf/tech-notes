# Maintenance

## Rotating TLS certs

If you use stunnel to host an HTTPS proxy in addition to the HTTP proxy, then from tme to time you would need to rotate the TLS certs.

Generating new certs can be done by [following the guide here](/services/nginx/signed.cert.md).

Once rotated, assuming the `stunnel.conf` points the the location of the new certs, it needs to be restarted

```
kill -9 $(pgrep stunnel)
stunnel
```

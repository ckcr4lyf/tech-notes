# Maintenance

## Rotating TLS certs

TLS is used to protect communication on two fronts:

* Postfix (SMTP): via STARTTLS on port 587
* Dovecot (IMAP): via TLS on port 993

We need to generare the certs for the domain on the MX record, e.g. `mail.saxrag.com` for me. These can be generated [following the guide here](/services/nginx/signed.cert.md).

Once rotated, postfix and dovecot need to be restarted. This can be done via:

```
postfix stop && postfix start
systemctl restart dovecot
```

You can use `openssl` to verify the new certs are being offered, e.g. for my domain:

```
openssl s_client -starttls smtp -connect mail.saxrag.com:587
openssl s_client -connect mail.saxrag.com:993
```

## TLS configs

YOu can find the location of the TLS certs/keys used by postfix/dovecot typically at:

```
/etc/postfix/main.cf
/etc/dovecot/conf.d/10-ssl.conf
```
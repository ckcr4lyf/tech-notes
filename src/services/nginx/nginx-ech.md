# Running ECH enabled nginx

ECH is a good way to hide the SNI of the website being connected to.

However, since ECH is not yet an official RFC (March 2024) and not many libs support it (especially on the server side), we need to do some hacky stuff to deploy it.

Specifically, we will use [openssl](https://github.com/sftcd/openssl/tree/ECH-draft-13c) & [nginx](https://github.com/sftcd/nginx/tree/ECH-experimental) forks by the greate people at [defo.ie](https://defo.ie/) to deploy ECH support.

## Building

### Building the OpenSSL fork

This fork adds support for ECH

### Building the nginx fork

### Some nginx dirs

```
cd ~/code/openssl-for-nginx/esnistuff
mkdir nginx
cd nginx
mkdir logs
mkdir www
mkdir echkeydir
```

### Generate the ECH keys

Put whatever `public_name` here you want snoopers to think you're connecting to (via SNI)!

```
cd ~/code/openssl-for-nginx/esnistuff
../apps/openssl ech -public_name example.com -pemout ./nginx/echkeydir/example.pem.ech
```

### Run nginx

Make sure to put the `custom_ech.conf` inside the `nginx` folder in `esnistuff`.

```
cd ~/code/openssl-for-nginx/esnistuff
../../nginx/objs/nginx -c custom_ech.conf
```

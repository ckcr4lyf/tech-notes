# Running ECH enabled nginx

ECH is a good way to hide the SNI of the website being connected to.

However, since ECH is not yet an official RFC (March 2024) and not many libs support it (especially on the server side), we need to do some hacky stuff to deploy it.

Specifically, we will use [openssl](https://github.com/sftcd/openssl/tree/ECH-draft-13c) & [nginx](https://github.com/sftcd/nginx/tree/ECH-experimental) forks by the greate people at [defo.ie](https://defo.ie/) to deploy ECH support.

## Building

### Building the OpenSSL fork

This fork adds support for ECH into openssl.

```
cd ~
mkdir -p code
cd code
git clone https://github.com/sftcd/openssl.git openssl-for-nginx
cd openssl-for-nginx
git checkout ECH-draft-13c
./config -d
make
```

### Building the nginx fork

This fork uses the forked OpenSSL along with Nginx to support incoming ECH TLS handshakes.

```
cd ~/code
git clone https://github.com/sftcd/nginx.git
cd nginx
git checkout ECH-experimental
./auto/configure --with-debug --prefix=nginx --with-http_ssl_module --with-openssl=$HOME/code/openssl-for-nginx  --with-openssl-opt="--debug"
make
```

Now nginx should be compiled with the ECH compatible OpenSSL!

### Some nginx dirs

We need to create some directories which nginx expects for logs and stuff.

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

## Reference

* https://github.com/sftcd/openssl/blob/9e66beb759d274f3069e19cc96c793712e83122c/esnistuff/nginx.md?plain=1#L172
* https://github.com/sftcd/openssl/issues/26
* https://guardianproject.info/2023/11/10/quick-set-up-guide-for-encrypted-client-hello-ech/

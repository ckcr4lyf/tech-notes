# Running ECH enabled nginx

ECH is a good way to hide the SNI of the website being connected to.

However, since ECH is not yet an official RFC (March 2024) and not many libs support it (especially on the server side), we need to do some hacky stuff to deploy it.

Specifically, we will use [openssl](https://github.com/sftcd/openssl/tree/ECH-draft-13c) & [nginx](https://github.com/sftcd/nginx/tree/ECH-experimental) forks by the great people at [defo.ie](https://defo.ie/) to deploy ECH support.

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

## Deploying

To deploy a website with ECH support behind nginx, we need to generate ECH keys, update our DNS, and then configure nginx to use ECH.

The steps are outlined below:

### Generate the ECH keys

Put whatever `public_name` here you want snoopers to think you're connecting to (via SNI)!

```
cd ~/code/openssl-for-nginx/esnistuff
../apps/openssl ech -public_name example.com -pemout ./nginx/echkeydir/example.pem.ech
```

### Setup DNS records for ECHConfig

Once you generate the ECH keys, the contents will look something like this:

```
-----BEGIN PRIVATE KEY-----
[REDACTED]
-----END PRIVATE KEY-----
-----BEGIN ECHCONFIG-----
AD7+DQA6hAAgACAmYD8cK8laATn/iKI1jt79RGkFzoppTl0LVpshC/Q1VQAEAAEAAQALZXhhbXBsZS5jb20AAA==
-----END ECHCONFIG-----
```

The base64 text is what you need to add to your domains "HTTPS" DNS record.

For example, if your domain name (whatever value you used for `public_name` in the keygen step doesn't matter) is `rfc5746.mywaifu.best` , then you need to add a HTTPS record in your DNS, as such:

![Cloudflare DNS example for ECH HTTPS](../../assets/ech_dns_https.png)

Note: the value should be: `ech="THE BASE 64 ECHCONFIG"`

### Configure nginx

A sample configuration file could look like this: (`nginx-ech.conf`):

```
worker_processes  1;
error_log  logs/error.log  info;

events {
    worker_connections  1024;
}

http {
    access_log          logs/access.log combined;
    ssl_echkeydir        echkeydir;
    server {
        listen              443 default_server ssl;
        ssl_certificate     cadir/domain.crt;
        ssl_certificate_key cadir/domain.key;
        ssl_protocols       TLSv1.3;
        server_name         rfc5746.mywaifu.best;
        location / {
            root   www;
            index  index.html index.htm;
        }
    }
}
```

Replace `server_name` with whatever your real server (domain) name is, e.g. for me it is `rfc5746.mywaifu.best`). Now put this config in `~/code/openssl-for-nginx/esnistuff/nginx/nginx-ech.conf` .

### Run nginx


```
cd ~/code/openssl-for-nginx/esnistuff
../../nginx/objs/nginx -c nginx-ech.conf
```

## Reference

* https://github.com/sftcd/openssl/blob/9e66beb759d274f3069e19cc96c793712e83122c/esnistuff/nginx.md?plain=1#L172
* https://github.com/sftcd/openssl/issues/26
* https://guardianproject.info/2023/11/10/quick-set-up-guide-for-encrypted-client-hello-ech/

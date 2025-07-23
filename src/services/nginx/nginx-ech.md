# Running ECH enabled nginx

ECH is a good way to hide the SNI of the website being connected to.

However, since ECH is not yet an official RFC (Feb 2025) and not many libs support it (especially on the server side), we need to do some hacky stuff to deploy it.

Specifically, we will use [openssl](https://github.com/sftcd/openssl/tree/ECH-draft-13c) & [nginx](https://github.com/sftcd/nginx/tree/ECH-experimental) forks by the great people at [defo.ie](https://defo.ie/) to deploy ECH support.

## Building

### Environment

Assuming a clean VPS. We're gonna do all our work in a `ech` directory in the home folder

```
sudo apt-get install libpcre3 libpcre3-dev
```

```
cd ~
mkdir -p ech
```

### Building the OpenSSL fork

This fork adds support for ECH into openssl.

```
cd ~
mkdir -p code
cd code
git clone -b ECH-draft-13c --single-branch https://github.com/sftcd/openssl.git
cd openssl
./config -d
make -j$(nproc)
```

### Building the nginx fork

This fork uses the forked OpenSSL along with Nginx to support incoming ECH TLS handshakes.

```
cd ~/code
git clone -b ECH-experimental --single-branch https://github.com/sftcd/nginx.git
cd nginx
./auto/configure --with-debug --prefix=nginx --with-http_ssl_module --with-openssl=$HOME/ech/openssl  --with-openssl-opt="--debug"
make -j$(nproc)
```

Now nginx should be compiled with the ECH compatible OpenSSL!

The binary should be at `~/ech/nginx/objs/nginx`

## Configs & keys

### Config directory

We'll now make a config directory for our ECH keys, TLS key & nginx configuration & data.

```
cd ~/ech
mkdir -p conf
cd conf
mkdir -p nginx/www
mkdir -p nginx/logs
mkdir -p nginx/conf/echkeydir
mkdir -p nginx/conf/tlskeydir
```

### Generate ECH keys

Next, we'll generate our keys for ECH, using the OpenSSL we compiled with ECH support. The public key is what will be used to encrypt the ClientHello.

Choose any public_name you want apeparing in the OuterSNI (which passive eavesdroppers can see). For instance:

```
cd ~/ech/conf/nginx/conf/echkeydir
../../../../openssl/apps/openssl ech -public_name cia.gov -pemout cia.gov.pem.ech
```

This file also contains the ECHCONFIG, which you'll need in the next step to setup DNS records.

### Generate TLS keys

These are the keys that will be actually used for the TLS connection with the Inner SNI (real hostname). You'd likely want them signed by a commonly trusted CA, such as Let's Encrypt. 

Actually generating the TLS keys is out of scope, but a guide [can be found here](../nginx/signed.cert.md).

The certificate and key should be placed in

```
~/ech/conf/nginx/conf/tlskeydir/domain.crt
~/ech/conf/nginx/conf/tlskeydir/domain.key
```

accordingly.

## Deploying

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
        listen              4443 default_server ssl;
        ssl_certificate     tlskeydir/domain.crt;
        ssl_certificate_key tlskeydir/domain.key;
        ssl_protocols       TLSv1.3;
        server_name         rfc5746.mywaifu.best;
        location / {
            root   www;
            index  index.html index.htm;
        }
    }
}
```

Replace `server_name` with whatever your real server (domain) name is, e.g. for me it is `rfc5746.mywaifu.best`). Now put this config in `~/ech/conf/nginx/conf/nginx.conf` .

### Run nginx


```
cd ~/ech/conf

# test configuration
../nginx/objs/nginx -t

# actuall start NGINX
../nginx/objs/nginx
```

## Bonus: Multiple ECHConfigs with different SNIs

If you configure nginx to listen on multiple ports, you can advertise separate ECHConfigs for each port, with their own SNI. Specifically, the ECHConfig advertised in the HTTPS RR follows "Port Prefix Naming" as per [Section 2.3 of RFC9460](https://datatracker.ietf.org/doc/html/rfc9460#section-2.3-7).

### Generating a new ECHConfig

Let's say, now instead of `example.com`, we want to use the SNI `cia.gov`. Let's first generate an ECHConfig for this domain:

```
cd ~/ech/conf/nginx/conf/echkeydir
../../../../openssl/apps/openssl ech -public_name cia.gov -pemout cia.gov.pem.ech
```

### Tell NGINX to listen on this port

If we want port 4443 to advertise this ECHConfig and be connectable, we just add a listen directive for this port in our NGINX config:

```
    listen 4443 default_server ssl;  
```

### Advertise the ECHConfig

Finally, we need a new DNS HTTPS record for specifically for this port, based on "Port Prefix Naming". So add a new HTTPS record for your domain, with the target as:

```
_4443._https.rfc5746.mywaifu.best  
```

Note: The new part is `_4443._https.`! This will tell browsers (or well, more generally ECH capable clients) the ECHConfig to use on this port.

### Reload NGINX

```
cd ~/ech/conf

# test configuration
../nginx/objs/nginx -t

# actuall start NGINX
../nginx/objs/nginx -s reload
```

## Reference

* https://github.com/sftcd/openssl/blob/9e66beb759d274f3069e19cc96c793712e83122c/esnistuff/nginx.md?plain=1#L172
* https://github.com/sftcd/openssl/issues/26
* https://guardianproject.info/2023/11/10/quick-set-up-guide-for-encrypted-client-hello-ech/

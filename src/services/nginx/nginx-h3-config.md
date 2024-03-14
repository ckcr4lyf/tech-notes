# Setting up TLS and Config

We will now generate our TLS keys and nginx config

## Generate TLS key & cert

```
cd /usr/local/nginx
openssl req -newkey rsa:2048 -keyout domain.key -x509 -days 365 -out domain.crt
```

## Setup the nginx config

Write this to `/usr/local/nginx/conf/nginx.conf`

```
events{}

http {
    log_format quic '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent" "$http3"';

    access_log logs/access.log quic;

    server {
        # for better compatibility it's recommended
        # to use the same port for quic and https
        listen 443 quic reuseport;
        listen 443 ssl;

        ssl_certificate     /usr/local/nginx/domain.crt;
        ssl_certificate_key /usr/local/nginx/domain.key;

        location / {
            # required for browsers to direct them to quic port
            add_header Alt-Svc 'h3=":443"; ma=86400';
        }
    }
}
```

## Make a dummy page

```
cd /var
mkdir -p www
cd www
mkdir -p html
cd html
echo "Hi" > index.html
```

## Run nginx!

Now you can run nginx and serve your website over HTTP/3:

```
LD_LIBRARY_PATH=/usr/local/lib/ /tmp/nginx-1.25.4/objs/nginx
```

## TODO

Make tutorial less hacky (LD library preload for LibreSSL, non standard nginx config dir etc.)

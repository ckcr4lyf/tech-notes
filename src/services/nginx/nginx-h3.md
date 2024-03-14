# Running HTTP/3 enabled nginx

For simiplicity, everything here will assume its being done in a root shell.

## Building

### Installing dependencies

```
apt-get update
apt-get install build-essential libpcre3 zlib1g libpcre3-dev 
```

### Build LibreSSL

```
cd /tmp
wget https://ftp.openbsd.org/pub/OpenBSD/LibreSSL/libressl-3.9.0.tar.gz
tar xvzf libressl-3.9.0.tar.gz
cd libressl-3.9.0
./configure
make check
make install

//cd ..
//mv libressl-3.9.0 libressl
```

### Building nginx

```
cd /tmp
wget http://nginx.org/download/nginx-1.25.4.tar.gz
tar xvzf nginx-1.25.4.tar.gz
cd nginx-1.25.4
./configure \
    --with-debug \
    --with-http_v3_module \
    --with-cc-opt="-I../libressl/build/include" \
    --with-ld-opt="-L../libressl/build/lib"
make
```

### Making the config dirs

```
cd /usr/local
mkdir nginx
cd nginx
mkdir logs
mkdir conf
```

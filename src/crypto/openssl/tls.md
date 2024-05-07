# Generating TLS certificates

## RSA

First, we generate and RSA key:

```
openssl genrsa -out example.com.key 4096
```

Generate the certificate signing request:

```
openssl req -key example.com.key -new -out example.com.csr
```

Self-signing the certificate:

```
openssl x509 -signkey example.com.key -in example.com.csr -req -days 3650 -out example.com.crt
```

## Elliptic Curve

Basically the same, but we will generate an Ed25519 key instead

```
openssl genpkey -algorithm ed25519 -out example.com.key
```

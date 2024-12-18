# HTTP(S) Proxy

Running an HTTP/S proxy is a useful way to try and achieve a couple of things:

* Hide the original IP address you're making the reqyest from (from the target website)
* Hide the intended website you're connecting to (from ISPs, censors) - only applicable to HTTPS proxies

Note: HTTP(S) proxies work best for HTTP(S) requests, though by using `CONNECT` it is possible to relay arbitrary TCP traffic as well.

However, proxies are typically configured at the application level. Even when you set an operating system level proxy (e.g. in Android), it is not necessary that an application respects it. **If you need all your internet traffic routed securely, you should use a VPN.**

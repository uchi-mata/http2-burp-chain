# Proxying HTTP2 Through Burp Suite

This repository contains a dockerized setup to proxy HTTP2 traffic through Burp Suite (let's hope that feature will be added soon and this won't be necessary anymore 🙂).

This setup was developed for a specific use case and could be way more generic, so please let me know if you come across any issues!

## Architecture

The setup is based on [NCC's article](https://www.nccgroup.trust/uk/about-us/newsroom-and-events/blogs/2018/may/testing-http2-only-web-services/) and dockerized for easier setup.

Traffic flows like this:
```
Intercepte ⇒ proxy2-1 ⇒ Burp Suite ⇒ proxy1-2 ⇒ Destination
```

* Interceptee is the application you want to proxy through Burp
* proxy2-1 translates HTTP2 to HTTP1 for Burp
* proxy1-2 translates Burp's HTTP1 traffic to HTTP2 for the destination system

## Setup

* Interceptee
    * Add the application you want to proxy to [interceptee/app/](interceptee/app/).
    * Add your build instructions to [interceptee/app/build.sh](interceptee/app/build.sh).
    * Add your app startup code to [interceptee/app/run.sh](interceptee/app/run.sh).
* proxy2-1/proxy1-2
    * The basic proxy configuration is included.
    * You have to add your custom TLS configuration files to the corresponding folders.
    * Change $BURPIP in [proxy2-1](etc-nghttpx-2-1/nghttpx.conf) to the IP your Burp Proxy is listening on.
    * Change hostnames to intercept in [proxy1-2](etc-nghttpx-1-2/nghttpx.conf).
* Burp must be configured as a transparent proxy that forwards all traffic to `proxy1-2`
* Destination
    * Add the hostnames you want to intercept to [docker-compose.yml](docker-compose.yml)
* Build containers
    * `cd proxy; docker build -t nghttpx .; cd ..`
    * `docker-compose build`
* Run `docker-compose up`

## Known Issues

* Some GRPC clients that make use of (HTTP2-specific) headers are currently not working, I will see whether nghttpx can be configured to pass those along somehow. 

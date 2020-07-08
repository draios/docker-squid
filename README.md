# docker-squid

Currently the only dockerfile available on docker hub for the squid HTTP proxy is https://hub.docker.com/r/datadog/squid, maintained by Datadog, stuck on a pretty old version, and (most problematically) built without SSL support.

This dockerfile will pull in the latest version of squid in the Alpine Linux repository, which is built with SSL support.

# Building

```
docker build -t localsquid --no-cache .
```

# Configuring

```
nathan@nathanb-dev ~/tmp % cat squid.conf
http_port 3128
https_port 3129 cert=/etc/squid/certificate.pem key=/etc/squid/key.pem

ssl_bump peek all
ssl_bump bump all

dns_v4_first on

acl localnet src 10.0.0.0/8     # RFC1918 possible internal network
acl localnet src 172.16.0.0/12  # RFC1918 possible internal network
acl localnet src 192.168.0.0/16 # RFC1918 possible internal network
acl localnet src fc00::/7       # RFC 4193 local private network range
acl localnet src fe80::/10      # RFC 4291 link-local (directly plugged) machines

acl SSL_ports port 443
acl SSL_ports port 6443

acl CONNECT method CONNECT

http_access allow localnet
http_access allow localhost

coredump_dir /squid/var/cache/squid
```

# Running

```
docker run --rm -it --name squid -p 3128:3128 -p 3129:3129 -v /home/nathan/tmp/squid.conf:/etc/squid/squid.conf -v /home/nathan/tmp/squidlogs:/var/log/squid -v /home/nathan/tmp/localhost.pem:/etc/squid/certificate.pem -v /home/nathan/tmp/localhost.key:/etc/squid/key.pem localsquid:latest
```

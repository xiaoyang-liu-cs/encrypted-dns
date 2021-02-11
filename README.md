# Encrypted-DNS

[![License](https://img.shields.io/github/license/Berkeley-Reject/Encrypted-DNS.svg?style=for-the-badge)](https://github.com/Berkeley-Reject/Encrypted-DNS/blob/master/LICENSE)
[![Releases](https://img.shields.io/github/v/release/Berkeley-Reject/Encrypted-DNS?style=for-the-badge)](https://github.com/Berkeley-Reject/Encrypted-DNS/releases)
[![Downloads](https://img.shields.io/pypi/dm/encrypted-dns?style=for-the-badge)](https://pypistats.org/packages/encrypted-dns)

[Issues](https://github.com/Berkeley-Reject/Encrypted-DNS/issues) |
[Pull requests](https://github.com/Berkeley-Reject/Encrypted-DNS/pulls) | 
[Contributors](https://github.com/Berkeley-Reject/Encrypted-DNS/graphs/contributors)

## Introduction

Encrypted-DNS operates as a DNS server that forward DNS queries over UDP, TCP, TLS or HTTPS, thus preventing your device from DNS cache poisoning and censorship.
It could also cache DNS records to accelerate further queries, block specific client, and ignore particular domain names.

### Features

* Encrypted DNS upstream servers (DNS-over-HTTPS, DNS-over-TLS)
* Improve accuracy with EDNS-Client-Subnet
* Authenticate DNS response with DNSSEC
* Transparent redirection of specific domains to specific resolvers
* Send queries through HTTP proxies
* Cache DNS response with default or customized TTL to reduce latency
* Force Safe search on search engines such as Google, Bing, DuckDuckGo
* Firewall rules: Rate limiting, client blacklist, and disable AAAA or ANY lookups

## Installation

* Install [Python](https://www.python.org/downloads/) 3.6+

* Install `encrypted-dns` package via `pip`

```
$ python3 -m pip install encrypted-dns
```

* Generate and edit config file

```
$ sudo encrypted-dns
$ vim ~/.config/encrypted_dns/config.json
```

* Run Encrypted-DNS Server

```
$ sudo encrypted-dns
```

* Test DNS Lookup

```
Linux or MacOS:
$ dig @127.0.0.1 www.google.com

Windows:
$ nslookup www.google.com 127.0.0.1
```

* Change DNS Address to `127.0.0.1`

## Configuration

Encrypted-DNS will generate a JSON file `~/.config/encrypted_dns/config.json`

### Inbounds

Encrypted-DNS will listen on the address and ports to receive DNS lookups.

The format of each inbound is `protocol://address:port`.

Currently, Encrypted-DNS only supports inbounds with `udp` and `tcp` protocols.

If `protocol` is not provided, Encrypted-DNS will listen to this inbound address through the `udp` protocol.

If `port` is not provided, Encrypted-DNS will use the default port of each protocol. (`53` for `udp` and `tcp`)

```
'inbounds': [
                '0.0.0.0',
                '0.0.0.0:5301',
                'tcp://0.0.0.0:5302'
            ]
```

### Outbounds

Encrypted-DNS will forward the DNS quires to the upstream DNS servers.

The `Outbounds` is a JSON array of DNS groups.


Here is an example of a DNS group:

```
'outbounds': [
    {
        'tag': 'unencrypted',
        'dns': ['1.0.0.1', 'tcp://8.8.4.4'],
        'concurrent': False,
        'domains': ['time.windows.com', sub:youtube.com', 'include:netflix.com']
    },
    {
        'tag': 'encrypted',
        'dns': ['https://cloudflare-dns.com', 'tls://dns.google'],
        'proxies': {
            'http': 'http://127.0.0.1:1088',
            'https': 'http://127.0.0.1:1088'
        },
        'concurrent': False,
        'domains': ['all']
    }
]
```

`tag` is the name of the DNS group

`dns` is an array of DNS upstreams
* The format of each upstream is `protocol://address:port`
* Encrypted-DNS supports these protocols: `udp`, `tcp`, `tls`, `https`
* If `protocol` is not provided, Encrypted-DNS will connect to the upstream through `udp` protocol.
* If `port` is not provided, Encrypted-DNS will use the default port of each protocol. (`53` for `udp` and `tcp`, `853` for `tls`, `443` for `https`)

`concurrent`
* If `concurrent` is `True`, Encrypted-DNS will forward queries to all servers in this group concurrently and send the first response to the client 
* If `concurrent` is `False`, Encrypted-DNS will forward queries to a random server in this group. 

Encrypted-DNS will only forward queries to this group only if the domain is included in the `domains`. For example:
* `www.google.com`: exact domain
* `sub:youtube.com`: subdomains of `youtube.com`, such as `m.youtube.com`, `www.youtube.com`
* `include:netflix.com`: domains include `netflix.com`, such as `www.netflix.com`, `netflix.com.example.com`, `whatisnetflix.command`
* `all`: all domains

### Bootstrap DNS Group

Encrypted-DNS will send DNS queries to the server in the `bootstrap` DNS group to retrieve the IP addresses of DNS-over-HTTPS or DNS-over-TLS server.

If the group is not specified, Encrypted-DNS will use `1.0.0.1` to resolve the IP addresses.

```
'outbounds': [
    {
        'tag': 'bootstrap',
        'dns': ['1.0.0.1', '8.8.4.4']
    },
    ...
]
```

### DNS Cache

Cache DNS responses to reduce latency for further queries.

If `override_ttl` is `-1`, Encrypted-DNS will use default TTL for each record.

```
'dns_cache': {
    'enable': True,
    'override_ttl': 3600
}
```

### Firewall

* `refuse_ANY` will ignore all queries with `ANY` type since it's often used in DNS reflection attacks.
* `AAAA_disabled` will ignore all quires with `AAAA` type.
* `rate_limit` will limit the amount of quires Encrypted-DNS could process every minute.
* `client_blacklist` will ignore all quires sent by specific clients.

```
'firewall': {
    'refuse_ANY': True,
    'AAAA_disabled': False,
    'rate_limit': 30,
    'client_blacklist': [
        '128.97.0.0',
    `   '128.97.0.1'
    ]
}

```

### Rules

* `force_safe_search` will enable Safe search on search engines: Google, Bing, Yahoo, DuckDuckGo, and Youtube.
* `hosts` will specify A record or CNAME record for domain names.

Rules to match domain in `hosts`:
* `www.google.com`: exact domain
* `sub:youtube.com`: subdomains of `youtube.com`, such as `m.youtube.com`, `www.youtube.com`
* `include:netflix.com`: domains include `netflix.com`, such as `www.netflix.com`, `netflix.com.example.com`, `whatisnetflix.command`
* `all`: all domains

```
'rules': {
    'force_safe_search': False,
    'hosts': {
        'localhost': '127.0.0.1',
        'sub:cloudflare-dns.com': '1.0.0.1',
        'dns.google': '8.8.4.4'
    }
},
```

### EDNS Client Subnet

EDNS Client Subnet is a DNS extension that allows Encrypted-DNS to specify the network subnet for the host on which behalf it is making a DNS query.

This is generally intended to help speed up the delivery of data from CDN, by allowing better use of DNS-based load balancing to select a service address serving the content expected to be hosted at that domain name, when the client computer is in a different network location from the recursive resolver.

To disable this feature, set `ecs_ip_address` to `null`.

```
'ecs_ip_address': '128.97.0.0'
```


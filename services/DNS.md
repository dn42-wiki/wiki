# DN42 DNS

This page covers guidance and examples on using DNS within DN42.

## Quick Start

It is recommended to run your own DNS resolver as this provides you with the most security and privacy. 
However, to get started, or if running your own resolver isn't desirable an anycast service
is available. The anycast service supports DNSSEC and will resolve public DNS names together with all the 
relevant DN42 and affiliated networks' names.

### Using the DNS Anycast Service

The DNS anycast service is provided by multiple operators, with each operator contributing to one of the two separate 
anycast services. By configuring both services, users get additional resiliency from having two, independent, resolvers. 

| Name | IPv4 | IPv6 |
|---|---|---|
| a0.recursive-servers.dn42 | 172.20.0.53 | fd42:d42:d42:54::1 |
| a3.recursive-servers.dn42 | 172.23.0.53 | fd42:d42:d42:53::1 |

To configure the service, ping both sets of addresses then set your primary nameserver to the lowest latency 
service and configure the other service as the secondary or backup nameserver.

Example resolv.conf, preferring a0.recursive-servers.dn42 and IPv4:

```text
nameserver 172.20.0.53
nameserver 172.23.0.53
nameserver fd42:d42:d42:54::1
nameserver fd42:d42:d42:53::1
search dn42
```

Example resolv.conf, preferring a3.recursive-servers.dn42 and IPv6:

```text
nameserver fd42:d42:d42:53::1
nameserver fd42:d42:d42:54::1
nameserver 172.23.0.53
nameserver 172.20.0.53
option inet6       # Linux/glibc
family inet6 inet4 # BSD
search dn42
```

## Advanced Configuration

There are multiple top level domains (TLDs) associated with DN42, its affiliated networks and for reverse DNS that must
be configured in order to run your own resolver. The registry is the authoritative source of active TLDs, but see also
this page [[dns/External-DNS|/services/dns/External-DNS]] in the wiki.

### Split horizon DNS

In this configuration, you run your own, caching resolver but forward DN42 related queries (with recursion bit set) 
to the anycast service. Example configurations for different recursor implementations are included in the [[dns/Configuration|/services/dns/Configuration]] page.

### Full recursion 

Authoritative DNS for DN42 is provided by the *.delegation-servers.dn42 servers, see the DNS architecture here 
[[New DNS|New-DNS]] Delegations servers have full support for DNSSEC. 

## Further Information

* [[dns/Configuration|/services/dns/Configuration]] - Forwarder configuration examples
* [[New DNS|New-DNS]] - current architecture
* [[dns/External-DNS|/services/dns/External-DNS]] - external DNS zones from interconnected networks
* [[Old Hierarchical DNS|Old-Hierarchical-DNS]] - deprecated
* [[Original DNS (deprecated)|Original-DNS-(deprecated)]] - deprecated

# DN42 DNS Architecture

Simple setup for new users:

All I want is to access an .dn42 domain. You can configure:
* b.recursive-servers.dn42. fd42:4242:2601:ac53::53, 172.20.129.2
* j.recursive-servers.dn42. fd42:5d71:219:1:69c2:2b0e:17e8:c215, 172.20.1.255

Please note that, these server may not resolve clearnet queries. Please check [[New DNS]] - for all the details about the current servers.

It is recommened that you setup your own resolver, please check [[dns/Configuration|Configuration]] - DN42 DNS forward configuration for BIND, dnsmasq, Unbound, PowerDNS, etc.

* [[New DNS]] - current architecture
* [[Old Hierarchical DNS]] - deprecated
* [[Original DNS (deprecated)]] - deprecated
* [[dns/External-DNS|External-DNS]] - external DNS zones from interconnected networks

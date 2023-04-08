[Tor](https://torproject.org/) ([dn42 mirror](http://tor.e-utp.dn42/)) is free software and an open network that helps you defend against traffic analysis, a form of network surveillance that threatens personal freedom and privacy, confidential business activities and relationships, and state security.

# Tor Bridges

Tor bridges allow for the Tor client to connect to a specific IP address and validate the fingerprint to permit safe connections into the Tor network for networks that do not directly access the public Internet, for example hosts that only have dn42 connections. The following bridges are currently available for use:

| Name                  | Bandwidth | Contact          | Protocol | Fingerprint                              | Info                               |
|-----------------------|-----------|------------------|----------|------------------------------------------|------------------------------------|
| photon.flat.dn42:8443 | 500kB/s   | irl@flat.dn42    | obfs4    | 83B02FB88253A7FD313B7912B12B05AF2A42D3B9 | Limited to 100GB transfer per week |
| gouda.flat.dn42:8443  | 500kB/s   | irl@flat.dn42    | obfs4    | DF8CA08A9BED62B319D1E52610510959374444A2 |                                    |
| tor.napshome.dn42:8443  | 3000KB/s+   | bjackson@napshome.net    | obfs4    | 71C924A772F69451FE97FE5A9025DEDDEF3DB664 |                                    |
| tor.napshome.dn42:9001  | 3000KB/s+   | bjackson@napshome.net    | plain    | 71C924A772F69451FE97FE5A9025DEDDEF3DB664 |                                    |

# Anycast Tor

There is an anycast address, 172.22.0.94 and fd42:d42:d42:9001::1 aka tor.dn42, that provides the following services:

| Service | Port     |
|---------|----------|
| ORPort  | 9001/tcp |
| DirPort | 9030/tcp |
| SOCKS   | 9050/tcp |

It should be noted that the host providing the SOCKS services is able to see *all* of the traffic going through it, you will not be protected for the portion of the journey between your machine and the Tor service. There is also no guarantee that the SOCKS proxy even provides a Tor service, it may simply just be a SOCKS proxy and you have no means by which to verify it.

There is also unfortunately no means by which to tell Tor to use a specific IP address as the entry point for a plain relay and while in theory you could connect to the ORPort and have a safe connection to Tor, there is no configuration option available for this.

# Tor SOCKS Proxies

_Note that the same warnings above also apply to the following proxies._

| Proxy URL                             | Bandwidth   | Contact     |
|---------------------------------------|-------------|-------------|
| socks5://tor.napshome.dn42:9050       | 100+ Mbit/s | Napsterbater|

| Offline                               |             |             |
|---------------------------------------|-------------|-------------|
| socks5://172.20.11.33:9050            | 100 Mbit/s  | twink0r     |


# DNS Proxy - Tor Hidden Services

_Note that the same warnings above also apply to the following proxies._

See <https://blog.benjojo.co.uk/post/tor-onions-to-v6-with-iptables-proxy> for more details

| Suffix Change                         | Bandwidth   | Contact     |
|---------------------------------------|-------------|-------------|
| .onion -> .onion.dns42 (IPv6 Mapping)   | 1000 Mbit/s | TheQ        |
| .onion -> .oni.dns42 (IPv6 Mapping)   | 1000 Mbit/s | TheQ        |



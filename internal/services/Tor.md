[Tor](https://torproject.org/) ([dn42 mirror](http://tor.e-utp.dn42/)) is free software and an open network that helps you defend against traffic analysis, a form of network surveillance that threatens personal freedom and privacy, confidential business activities and relationships, and state security.

# Tor Bridges

Tor bridges allow for the Tor client to connect to a specific IP address and validate the fingerprint to permit safe connections into the Tor network for networks that do not directly access the public Internet, for example hosts that only have dn42 connections. The following bridges are currently available for use:

| Name                  | Bandwidth | Contact          | Protocol | Fingerprint                              | Info                               |
|-----------------------|-----------|------------------|----------|------------------------------------------|------------------------------------|
| photon.flat.dn42:8443 | 500kB/s   | irl@flat.dn42    | obfs4    | 79B30C78C9DA0F812589D336B399307435DC452A | Limited to 100GB transfer per week |

# Anycast Tor

There is an anycast address, 172.22.0.94 aka tor.dn42, that provides the following services:

| Service | Port     |
|---------|----------|
| ORPort  | 9001/tcp |
| DirPort | 9030/tcp |
| SOCKS   | 9050/tcp |

It should be noted that the host providing the SOCKS services is able to see *all* of the traffic going through it, you will not be protected for the portion of the journey between your machine and the Tor service. There is also no guarantee that the SOCKS proxy even provides a Tor service, it may simply just be a SOCKS proxy and you have no means by which to verify it.

There is also unfortunately no means by which to tell Tor to use a specific IP address as the entry point for a plain relay and while in theory you could connect to the ORPort and have a safe connection to Tor, there is no configuration option available for this.

# Tor SOCKS Proxies

_Note that the same warnings above also apply to the following proxies._

| Proxy URL                             | Bandwidth   | Contact     | Fingerprint                      |
|---------------------------------------|-------------|-------------|----------------------------------|
| socks5://172.20.11.33:9050            | 100 Mbit/s  | twink0r     | ?                                |
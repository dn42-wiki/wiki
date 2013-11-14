# DNS

*(tl;dr)* We have a TLD for dn42, which is `.dn42`. The anycast resolver for `.dn42` runs on `172.22.0.53`.
**DNS is build from [[whois database|Services Whois]]. So please edit your DNS-records there.**

## Using the DNS service

Below are several ways to use the `dn42` DNS service, from easiest to more challenging. The recommended method is the second one.

### Using the anycast resolver directly

Please be aware that this method sends **all** your DNS queries (e.g. `google.com`) to a random DNS server inside dn42. The server could fake the result and point you towards the russian mafia. They probably won't, but think about what you are doing. At the end of the day, your ISP could be evil as well, so it always boils down to a question of trust.

To do this, just use `172.22.0.53` as your resolver, for instance in `/etc/resolv.conf`.

### Forwarding `.dn42` queries to the anycast resolver

If you run your own resolver (`unbound`, `dnsmasq`, `bind`), you can configure it to forward dn42 queries to the anycast DNS resolver. See [[DNS forwarder configuration|Services DNS Configuration]].

### Recursive resolver

You may also want to configure your resolver to recursively resolve dn42 domains. For this, you need to find authoritative DNS servers for the `dn42` zone (and for the reverse zones). See [[Recursive DNS resolver]].

### Building the dn42 zones from the registry

Finally, you may want to host your own authoritative DNS server for the `dn42` zone and the reverse zones. The zone files are built from the monotone repository: scripts are provided in the repository itself.

## Register a `.dn42` domain name

The root zone for `dn42.` is built from the [[whois registry|Services Whois]]. If you want to register a domain name, you need to add it to the registry (of course, you also need one or two authoritative nameservers).

## DNS services for other networks

Other networks are interconnected with dn42 (ChaosVPN, Freifunk, etc). Some of them also provide DNS service, you can configure your resolver to use it. See [[External DNS]].

## Providing DNS service

See [[Providing Anycast DNS]].
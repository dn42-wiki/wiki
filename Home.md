## About dn42

dn42 is a large, dynamic [VPN](https://en.wikipedia.org/wiki/Virtual_private_network) that employs Internet technologies ([BGP](https://en.wikipedia.org/wiki/Bgp), whois database, [DNS](https://en.wikipedia.org/wiki/Domain_Name_System), etc.). Participants connect to each other using network tunnels ([GRE](/howto/GRE-on-FreeBSD), [OpenVPN](/howto/openvpn), [WireGuard](/howto/wireguard), [Tinc](/howto/tinc), [IPsec](/howto/IPsec-with-PublicKeys)) and exchange routes using the Border Gateway Protocol.

Network addresses are assigned in the `172.20.0.0/14` range with private AS numbers (see [registry](/services/Whois)), as well as IPv6 addresses from the ULA range (`fd00::/8`) - see [FAQ](/FAQ#frequently-asked-questions_what-about-ipv6-in-dn42).

A variety of [services](/internal/Internal-Services) are available on the network, only accessible from within dn42. dn42 is also interconnected with other networks, such as [ChaosVPN](http://wiki.hamburg.ccc.de/ChaosVPN) and various [Freifunk](https://en.wikipedia.org/wiki/Freifunk) networks.

Still have questions? Check out our [FAQs](/FAQ).

## Why dn42?

dn42 can be used to learn networking and to connect private networks, such as hackerspaces or community networks. But above all, experimenting with routing in dn42 is fun!

### Experiment with routing technology

dn42 is primarily useful for learning routing technologies such as BGP within a reasonably large network (1,500+ AS, 1,700+ prefixes).

Since dn42 closely mirrors the Internet, it serves as a hands-on testing ground for new ideas, or simply for learning real networking concepts that aren't practical on the public Internet (BGP multihoming, transit, etc.). The biggest advantage: if you break something, no large network operator will be yelling at you.

### Connect hackerspaces and private networks

dn42 provides a way to connect hackerspaces and other private networks, enabling them to share services with each other.

Most end-user networks rely on [NAT](https://en.wikipedia.org/wiki/Network_address_translation) to share a single public IPv4 address among multiple devices, making it difficult to host services directly. You may also want to offer services to other hackerspaces without exposing them to the entire Internet.

dn42 solves this problem. By addressing your network within dn42, your devices can communicate transparently with all other participants - no NAT required. You still maintain full control: like with IPv6, you can use a firewall to restrict access while selectively allowing traffic from dn42.

If your hackerspace uses dn42 to provide services, please let us know on this wiki or the mailing list. It's rewarding to see the network put to practical use!

## Join or contact us

dn42 is operated by volunteers with no central authority. Visit the [contact](/contact) page to learn how to collaborate or get in touch.

Ready to join? The [Getting Started](/howto/Getting-Started) guide will help you set up your first node.

## External resources

- [Wikipedia: Decentralized network 42](https://en.wikipedia.org/wiki/Decentralized_network_42)
- [26c3 lecture](https://fahrplan.events.ccc.de/congress/2009/Fahrplan/events/3504.en.html)
- [GPN8 lecture](https://entropia.de/GPN8:dn42)
- [nobody's dn42 guide](http://nowhere.ws/guides/dn42/)
- [mrmcd0x8 lecture](https://web.archive.org/web/20090831211324/http://mrmcd0x8.metarheinmain.de/fahrplan/events/3321.de.html)
- [dn42 on hackerspaces.org](https://wiki.hackerspaces.org/Category:DN42)
- [pentaradio24 podcast (German)](https://www.c3d2.de/news/pentaradio24-20150428.html)
- [dn42 in your browser](http://sandbox.freertr.org/)
- [dn42 in your terminal](http://portable.freertr.org/)

## Participant groups

- [SpaceBoyz](http://spaceboyz.net)
- [CCC Aachen](https://aachen.ccc.de)
- [CCC Bremen](http://ccchb.de)
- [CCC Darmstadt](http://darmstadt.ccc.de)
- [CCC Dresden](http://c3d2.de)
- [CCC Düsseldorf](https://www.chaosdorf.de)
- [CCC Munich](https://www.muc.ccc.de)
- [Chaostreff Chemnitz](https://chaoschemnitz.de)
- [/dev/nulll](https://dev.0l.de)
- [freifunk](http://freifunk.net)
- [NoName e.V. Heidelberg](https://www.noname-ev.de)
- [raumzeitlabor / hackerspace rhein-neckar](http://www.raumzeitlabor.de)
- [Hackerspace Brussels (HSB)](http://hackerspace.be)
- [hsmr / Hackspace Marburg](https://hsmr.cc)
- [Whitespace (0x20)](http://www.0x20.be)
- [Revelation Space](http://www.revspace.nl)
- [SNE group](https://www.os3.nl)
- [smrsh](http://www.smrsh.net)
- [Breizh-Entropy](http://wiki.breizh-entropy.org/wiki/DN42)
- [Fédération FDN](https://www.ffdn.org)
- [Le LOOP](https://leloop.org/)
- [ACME Labs](https://acmelabs.space)
- [fixmix Technologies Ltd](https://dn42.fixmix.tech/)
- [Strategic Explorations Ltd](https://strexp.net)
- [perchnet](/perchnet) (VPS donated by [Evolution Host](https://evolution-host.com))

## About this wiki

This wiki is the main reference for dn42. It is available in read-only mode from the Internet at:

- [wiki.dn42.us](https://wiki.dn42.us)
- [dn42.dev](https://dn42.dev)
- [dn42.pp.ua](https://dn42.pp.ua)
- [dn42.eu](https://dn42.eu)
- [dn42.wiki](https://dn42.wiki)
- [dn42.cc](https://dn42.cc)
- [dn42.de](https://dn42.de) (IPv6 only)
- [dn42.jp](https://dn42.jp)

Editing is available from within dn42 at <https://wiki.dn42> ([HTTPS certificate](/services/Certificate-Authority) required).

### DN42 logo

An SVG of the DN42 logo is available [here](/dn42.svg).
## About dn42

dn42 is a big dynamic [VPN](http://en.wikipedia.org/wiki/Virtual_private_network), which employs Internet technologies ([BGP](http://en.wikipedia.org/wiki/Bgp), whois database, [DNS](http://en.wikipedia.org/wiki/Domain_Name_System), etc).  Participants connect to each other using network tunnels ([GRE](http://en.wikipedia.org/wiki/Generic_Routing_Encapsulation), [OpenVPN](http://en.wikipedia.org/wiki/Openvpn), [Tinc](http://tinc-vpn.org/), [IPsec](http://en.wikipedia.org/wiki/IPsec)) and exchange routes thanks to the Border Gateway Protocol.  Network addresses are assigned in the `172.22.0.0/15` and `172.20.0.0/16` range and private AS numbers are used (see [registry](/services/Whois)) as well as IPv6 addresses from the ULA-Range (`fd00::/8`) - see [FAQ](https://internal.dn42/FAQ#frequently-asked-questions_what-about-ipv6-in-dn42).

A number of services are provided on the network: see [internal](http://wiki.dn42/internal/Internal-Services) (only available from within dn42).  Also, dn42 is interconnected with other networks, such as [ChaosVPN](http://wiki.hamburg.ccc.de/ChaosVPN) or some [Freifunk](http://en.wikipedia.org/wiki/Freifunk) networks.

Still have questions? We have  [[FAQs|FAQ]] listed.

## Why dn42?

dn42 can be used to learn networking and to connect private networks, such as hackerspaces or community networks. But above all, experimenting with routing in dn42 is fun!

### Experiment with routing technology

Participating in dn42 is primarily useful for learning routing technologies such as BGP, using a reasonably large network (~200 AS, ~400 prefixes).

Since dn42 is very similar to the Internet, it can be used as a hands-on testing ground for new ideas, or simply to learn real networking stuff that you probably can't do on the Internet (BGP multihoming, transit).  The biggest advantage when compared to the Internet: if you break something in the network, you won't have any big network operator yelling angrily at you.

### Connect hackerspaces

dn42 is also a great way to connect hacker spaces in a secure way, so that they can provide services to each other.

Have you ever wanted to SSH on your Raspberry Pi hosted at your local hacker space and had trouble doing so because of NAT? If your hacker space was using dn42, it could have been much easier.

Nowadays, most end-user networks use [NAT](http://en.wikipedia.org/wiki/Network_address_translation) to squeeze all those nifty computing devices behind a single public IPv4 address.  This makes it difficult to provide services directly from a machine behind the NAT.  Besides, you might want to provide some services to other hackerspaces, but not to anybody on the Internet.

dn42 solves this problem.  By addressing your network in dn42, your devices can communicate with all other participants in a transparent way, without resorting to this ugly thing called NAT.  Of course, this doesn't mean that you have to fully open your network to dn42: similarly to IPv6, you can still use a firewall (but you could, for instance, allow incoming TCP 22 and TCP 80 from dn42 by default).

If your hackerspace is actually using dn42 to provide some services, please let us know! (on this wiki or on the mailing list). It's very rewarding when the network is actually used for something :)

## Join or Contact us

dn42 is operated by a group of volunteers. There is no central authority which controls or impersionate the network. Take a look at the [[contact]] page to see how to collaborate or contact us.

The [[Getting started]] page helps you to get your first node inside the network.

## External resources about dn42

 * [Wikipedia about dn42](http://en.wikipedia.org/wiki/Decentralized_network_42)
 * [Lecture on 26c3](http://events.ccc.de/congress/2009/Fahrplan/events/3504.en.html)
 * [Lecture on GPN8](http://entropia.de/wiki/GPN8:dn42)
 * [soup.io group](http://dn42.soup.io/)
 * [nobody about dn42](http://nowhere.ws/guides/dn42/)
 * [Lecture on mrmcd0x8](http://mrmcd0x8.metarheinmain.de/fahrplan/events/3321.de.html)
 * [dn42-category in hackerspaces.org wiki](https://hackerspaces.org/wiki/Category:DN42)
 * [pentaradio24 – german podcast](https://www.c3d2.de/news/pentaradio24-20150428.html)

## Participant Groups

* [SpaceBoyz](http://spaceboyz.net)
* [CCC Bremen](http://ccchb.de)
* [CCC Darmstadt](http://darmstadt.ccc.de)
* [CCC Dresden](http://c3d2.de)
* [CCC Düsseldorf](https://www.chaosdorf.de)
* [CCC Munich](https://www.muc.ccc.de)
* [/dev/nulll](https://dev.0l.de)
* [freifunk](http://freifunk.net)
* [NoName e.V. Heidelberg](https://www.noname-ev.de)
* [raumzeitlabor/hackerspace rhein-neckar](http://www.raumzeitlabor.de)
* [Cyberpipe](https://www.kiberpipa.org)
* [Hackerspace Brussels (HSB)](http://hackerspace.be)
* [[hsmr] / Hackspace Marburg](https://hsmr.cc)
* [Whitespace (0x20)](http://www.0x20.be)
* [Revelation Space](http://www.revspace.nl)
* [SNE group](https://www.os3.nl)
* [smrsh](http://www.smrsh.net)
* [Hackspace Jena e.V.](https://kraut.space)
* [breizh-entropy](http://breizh-entropy.dn42)
* [Fédération FDN](https://www.ffdn.org)
* [Le LOOP](https://leloop.org/)
* [Hackerspace Bielefeld](https://hackerspace-bielefeld.de)
* [NixNodes](https://nixnodes.net)

## About this wiki

This wiki is the main reference about dn42.  It is available in read-only mode [from the Internet](https://dn42.net), [tor](http://jsptropkiix3ki5u.onion) and [i2p](http://beb6v2i4jevo72vvnx6segsk4zv3pu3prbwcfuta3bzrcv7boy2q.b32.i2p/) and for editing from within dn42, at [https://wiki.dn42](https://wiki.dn42) - [https](services/Certificate-Authority) required for editing.

A [copy of the old wiki](http://dn42.volcanis.me/initenv.1.html) is available for reference, but **beware**, most of the information there is outdated.
## About dn42

dn42 is a big dynamic VPN network, which employs Internet technologies ([BGP](http://en.wikipedia.org/wiki/Bgp), whois database, DNS, etc).  Participants connect to each other using network tunnels ([GRE](http://en.wikipedia.org/wiki/Generic_Routing_Encapsulation), [OpenVPN](http://en.wikipedia.org/wiki/Openvpn), [Tinc](http://tinc-vpn.org/), IPsec), and exchange routes thanks to the Border Gateway Protocol.  Network addresses are assigned in the 172.22.0.0/15 range, and private AS numbers are used: see [registry](Services-Whois).

A number of services are provided on the network: see [internal](http://internal.dn42) (only available from within dn42).  Also, dn42 is interconnected with other networks, such as [ChaosVPN](http://wiki.hamburg.ccc.de/ChaosVPN) or some Freifunk networks.

Still have questions? We have a [FAQ].

## Why dn42?

dn42 can be used to learn networking and to connect private networks, such as hackerspaces or community networks. But above all, experimenting with routing in dn42 is fun!

### Experiment with routing technology

Participating in dn42 is primarily useful for learning routing technologies such as BGP, using a reasonably large network (~100 AS, ~300 prefixes).

Since dn42 is very similar to the Internet, it can be used as a hands-on testing ground for new ideas, or simply to learn real networking stuff that you probably can't do on the Internet (BGP multihoming, transit).  The biggest advantage when compared to the Internet: if you break something in the network, you won't have any big network operator yelling angrily at you.

### Connect hackerspaces

dn42 is also a great way to connect hackerspaces in a secure way, so that they can provide services to each other.

Have you ever wanted to SSH on your raspberrypi hosted at your local hackerspace, and had trouble doing so because of NAT? If your hackerspace was using dn42, it could have been much easier.

Nowadays, most end-user networks use NAT to squeeze all those nifty computing devices behind a single public IPv4 address.  This makes it difficult to provide services directly from a machine behind the NAT.  Besides, you might want to provide some services to other hackerspaces, but not to anybody on the Internet.

dn42 solves this problem.  By addressing your network in dn42, your devices can communicate with all other participants in a transparent way, without resorting to this ugly thing called NAT.  Of course, this doesn't mean that you have to fully open your network to dn42: similarly to IPv6, you can still use a firewall (but you could, for instance, allow incoming TCP 22 and TCP 80 from dn42 by default).

If your hackerspace is actually using dn42 to provide some services, please let us know! (on this wiki or on the mailing list). It's very rewarding when the network is actually used for something :)


## About this wiki

This wiki is the main reference about dn42.  It is available in read-only mode [from the Internet](https://dn42.net), and for editing from within dn42, at [http://internal.dn42](http://internal.dn42).

A [copy of the old wiki](http://dn42.volcanis.me/initenv.1.html) is available for reference, but **beware**, most of the information there is outdated.

## Contact

### Mailing list

* is at: dn42 /at/ lists.spaceboyz.net,
* send a mail to dn42-subscribe /at/ lists.spaceboyz.net to subscribe
* has [archive](http://lists.spaceboyz.net/pipermail/dn42/)
* can be accessed via [NNTP](/Services-News) as well. The group is alt.net.dn42.users

### IRC

* #dn42 in [hackint](http://www.hackint.eu/)

## Network
### Looking glass

See [[Looking Glasses]].

### Network maps
* [net.smrsh.dn42/routes/d3js.html](http://net.smrsh.dn42/routes/d3js.html) aka 172.23.174.1 (dn42) or [dn42.smrsh.net/routes/d3js.html](http://dn42.smrsh.net/routes/d3js.html) (Internet)
* [map.nixnodes.net](http://map.nixnodes.net)

The data for these maps is collected using AS paths from various AS.

### Tools

[Munin](/Munin)

## External resources about dn42

 * [Wikipedia about dn42](http://en.wikipedia.org/wiki/Decentralized_network_42)
 * [Lecture on 26c3](http://events.ccc.de/congress/2009/Fahrplan/events/3504.en.html)
 * [Lecture on GPN8](http://entropia.de/wiki/GPN8:dn42)
 * [soup.io group](http://dn42.soup.io/)
 * [nobody about dn42](http://nowhere.ws/guides/dn42/)
 * [Lecture on mrmcd0x8](http://mrmcd0x8.metarheinmain.de/fahrplan/events/3321.de.html)
 * [dn42-category in hackerspaces.org wiki](https://hackerspaces.org/wiki/Category:DN42)


## Participant Groups

* [SpaceBoyz](http://spaceboyz.net)
* [planet cyborg](http://planetcyborg.de)
  * [mw](http://mw.vc)
* [CCC Bremen](http://ccchb.de)
* [CCC Dresden](http://c3d2.de)
* [CCC Düsseldorf](https://www.chaosdorf.de)
* [CCC Munich](https://www.muc.ccc.de)
* [/dev/nulll](https://dev.0l.de)
* [freifunk](http://freifunk.net)
* [NoName e.V. Heidelberg](https://www.noname-ev.de)
* [raumzeitlabor/hackerspace rhein-neckar](http://www.raumzeitlabor.de)
* [Freifunk Aurich](http://www.freifunk-aurich.de)
* [Cyberpipe](https://www.kiberpipa.org)
* [Hackerspace Brussels (HSB)](http://hackerspace.be)
* [[hsmr] / Hackspace Marburg](https://hsmr.cc)
* [Whitespace (0x20)](http://www.0x20.be)
* [Revelation Space](http://www.revspace.nl)
* [SNE group](https://www.os3.nl)
* [smrsh](http://www.smrsh.net)
* [Hackspace Jena e.V.](https://www.hackspace-jena.de)
* [breizh-entropy](http://breizh-entropy.dn42)
* [Fédération FDN](https://www.ffdn.org)
* [Le LOOP](https://leloop.org/)
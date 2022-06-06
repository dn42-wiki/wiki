## About dn42

dn42 is a big dynamic [VPN](http://en.wikipedia.org/wiki/Virtual_private_network), which employs Internet technologies ([BGP](http://en.wikipedia.org/wiki/Bgp), whois database, [DNS](http://en.wikipedia.org/wiki/Domain_Name_System), etc).  Participants connect to each other using network tunnels ([GRE](/howto/GRE-on-FreeBSD), [OpenVPN](/howto/openvpn), [WireGuard](/howto/wireguard), [Tinc](/howto/tinc), [IPsec](/howto/IPsec-with-PublicKeys)) and exchange routes thanks to the Border Gateway Protocol.  Network addresses are assigned in the `172.20.0.0/14` range and private AS numbers are used (see [registry](/services/Whois)) as well as IPv6 addresses from the ULA-Range (`fd00::/8`) - see [FAQ](/FAQ#frequently-asked-questions_what-about-ipv6-in-dn42).

A number of services are provided on the network: see [internal](/internal/Internal-Services) (only available from within dn42).  Also, dn42 is interconnected with other networks, such as [ChaosVPN](http://wiki.hamburg.ccc.de/ChaosVPN) or some [Freifunk](http://en.wikipedia.org/wiki/Freifunk) networks.

Still have questions? We have  [FAQs](/FAQ) listed.

## Why dn42?

dn42 can be used to learn networking and to connect private networks, such as hackerspaces or community networks. But above all, experimenting with routing in dn42 is fun!

### Experiment with routing technology

Participating in dn42 is primarily useful for learning routing technologies such as BGP, using a reasonably large network (> 1500 AS, > 1700 prefixes).

Since dn42 is very similar to the Internet, it can be used as a hands-on testing ground for new ideas, or simply to learn real networking stuff that you probably can't do on the Internet (BGP multihoming, transit).  The biggest advantage when compared to the Internet: if you break something in the network, you won't have any big network operator yelling angrily at you.

### Connect hackerspaces

dn42 is also a great way to connect hacker spaces in a secure way, so that they can provide services to each other.

Have you ever wanted to SSH on your Raspberry Pi hosted at your local hacker space and had trouble doing so because of NAT? If your hacker space was using dn42, it could have been much easier.

Nowadays, most end-user networks use [NAT](http://en.wikipedia.org/wiki/Network_address_translation) to squeeze all those nifty computing devices behind a single public IPv4 address.  This makes it difficult to provide services directly from a machine behind the NAT.  Besides, you might want to provide some services to other hackerspaces, but not to anybody on the Internet.

dn42 solves this problem.  By addressing your network in dn42, your devices can communicate with all other participants in a transparent way, without resorting to this ugly thing called NAT.  Of course, this doesn't mean that you have to fully open your network to dn42: similarly to IPv6, you can still use a firewall (but you could, for instance, allow incoming TCP 22 and TCP 80 from dn42 by default).

If your hackerspace is actually using dn42 to provide some services, please let us know! (on this wiki or on the mailing list). It's very rewarding when the network is actually used for something :)

## Join or Contact us

dn42 is operated by a group of volunteers. There is no central authority which controls or impersonates the network. Take a look at the [contact](/contact) page to see how to collaborate or contact us.

The [Getting started](/howto/Getting-Started) page helps you to get your first node inside the network.

## External resources about dn42

 * [Wikipedia about dn42](https://en.wikipedia.org/wiki/Decentralized_network_42)
 * [Lecture on 26c3](https://fahrplan.events.ccc.de/congress/2009/Fahrplan/events/3504.en.html)
 * [Lecture on GPN8](https://entropia.de/GPN8:dn42)
 * [nobody about dn42](http://nowhere.ws/guides/dn42/)
 * [Lecture on mrmcd0x8](https://web.archive.org/web/20090831211324/http://mrmcd0x8.metarheinmain.de/fahrplan/events/3321.de.html)
 * [dn42-category in hackerspaces.org wiki](https://wiki.hackerspaces.org/Category:DN42)
 * [pentaradio24 – german podcast](https://www.c3d2.de/news/pentaradio24-20150428.html)
 * [dn42 in your browser](http://www.freertr.net/online.html)

## Participant Groups

* [SpaceBoyz](http://spaceboyz.net)
* [CCC Aachen (German)](https://aachen.ccc.de)
* [CCC Bremen (German)](http://ccchb.de)
* [CCC Darmstadt (German)](http://darmstadt.ccc.de)
* [CCC Dresden (German)](http://c3d2.de)
* [CCC Düsseldorf (German)](https://www.chaosdorf.de)
* [CCC Munich (German)](https://www.muc.ccc.de)
* [Chaostreff Chemnitz (German)](https://chaoschemnitz.de)
* [/dev/nulll](https://dev.0l.de)
* [freifunk (German)](http://freifunk.net)
* [NoName e.V. Heidelberg (German)](https://www.noname-ev.de)
* [raumzeitlabor/hackerspace rhein-neckar (German)](http://www.raumzeitlabor.de)
* [Hackerspace Brussels (HSB)](http://hackerspace.be)
* [[hsmr] / Hackspace Marburg (German)](https://hsmr.cc)
* [Whitespace (0x20)](http://www.0x20.be)
* [Revelation Space (Dutch)](http://www.revspace.nl)
* [SNE group](https://www.os3.nl)
* [smrsh](http://www.smrsh.net)
* [Hackspace Jena e.V. (German)](https://kraut.space)
* [Breizh-Entropy (French)](http://wiki.breizh-entropy.org/wiki/DN42)
* [Fédération FDN (French)](https://www.ffdn.org)
* [Le LOOP (French)](https://leloop.org/)
* [Hackerspace Bielefeld (German)](https://hackerspace-bielefeld.de)
* [fixmix Technologies Ltd](https://dn42.fixmix.tech/)
* [Strategic Explorations Ltd](https://strexp.net)

## About this wiki

This wiki is the main reference about dn42.  It is available in read-only mode from the Internet [here](https://wiki.dn42.us) or [here](https://dn42.dev) or [here](https://dn42.tk) or [here](https://dn42.eu) or [here (v6 only)](https://dn42.de) and for editing from within dn42, at [https://wiki.dn42](https://wiki.dn42) - [https](services/Certificate-Authority) required for editing.

### DN42 Logo

An svg of the DN42 Logo is available [here](/dn42.svg).

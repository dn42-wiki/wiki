# Historical Services

**The services below were available on DN42 in the past.**

**This section exists to serve as an inspiration for people wanting to provide a service to the DN42 community.**
***

You can inspect the services status [on this page](https://services.dn42)

## Network-related
  * [net.smrsh.dn42/routes/d3js.html](http://net.smrsh.dn42/routes/d3js.html) aka 172.23.174.1 (dn42) or [dn42.smrsh.net/routes/d3js.html](http://dn42.smrsh.net/routes/d3js.html) (Internet)
  * Polynome has some nice scripts and visualizations here: <http://dataviz.polynome.dn42/dn42-netblock-visu/registry.html>
  * DN42 Toplevel domain DNS monitoring: [gatuno.dn42/dns](http://gatuno.dn42/dns)
  * Free DNS Hosting. You can host any toplevel or subdomain from dn42: [gatuno.dn42/managed](http://gatuno.dn42/managed/)
  * Nixnodes original Map of the network: [map.nixnodes.net](http://map.nixnodes.net)
  * IP base network map: [map.jh0project.dn42](https://jh0project.dn42/map) (DN42) or [dn42.jh0project.com](https://dn42.jh0project.com/map) (IANA) _(uses ping and traceroute the whole DN42's ipv4 addresss block.)_
  * BGP lookup tool: [bgp.charlie.dn42](https://bgp.charlie.dn42/) (DN42)
  * Observed routes over time (from the perspective of the C4TG1RL5 network): <https://catgirls.dn42/routes/>

### DNS Hosting
Free DNS Hosting is provided by tombii - currently in a beta test phase. Please contact tombii in #dn42 to get an account.

DNS Hosting with PowerDNS GUI is provided by Nellicus. Support for Domains and RDNS. Please contact nellicus in #dn42 or email admin (at) nellic.us

DNS Hosting with PowerDNS GUI is also provided by KaiRaphixx. Support for Domains and RDNS. Please contact KaiRaphixx in #dn42 or email ich (at) kai.cool

### DNS tunnel

This DNS tunnel service uses [Iodine](http://code.kryo.se/iodine/), and provides access to the dn42 network. Useful when you're on a shitty network (airport, train station) that still allows DNS.

Use the anycast DNS servers (172.22.0.53) inside your tunnel.

| Hostname / IP                                     | Password |
|:------------------------------------------------- |:-------- |
| t.polyno.me (172.23.185.193)                      | dn42     |
| t.hax404.de (172.23.136.98)                       | dn42     |


### DNS Tools

This tool allows you to lookup your dn42 domain name and check to see if your name servers are all working and have the correct information.

Select "Disable Recursion" to check only entries found in the registry or leave it off to check all (both are useful tests).

Currently this system only supports IPv4.

<http://mwd.dn42/dns.php>

MWD will also provide a secondary DNS server and/or cacti monitoring of your devices. Just ask on IRC. More info: <http://mwd.dn42>

### Getting your current dn42 IPv4/IPv6 address
  * What is my IP: [ip4.dn42](http://ip4.dn42/), [ip6.dn42](http://ip6.dn42/)
  * What is my IP: [whatismyip.dn42](http://whatismyip.dn42/)
  * <http://wieistmeineip.dn42> provides a service like <http://wieistmeineip.de>, but for dn42.
wieistmeineip.dn42 also provides a telnet service that returns the address you connected with. This service only shows you the address of the preferred protocol, but there are also ipv4.wieistmeineip.dn42 and ipv6.wieistmeineip.dn42 that accept only connections via IPv4/IPv6.
  * You can also use <http://whatismyip.dn42> from inside dn42 to get your IPv4 and IPv6 address. It also returns information about your latency, netblock details, and route information.
  * An alternative is available at <https://ip.naive.network>, which displays your clearnet and dn42 IP addresses.

## Search engines

| Hostname / IP                                     | Remarks                                                  |
|:------------------------------------------------- |:-------------------------------------------------------- |
| <http://yacy.dn42> (OFFLINE 2020-01-18)             | YaCy search engine. Indexing local nets                  |
| _Configuring Yacy Network settings:_              |[YaCy Network Configuration](http://yacy.dn42/yacy.network.dn42.unit) |
| <http://mhm.dn42/search>                            | Hosted by toBee                                          |
| <search.marlinc.dn42>                               | Hosted by marlinc                                        |

## Radio and Video Streaming

| Hostname / IP                                     | Remarks                                                         |
|:------------------------------------------------- |:--------------------------------------------------------------- |
| <https://invidious.doxz.dn42/> (BROKEN 2021-04-19)  | Invidious instance with proxy (Youtube)                         |
| <http://stream.media.dn42/>                         | icecast-relay, contact toBee for more streams (DOWN 2020-11-02) |
| <http://radio.hex.dn42/>                            | Ambient musics                                                  |
| <https://yp.unknownts.dn42/>                        | A YellowPages for internet radio stations inside dn42          |
| <http://deep.radio.unknownts.dn42/>                 | Internet radio playing random music                            |
| <http://rickroll.dn42>                              | Play Rickroll Video                                            |

## Images, E-Books, Videos and other Media

| Hostname / IP                                     | Remarks                                                  |
|:------------------------------------------------- |:-------------------------------------------------------- |
| <http://img.dn42>                                   | Imagehoster                                              |
| <http://chan.dn42>                                  | DN42-Chan, an imageboard                                 |

## File Sharing

### FTP / HTTP

| Hostname / IP                                               | Space | Speed       | Remarks                                        |
|:----------------------------------------------------------- |:----- |:----------- |:---------------------------------- |
| <http://filer.mhm.dn42>                                       |  4TB  | 1GBit       | 24/7/365                           |
| <http://data.0l.dn42>                                         |  5TB  | 1GBit       | 24/7/365, download, dn42 MRT dumps |
| <http://seafile.dn42>                                         |       |             | Opensource Dropbox, yay!                       |

### Tahoe LAFS
Some people runs [Tahoe LAFS](/services/Tahoe-LAFS) nodes to provide a secure decentralized crypted file storage but in dn42.

### ipfs
bootstrap peers
```
/ip4/172.20.161.135/tcp/4001/ipfs/QmYgD1wdPjx5oWzYJ195K84PqAXRnw9mcqbyZYAdXfaYkD
/ip4/172.20.52.220/tcp/4001/ipfs/QmW5ZhZFav8MZLJyvKuK6pKaR4vnQ5MVHfXY3LuMXqa4kc
```
test hashes
```
/ipfs/QmQ7psrGrXS3GFNC4BtU6pJXq6G7ps5NbYrhS2VYFufj9T
/ipfs/QmYLapmcSU7q93Ta4eHMh8fq9ios2HTSdbpHDRQwGG6ocJ
```
cdn (currently only jquery
```
/ipns/QmW5ZhZFav8MZLJyvKuK6pKaR4vnQ5MVHfXY3LuMXqa4kc/cdn/jquery
```
Until browsers have ipfs access (either through native support or js), one can use the http gateway
```
https://rest.dn42/
```

### Torrent Search Engine

* <https://magnetic.dn42> (DHT Search Engine)

### Torrent Index

* <http://torrents.dn42>

### Torrent Tracker

| Hostname / IP        | Port | Protocol    | Remarks        | Announce URL                            |
|:---------------------|:-----|:------------|:---------------|:----------------------------------------|
| tracker.mhm.dn42     | 6969 | TCP & UDP   | Opentracker    | <http://tracker.mhm.dn42:6969/announce>   |
| tracker.mhm.dn42     | 80   | TCP & UDP   | Opentracker    | <http://tracker.mhm.dn42/announce>        |

## NTP

| Hostname / IP                                     | Remarks                             |
|:------------------------------------------------- |:----------------------------------- |
| ntp.e-utp.dn42 (172.22.165.50)                    | Stratum 1, GPS+NMEA                 |
| ntp1.nixnodes.dn42 (172.22.177.123)               |                                     |
| ntp2.nixnodes.dn42 (172.22.177.124)               |                                     |
| ntp.martin89.dn42                                 | more than one A records/server      |
| tick.gotroot.dn42 (172.20.14.247)                 | Stratum 1, GPS, Vancouver Canada    |
| tock.gotroot.dn42 (172.20.14.250)                 | Stratum 2, Anycast on each node     |
| ntp.yuetau.dn42 (172.21.68.50)                    | Anycast on all node                 |

## OS Mirror/Repository's

Also check [Repository Mirrors](/services/Repository-Mirrors)

| Hostname / IP                                     | What's Available:                   | Updates
|:------------------------------------------------- |:----------------------------------- |:----------------------------------- |
| <http://debian.trunet.dn42>                       | Debian mirror                       | Each 6 hours                        |
| <http://ubuntu.trunet.dn42>                       | Ubuntu releases mirror              | Each 4 hours                        |
| <http://archive.ubuntu.trunet.dn42>               | Ubuntu archive mirror               | Each 6 hours                        |
| <http://centos.trunet.dn42>                       | CentOS mirror                       | Each 6 hours                        |
| ~~http://files.twink0r.dn42~~(OFFLINE 2016-08-24) | Debian, Ubuntu                      |                                     |
| ~~http://freebsd.e-utp.dn42~~(OFFLINE 2016-08-24) | FreeBSD Homepage mirror             |                                     |
| <http://mirrors.zhaofeng.dn42/archlinux>          | Arch Linux                          | Every hour          |

### Direct Connect
Some [Advanced Direct Connect](https://en.wikipedia.org/wiki/Advanced_Direct_Connect) Hubs are being run DN42 internally. Choose a [client](https://en.wikipedia.org/wiki/Comparison_of_ADC_software#Client_software) and connect to exchange files.

| Address                        |
|:-------------------------------|
| adcs://hub.dcpp.dn42:1511      |
| dchub://hub.dcpp.dn42:2780     |
| dchub://dcpp.grmml.dn42:4111   |


## Misc

| Hostname / IP                                     | Remarks                                                                        |
| ------------------------------------------------- | ------------------------------------------------------------------------------ |
| <https://bin.dn42> | AES-encrypted pastebin-like service ([zerobin](https://github.com/sebsauvage/ZeroBin)) |
| <http://pastebin.trunet.dn42>                       | AES-encrypted pastebin-like ([zerobin](https://github.com/sebsauvage/ZeroBin)) |
| ~~http://zerobin.e-utp.dn42~~                     | AES-encrypted pastebin-like, second one ([zerobin](https://github.com/sebsauvage/ZeroBin)) |
| <https://pad.dn42> | [Etherpad](http://etherpad.org) service for collaborative work |
| <http://ip.synhacx.dn42>                            | Basic "whatismyip" service ([description](http://synhacx.dn42/showmyip))       |
| <http://tor.e-utp.dn42>                             | Tor Project Homepage mirror  |
| <http://ngit.dn42>                                  ||
| nntp://news.blacksheep.dn42                       | Martin's newsgroup server (ping MB-DN42 for a rw account or a nntp/uucp feed)  |
| mumble://shard.smrsh.dn42:64738                   | [Mumble](http://mumble.sourceforge.net/) Voice Chat |
| ts3.kai-server.dn42 / ts3.fastnameserver.eu       | Teamspeak 3 Server (also reachable over clearnet) |
| <https://whois.rest.dn42/>                          | whois restful API |
| [pgp.dn42](http://pgp.dn42)                                          | PGP keyserver, [synchronizes](http://pgp.dn42/pks/lookup?op=stats) with the SKS keyservers |
| https://git.dn42[.us]                             | Git Repository Hosting (Signup: email ssh pubkey to xuu@dn42.us) |
| https://git.dn42[.us]/pubkeys/[username]          | Get ssh public keys from Git Users of git.dn42. |
| http://teams.dn42[.us]/dn42                       | Mattermost (Slack clone) instance: get notifications for wiki/CA changes here  |
| <http://nowhere.ws/dn42>                            | Some random stuff concerning dn42, packages for Debian, e.g. Quagga        |
| <https://paste.weiti.dn42>                          | AES-encrypted pastebin-like (privatebin) |
| <https://bbs.dn42>, <https://dn42bbs.0b1.me> via Clearnet | A general BBS powered by Flarum for virtually any topics. Maintained by nicholascw.|
| <http://jack.pyropeter.eu/dn42/routecount/>         | Statistics about the number of v4/v6 routes seen by AS76115 (Since Aug. 2014)  |
| [Clearnet](https://flapping.p2p-node.de/dashboard/), [dn42](https://flapping.bandura.dn42/dashboard), [NeoNetwork](https://flapping.bandura.neo/dashboard/) | FlapAlertedPro by Kioubit hosted by mark22k |

## Gaming

| Hostname / IP                                     | Game                   | Remarks                    |
|:------------------------------------------------- |:---------------------- |:-------------------------- |
| mc.nia.dn42 (172.20.168.137, fd01:1926:817:7::)   | Minecraft              | Latest Stable, Optimized for CN , Map available on mc-map.nia.dn42 |
| hulk.mhm.dn42 (172.23.67.1)                       | Tetrinet               |                            |
| ns1.deltaman.dn42 (172.22.134.131, fd1b:7f7d:dd55:4600:219:ff:fe00:fafe) | OpenTTD | 1.10.3, Hosted in NL   |
| mc.nico.dn42 | Minecraft | 1.16.5, [Forge Modded](https://bbs.dn42/d/17-modded-116-minecraft-server), IPv4 & IPv6, Central US Server |
| mc.northrend.dn42 (172.20.222.240)		    | Minecraft		     | latest, IPv4 only   |
| redtrap.northrend.dn42 (172.20.222.252)	    | Minecraft		     | RedTrap DN42 Bridge, 1.8.8 - 1.19, IPv4 only   |

## DN42 FreePhone
Somebody was providing a FreePhone [here](/services/FreePhone)

## Email Providers
There is a page for email Providers [here](/services/E-Mail-Providers)

# Other Networks

## ChaosVPN

  * Anybody can add services to this list, which will be monitored for uptime: <http://10.100.44.1>
  * Check your IP and reverse lookup: [ifconfig.hack](http://ifconfig.hack)
  * View of the network: <http://vpnhub1-intern.hamburg.ccc.de/chaosvpn.png>
  * List of nodes: <http://vpnhub1-intern.hamburg.ccc.de/chaosvpn.nodes.html>

## Freifunk

### Augsburg

We have a plugin that enables us to announce services in the mesh. So instead of listing them here again just have a look at <http://10.11.0.8/cgi-bin/luci/freifunk/services> to see what we have to offer.
(Upload is not fast, most probably DSL speed only)

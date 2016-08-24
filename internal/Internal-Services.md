# Internal services

You are asked to show some creativity in terms of network usage and content. ;)

More ideas inspiration is collected on another [page](/internal/ideas).

You can inspect the service status [on this page](https://services.dn42)

## CA

xuu is maintaining an [[certificate authority]] for internal services.

## Network-related

  * See [[Looking Glasses]] for more network diagnostic tools
  * Polynome has some nice scripts and visualizations here: http://dataviz.polynome.dn42/dn42-netblock-visu/registry.html
  * [net.smrsh.dn42/routes/d3js.html](http://net.smrsh.dn42/routes/d3js.html) aka 172.23.174.1 (dn42) or [dn42.smrsh.net/routes/d3js.html](http://dn42.smrsh.net/routes/d3js.html) (Internet)
  * [map.nixnodes.net](http://map.nixnodes.net)

The data for these maps is collected using AS paths from various AS.

### DNS Hosting
Free DNS Hosting is provided by tombii - currently in a beta test phase. Please contact tombii in #dn42 to get an account.

DNS Hosting with PowerDNS GUI is provided by Nellicus. Support for Domains and RDNS. Please contact nellicus in #dn42 or email admin (at) nellic.us
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

http://mwd.dn42/dns.php

MWD will also provide a secondary DNS server and/or cacti monitoring of your devices. Just ask on IRC. More info: http://mwd.dn42

### Getting you current dn42 IPv4/IPv6 address

http://wieistmeineip.dn42 provides a service like http://wieistmeineip.de, but for dn42.
wieistmeineip.dn42 also provides a telnet service that returns the address you connected with. This service only shows you the address of the preferred protocol, but there are also ipv4.wieistmeineip.dn42 and ipv6.wieistmeineip.dn42 that accept only connections via IPv4/IPv6.

You can also use http://whatismyip.dn42 from inside dn42 to get your IPv4 and IPv6 address. It also returns information about your latency, netblock details, and route information.

## IRC

| Hostname / IP                               | Remarks                                   |
|:------------------------------------------- |:----------------------------------------- |
| irc://irc.hackint.dn42/dn42 (172.20.66.67)  | DN42                                      |
| irc://irc.hackint.hack/dn42 (172.31.0.30)   | ChaosVPN                                  |
| irc://irc.nixnodes.dn42                     | NixNodes IRC Network                      |
| ircs://fpletz.darkfasel.dn42 (172.23.214.5) | darkfasel, TLSv1+ only, Ports 6697 & 9999 |

## Search engines

| Hostname / IP                                     | Remarks                                                  |
|:------------------------------------------------- |:-------------------------------------------------------- |
| http://mhm.dn42/search                            | Hosted by toBee                                          |
| http://yacy.dn42                                  | YaCy search engine. Indexing local nets|
| http://yacy.marlinc.dn42:8090/                    | Marlinc's YaCy node.                   |
| https://surf.dn42/                                | siska's YaCy node.                     |
| http://yacy.hexa.dn42/                            | hexa-'s YaCy node.                     |
|                                                   |[YaCy Network Configuration](http://yacy.dn42/yacy.network.dn42.unit)|
| http://search.dn42 (172.23.184.1)                 | a few chosen HTTP domains are crawled  (taken from the wiki). The previous method, "crawl everything available from the wiki", generated too much data because of FTPs. |
| https://surf.dn42                                 | YaCy node                                       |

## Images and Media

| Hostname / IP                                     | Remarks                                                  |
|:------------------------------------------------- |:-------------------------------------------------------- |
| http://img.dn42                                   | Imagehoster                                              |
| http://chan.dn42                                  | DN42-Chan, an imageboard                                 |
| http://media.dn42                | A mediagoblin-Instance  |

## Radio and Video Streaming

| Hostname / IP                                     | Remarks                                                  |
|:------------------------------------------------- |:-------------------------------------------------------- |
| http://sprawl.smrsh.dn42:8000/                    | [smrsh radio](http://smrsh.net/radio)                    |
| http://stream.media.dn42/                         | icecast-relay, contact toBee for more streams             |

## Voice and video calls

| Hostname / IP                                     | Remarks                                                  |
|:------------------------------------------------- |:-------------------------------------------------------- |
| http://zaledia.dn42/                              | Zaledia VOIP service. Contact ranma on IRC OR julien@zaledia.dn42 or julien.owls@gmail.com to get your account. 

## File sharing

### Tahoe LAFS
Some people runs [Tahoe LAFS](/services/Tahoe-LAFS) nodes to provide a secure decentralized crypted file storage butt in dn42.

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

### Direct Connect
An [Advanced Direct Connect](https://en.wikipedia.org/wiki/Advanced_Direct_Connect) Hub is being run at `hub.dcpp.dn42:2780`. Choose a [client](https://en.wikipedia.org/wiki/Comparison_of_ADC_software#Client_software) and connect to exchange files.

### FTP / HTTP

**FIXME**: Please add info about (approximate) bandwidth of the servers.

| Hostname / IP                                               | Space | Speed       | Remarks                                        |
|:----------------------------------------------------------- |:----- |:----------- |:---------------------------------------------- |
| http://172.22.92.2                                  |       | ~60kbps     | mostly up                                      |    
| http://seafile.dn42                                         |       |             | Opensource Dropbox, yay!                       |
| http://files.feuerrot.dn42                                  |  6TB  | 1Gbit       | http, ftp, nfs, rsync                          |
| sftp://anonsftp:Iich0zieC3retaid@files.crest.dn42:2212/     | 12TB  | 1Gb/s       | incoming writable                              |
| http://files.martin89.dn42/                                 |       | max 2Mbit/s | download only                                  |
| http://filer.mhm.dn42                                       |  4TB  | 1GBit | 24/7/365 |  |
| http://storage.hq.c3d2.de:8080/rpool                        |       | 2.4Mbit/s   | download only webdav:k-ot|
| ftp://nas.jan.dn42/                                         |  6TB  | 10 Mbit/s   | anonymous read/write                           |

### Torrent Tracker

| Hostname / IP        | Port | Protocol    | Remarks        | Announce URL                            |
|:---------------------|:-----|:------------|:---------------|:----------------------------------------|
| tracker.grmml.dn42   | 6969 | TCP & UDP   | Opentracker    | http://tracker.grmml.dn42:6969/announce |
| tracker.mhm.dn42     | 6969 | TCP & UDP   | Opentracker    | http://tracker.mhm.dn42:6969/announce   |
| tracker.mhm.dn42     | 80   | TCP & UDP   | Opentracker    | http://tracker.mhm.dn42/announce        |

### Torrent Index

* http://torrents.dn42
* https://torrents.dn42

## Proxies

 See http://wiki.hamburg.ccc.de/ChaosVPN:Proxy

### Tor

| Hostname / IP                                     | Bandwidth   | Nickname     | Info   |
| ------------------------------------------------- | ----------- | ------------ | ------------  |
| socks5://172.23.162.132:9050                      | 100 Mbit/s  | [GrmmlLitavis] | https://atlas.torproject.org/#details/7CB8C31432A796731EA7B6BF4025548DFEB25E0C
| socks5://172.20.11.33:9050                        | 100 Mbit/s  | [twink0r]     
|tor.dn42 (172.22.0.94 [anycast])                   | -- | -- | ORPort: 9001/tcp; DirPort: 9030/tcp; SocksPort: 9050/tcp |

## NTP

| Hostname / IP                                     | Remarks                             |
|:------------------------------------------------- |:----------------------------------- |
| ntp.e-utp.dn42 (172.22.165.50)                    | Stratum 1, GPS+NMEA                 |
| ntp1.nixnodes.dn42 (172.22.177.123)               |                                     |
| ntp2.nixnodes.dn42 (172.22.177.124)               |                                     |
| ntp.martin89.dn42                                 | more than one A records/server      |
| tick.gotroot.dn42 (172.20.14.247)                 | Stratum 1, GPS, Vancouver Canada    |
| tock.gotroot.dn42 (172.20.14.250)                 | Stratum 2, Anycast on each node     |

## OS Mirror/Repository's

Also check [Repository Mirrors](services/Repository-Mirrors)

| Hostname / IP                                     | What's Available:                   | Updates
|:------------------------------------------------- |:----------------------------------- |:----------------------------------- |
| http://debian.mirror.martin89.dn42                | Debian mirror                       |                                     |
| http://ubuntu.trunet.dn42                         | Ubuntu releases mirror              | Each 4 hours                        |
| http://archive.ubuntu.trunet.dn42                 | Ubuntu archive mirror               | Each 6 hours                        |
| ~~http://files.twink0r.dn42~~(OFFLINE 2016-08-24) | Debian, Ubuntu                      |                                     |
| ~~http://freebsd.e-utp.dn42~~(OFFLINE 2016-08-24) | FreeBSD Homepage mirror             |                                     |

## Gaming

| Hostname / IP                                     | Game                   | Remarks                    |
|:------------------------------------------------- |:---------------------- |:-------------------------- |
| hulk.mhm.dn42 (172.23.67.1)                       | Tetrinet               |                            |
| gaming.marlinc.dn42:27015                         | Counter Strike: Source |                            |
| 172.22.177.92:27017 (external:gmod.nixnodes.net)                              | Garry's Mod: Sandbox   | LUA coding, cinema, steam + non-steam, pass: 42 (required from public)   |

## Misc 

| Hostname / IP                                     | Remarks                                                                        |
| ------------------------------------------------- | ------------------------------------------------------------------------------ | 
| http://teams.dn42[.us]/dn42                       | Mattermost (Slack clone) instance: get notifications for wiki/CA changes here  |
| http://nowhere.ws/dn42                            | Some random stuff concerning dn42, packages for Debian, e.g. Quagga            |
| http://pastebin.trunet.dn42                       | AES-encrypted pastebin-like ([zerobin](https://github.com/sebsauvage/ZeroBin)) |
| ~~https://paste.synhacx.dn42~~(OFFLINE 2016-08-24)| AES-encrypted pastebin-like ([zerobin](https://github.com/sebsauvage/ZeroBin)) |
| ~~http://zerobin.e-utp.dn42~~(OFFLINE 2016-08-24) | AES-encrypted pastebin-like, second one ([zerobin](https://github.com/sebsauvage/ZeroBin)) |
| ~~https://flo.dn42/paste/~~(OFFLINE 2016-08-24)   | AES-256-encrypted pastebin-like, with HTTPS ([zerobin])                        |
| ~~https://szf.dn42/paste/~~(OFFLINE 2016-08-24)   | AES-encrypted pastebin-like, another one                                       |
| http://ip.synhacx.dn42                            | Basic "whatismyip" service ([description](http://synhacx.dn42/showmyip))       |
| http://nixnodes.dn42/ip                           | Simple 'myip' service                                                          |
| https://szf.dn42/ip (text) https://szf.dn42/ifconfig (html)   | Another simple 'myip' service                                      |
| https://weiti.dn42/cgi-bin/my-ip | Another 'myip' service |
| https://git.dn42[.us]                             | Git Repository Hosting (Signup: email ssh pubkey to xuu@dn42.us)               |
| https://git.dn42[.us]/pubkeys/[username]          | Get ssh public keys from Git Users of git.dn42. |
| http://ngit.dn42                                  | |
| http://tor.e-utp.dn42                             | Tor Project Homepage mirror                                                    |
| nntp://news.blacksheep.dn42                       | Martin's newsgroup server (ping MB-DN42 for a rw account or a nntp/uucp feed)  |
| mumble://shard.smrsh.dn42:64738                   | [Mumble](http://mumble.sourceforge.net/) Voice Chat                            |
| http://wiki.dn42, http://internal.dn42, [dn42.i2p](http://beb6v2i4jevo72vvnx6segsk4zv3pu3prbwcfuta3bzrcv7boy2q.b32.i2p/) (i2p), jsptropkiix3ki5u.onion  | This wiki! Web Hosted by [xuu](https://xuu.dn42). Git Repo hosted on git.dn42  |                
| http://jack.pyropeter.eu/dn42/routecount/         | Statistics about the number of v4/v6 routes seen by AS76115 (Since Aug. 2014)  |
| xmpp://jabber.szf.dn42                            | XMPP/Jabber server, port 5222 w/ STARTTLS only |
| https://whois.rest.dn42/                          | whois restful API |

# Other networks

## Public Internet

 * https://mirror.frubar.net 100MBit
 * https://frucman.frubar.net

## AnoNet

A wiki page dedicated to the AnoNet Network: http://wiki.qontrol.nl/Anonet

## ChaosVPN

  * Anybody can add services to this list, which will be monitored for uptime: http://10.100.44.1
  * Check your IP and reverse lookup: [ifconfig.hack](http://ifconfig.hack)
  * View of the network: http://vpnhub1-intern.hamburg.ccc.de/chaosvpn.png
  * List of nodes: http://vpnhub1-intern.hamburg.ccc.de/chaosvpn.nodes.html

## Freifunk

### Augsburg

We have a plugin that enables us to announce services in the mesh. So instead of listing them here again just have a look at http://10.11.0.8/cgi-bin/luci/freifunk/services to see what we have to offer.
(Upload is not fast, most probably DSL speed only)

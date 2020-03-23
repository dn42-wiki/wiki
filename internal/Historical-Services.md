# Historical Services

**The services below were available on DN42 in the past.**

**This section exists to serve as an inspiration for people wanting to provide a service to the DN42 community.**
***

You can inspect the services status [on this page](https://services.dn42)

## Network-related
  * [net.smrsh.dn42/routes/d3js.html](http://net.smrsh.dn42/routes/d3js.html) aka 172.23.174.1 (dn42) or [dn42.smrsh.net/routes/d3js.html](http://dn42.smrsh.net/routes/d3js.html) (Internet)
  * Polynome has some nice scripts and visualizations here: http://dataviz.polynome.dn42/dn42-netblock-visu/registry.html

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

http://mwd.dn42/dns.php

MWD will also provide a secondary DNS server and/or cacti monitoring of your devices. Just ask on IRC. More info: http://mwd.dn42

### Getting your current dn42 IPv4/IPv6 address

http://wieistmeineip.dn42 provides a service like http://wieistmeineip.de, but for dn42.
wieistmeineip.dn42 also provides a telnet service that returns the address you connected with. This service only shows you the address of the preferred protocol, but there are also ipv4.wieistmeineip.dn42 and ipv6.wieistmeineip.dn42 that accept only connections via IPv4/IPv6.

You can also use http://whatismyip.dn42 from inside dn42 to get your IPv4 and IPv6 address. It also returns information about your latency, netblock details, and route information.

An alternative is available at https://ip.naive.network, which displays your clearnet and dn42 IP addresses.

## Search engines

| Hostname / IP                                     | Remarks                                                  |
|:------------------------------------------------- |:-------------------------------------------------------- |
| http://yacy.dn42 (OFFLINE 2020-01-18)             | YaCy search engine. Indexing local nets                  |
| _Configuring Yacy Network settings:_     |[YaCy Network Configuration](http://yacy.dn42/yacy.network.dn42.unit) |


## File Sharing

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

### Torrent Index

* http://torrents.dn42

### Torrent Tracker

| Hostname / IP        | Port | Protocol    | Remarks        | Announce URL                            |
|:---------------------|:-----|:------------|:---------------|:----------------------------------------|
| tracker.mhm.dn42     | 6969 | TCP & UDP   | Opentracker    | http://tracker.mhm.dn42:6969/announce   |
| tracker.mhm.dn42     | 80   | TCP & UDP   | Opentracker    | http://tracker.mhm.dn42/announce        |

## NTP

| Hostname / IP                                     | Remarks                             |
|:------------------------------------------------- |:----------------------------------- |
| ntp.e-utp.dn42 (172.22.165.50)                    | Stratum 1, GPS+NMEA                 |
| ntp1.nixnodes.dn42 (172.22.177.123)               |                                     |
| ntp2.nixnodes.dn42 (172.22.177.124)               |                                     |

## OS Mirror/Repository's

Also check [Repository Mirrors](/services/Repository-Mirrors)

| Hostname / IP                                     | What's Available:                   | Updates
|:------------------------------------------------- |:----------------------------------- |:----------------------------------- |
| http://debian.trunet.dn42                         | Debian mirror                       | Each 6 hours                        |
| http://ubuntu.trunet.dn42                         | Ubuntu releases mirror              | Each 4 hours                        |
| http://archive.ubuntu.trunet.dn42                 | Ubuntu archive mirror               | Each 6 hours                        |
| http://centos.trunet.dn42                         | CentOS mirror                       | Each 6 hours                        |
| ~~http://files.twink0r.dn42~~(OFFLINE 2016-08-24) | Debian, Ubuntu                      |                                     |
| ~~http://freebsd.e-utp.dn42~~(OFFLINE 2016-08-24) | FreeBSD Homepage mirror             |                                     |
| http://mirrors.zhaofeng.dn42/archlinux            | Arch Linux                          | Every hour          |


## Misc 

| Hostname / IP                                     | Remarks                                                                        |
| ------------------------------------------------- | ------------------------------------------------------------------------------ | 
|https://bin.dn42 | AES-encrypted pastebin-like service ([zerobin](https://github.com/sebsauvage/ZeroBin)) | 
| http://pastebin.trunet.dn42                       | AES-encrypted pastebin-like ([zerobin](https://github.com/sebsauvage/ZeroBin)) |
| ~~http://zerobin.e-utp.dn42~~                     | AES-encrypted pastebin-like, second one ([zerobin](https://github.com/sebsauvage/ZeroBin)) | 
| https://pad.dn42 | [Etherpad](http://etherpad.org) service for collaborative work |
| http://ip.synhacx.dn42                            | Basic "whatismyip" service ([description](http://synhacx.dn42/showmyip))       |
| http://tor.e-utp.dn42                             | Tor Project Homepage mirror  |   
| http://ngit.dn42                                  ||
| nntp://news.blacksheep.dn42                       | Martin's newsgroup server (ping MB-DN42 for a rw account or a nntp/uucp feed)  |
| mumble://shard.smrsh.dn42:64738                   | [Mumble](http://mumble.sourceforge.net/) Voice Chat |
| ts3.kai-server.dn42 / ts3.fastnameserver.eu       | Teamspeak 3 Server (also reachable over clearnet) |
| https://whois.rest.dn42/                          | whois restful API |
| [pgp.dn42](http://pgp.dn42)                                          | PGP keyserver, [synchronizes](http://pgp.dn42/pks/lookup?op=stats) with the SKS keyservers |
         

## DN42 FreePhone
Somebody was providing a FreePhone [here](/services/FreePhone)

## Email Providers
There is a page for email Providers [here](/services/E-Mail-Providers)

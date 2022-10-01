# Internal services

You are asked to show some creativity in terms of network usage and content. ;)

**More inspiration is collected [here](/internal/Historical-Services) and [here](/internal/Ideas).**

## Search engine

There is a search engine at [search.dn42](https://search.dn42) that can also be used to discover services and content. It attempts to index all sites on DN42.

## Certificate Authority

xuu is maintaining an [certificate authority](/services/Certificate-Authority) for internal services.

n0emis maintains an [ACME server](https://acme.dn42) (with accompanying CA), compatible with any LetsEncrypt client like Certbot, Dehydrated or Caddy.

## Network-related
  * See [Looking Glasses](/services/Looking-Glasses) for more network diagnostic tools
  * Realtime network map: [map.dn42](http://map.dn42/) (DN42) or [map42.0x7f.cc](https://map42.0x7f.cc), [map.kuu.moe](https://map.kuu.moe) (IANA) _(Note: This is a direct copy of nixnodes map with some fixes and new functions since original map is no longer get maintained. This map is currently using MRT dump from GRC as source. We will pull new dumps from GRC every 15 minutes.)_
  * IP base network map: [map.jh0project.dn42](https://jh0project.dn42/map) (DN42) or [dn42.jh0project.com](https://dn42.jh0project.com/map) (IANA) _(uses ping and traceroute the whole DN42's ipv4 addresss block.)_
  * Network Information Service: [info.nia.dn42](http://info.nia.dn42) (DN42) or [bgp42.strexp.net](https://bgp42.strexp.net) (IANA). Main functions including _network information_, _network map (from map.dn42, require WebGL)_, _network ranking (based on centrality)_, _ROA alerting_ and _path finder_.
  * Yet Another network map: [map.jerry.dn42](https://map.jerry.dn42/) (DN42) or [map.meson.cc](https://map.meson.cc) (via clearnet) _(uses MRT dump as source, updated every 15 minutes.)_
  * Nixnodes original Map of the network: [map.nixnodes.net](http://map.nixnodes.net)
  * DN42 IP address lookup tool: [dn42.g-load.eu/ip](https://dn42.g-load.eu/ip/)
  * New DNS System monitoring: [grafana.burble.com/d/E4iCaHoWk/dn42-dns-status](https://grafana.burble.com/d/E4iCaHoWk/dn42-dns-status?orgId=1&refresh=1m)
  * whatsmyip: 
    * ipv4+ipv6: [myip.dn42](http://myip.dn42/) 
    * ipv4 only: [v4.myip.dn42](http://v4.myip.dn42/) or [172.20.0.81](http://172.20.0.81)
    * ipv6 only: [v6.myip.dn42](http://v6.myip.dn42/) or [fd42:d42:d42:81::1](http://[fd42:d42:d42:81::1]/)
    * /index.html: "fancy" human readable site; /raw: just your ip address; /api: json with ip,serverip,AS of server, location of server and "node_id"

### GeoIP Services
[Map.dn42](http://map.dn42/) provides a simple GeoIP service. This service uses rDNS records and country field in aut-num objects as a reference, currently designed for fun :P

#### API
Results are in JSON format. 

http://ipip.map.dn42/whois?ip=[DN42_IP]&lang=en  
http://ipip.map.dn42/whois?asn=AS[DN42_ASN]

#### Client
There is a client software using above apis to provide GeoIP-based traceroute. 
It is a modified IPIP.NET Best Trace software with DN42 support injection.

Windows only, no virus scan report available, but our DLL source is provided with the modified client. It's highly recommended to run this tool in a sandbox. 

** Since the original software is not open source, so use it at your own risk. **

Preview: http://img.dn42/images/GEOTRACE42.jpg  
Link: http://map.dn42/BestTrace42.zip

### ASN Authentication Solution
Authenticate your users by having them verify their ASN ownership with KIOUBIT-MNT using their registry-provided methods in an automated way. An example of this is the automatic peering system for the Kioubit Network.
To use the service, please message Kioubit on IRC to have your domain activated and receive the required documentation.

## IRC

| Hostname / IP                 | SSL           | Remarks                                     |
|:------------------------------|:------------- |:--------------------------------------------|
| irc.hackint.dn42              |  Yes          | DN42                                        |
| irc.hackint.hack/dn42         |  Yes          | ChaosVPN                                    |
| irc.dn42                      |  Yes          | Internal IRC                                |
| 172.22.69.1 / fda7:3ae7:e04d::1 | Yes         | BonoboNET                                   |

### Clients

| Hostname / IP | Remarks |
|:--------------|:--------|
| https://lounge.burble.dn42 | [thelounge](https://thelounge.chat/) for lurking on #dn42, see [burble.dn42 services](https://dn42.burble.com/home/burble-dn42-services). |

## Images, E-Books, Videos and other Media

| Hostname / IP                                     | Remarks                                                  |
|:------------------------------------------------- |:-------------------------------------------------------- |
| http://j.munsternet.dn42                          | Jellyfin instance with movies and TV shows (test).       |

## Radio and Video Streaming

| Hostname / IP                                     | Remarks                                                        |
|:------------------------------------------------- |:-------------------------------------------------------------- |
| https://live.jerry.dn42/                          | Live audio stream powered by mpd                               |
| http://deep.radio.unknownts.dn42/                 | Internet radio playing random music                            |
| https://dn42:dn42@tv.munsternet.dn42/playlist     | TV Channels Streaming                                          |
| http://icy.jones.dn42                             | Home grown Icecast Radio covering a number of genres (HLS & Player coming soon [ish]!)          |
| http://rickroll.dn42                              | Play Rickroll Video


## File Sharing

### Direct Connect
Some [Advanced Direct Connect](https://en.wikipedia.org/wiki/Advanced_Direct_Connect) Hubs are being run DN42 internally. Choose a [client](https://en.wikipedia.org/wiki/Comparison_of_ADC_software#Client_software) and connect to exchange files.

| Address                        |
|:-------------------------------|
| adcs://hub.dcpp.dn42:1511      |
| dchub://hub.dcpp.dn42:2780     |
| dchub://dcpp.grmml.dn42:4111   |

### FTP / HTTP

**FIXME**: Please add info about (approximate) bandwidth of the servers.

| Hostname / IP                                               | Space | Speed       | Remarks                                        |
|:----------------------------------------------------------- |:----- |:----------- |:---------------------------------------------- |                              
| http://files.nop.dn42                                       |       | max 1Mbit/s | download only                                  |

## VPN

DN42 Network Access over Automatic Wireguard VPN Service (IPv6 only, fd00::/8)
provided by TheQ at https://dn42.0011.de/vpnusers 

## Proxies
 See http://wiki.hamburg.ccc.de/ChaosVPN:Proxy

### Telegram
A MTProxy server is available at [mtp.jerry.dn42:8044](https://t.me/proxy?server=mtp.jerry.dn42&port=8044&secret=ee1419944c0a129dbba2beb2636fcf361a616e64726f69642e676f6f676c65736f757263652e636f6d).

## Tor

Entry points to the Tor network are available on dn42. See [Tor](/internal/Tor) for more details.

## NTP

| Hostname / IP                                     | Remarks                             |
|:------------------------------------------------- |:----------------------------------- |
| ntp.martin89.dn42                                 | more than one A records/server      |
| tick.gotroot.dn42 (172.20.14.247)                 | Stratum 1, GPS, Vancouver Canada    |
| tock.gotroot.dn42 (172.20.14.250)                 | Stratum 2, Anycast on each node     |
| *.burble.dn42 | All burble.dn42 nodes provide NTP over clearnet and DN42. See also [burble.dn42 public services](https://dn42.burble.com/services/public/) |
| ntp.yuetau.dn42 (172.21.68.50)                    | Anycast on all node     |

## OS Mirror/Repository's

Repository Mirrors are listed on another page: [Repository Mirrors](/services/Repository-Mirrors)


## Gaming

| Hostname / IP                                     | Game                   | Remarks                    |
|:------------------------------------------------- |:---------------------- |:-------------------------- |
| hulk.mhm.dn42 (172.23.67.1)                       | Tetrinet               |                            |
| mc.nia.dn42 (172.20.168.137, fd01:1926:817:7::)   | Minecraft              | Latest Stable, Optimized for CN , Map available on mc-map.nia.dn42  |
| yuan.nia.dn42 (172.20.168.244)                    | Genshi Impact (Unofficial) | Latest development branch, Scripts included, Optimized for CN, Not stable yet  |
| mc.nico.dn42 | Minecraft | 1.16.5, [Forge Modded](https://bbs.dn42/d/17-modded-116-minecraft-server), IPv4 & IPv6, Central US Server |
| mc.jerry.dn42 & jerry.dn42                        | Minecraft              | latest, IPv4 & IPv6 |
| mc.northrend.dn42 (172.20.222.240)		    | Minecraft		     | latest, IPv4 only   |
| redtrap.northrend.dn42 (172.20.222.252)	    | Minecraft		     | RedTrap DN42 Bridge, 1.8.8 - 1.19, IPv4 only   |
| ttd.jerry.dn42                                    | OpenTTD                | latest, IPv4 & IPv6 |
| stk.jerry.dn42:2759, stk.jerry.neo:2759           | SuperTuxKart           | latest, IPv4 only   |
| ns1.deltaman.dn42 (172.22.134.131, fd1b:7f7d:dd55:4600:219:ff:fe00:fafe) | OpenTTD      | 1.10.3, Hosted in NL   |
| terraria.pebkac.dn42:7777 (IPv4 only/TCP) | [Terraria](https://terraria.org/) | Ask [TOMKAP-DN42](https://explorer.dn42.pebkac.gr/?#/person/TOMKAP-DN42) for password/whitelisting |
| factorio.catgirls.dn42 (IPv6 only)                | factorio               | Still in testing, expect downtime |
| The burble.dn42 shell servers include a number of classic text games, see [shell access](https://dn42.burble.com/services/shell/#classic-games) | Various | Log in to the shell servers for more |

## Voice chat

|Hostname / IP         | Remarks              |
|:---------------------|:---------------------|
| mumble://ty3r0x.dn42:64738 | Ty3r0X's Lair (men's club)|

## VOIP/SIP Endpoints

 - burble.dn42 runs an [asterisk based VOIP service](https://dn42.burble.com/services/public/#voip) with various test extensions and real hardware modems for dialing in to dn42

## Challenges

Test out your skills with online challenges

| Start here                                        | Remarks                            |
|:------------------------------------------------- |:---------------------------------- |
| https://burble.dn42/services/ping/                | burble.dn42 ping challenge         |


## Shell

Providers of shell access:

| Person        | Hostname                               | Net              | Description | Contact       |
|:------------- |:-------------------------------------- |:---------------- |:---------------- |:------------- |
| mc36          | telnet test.nop.dn42                   | dn42 only        |looking glass     | -             |
| JerryXiao     | ssh lg@lg.jerry.dn42                   | dn42 and icvpn   |looking glass     | -             |
| burble        | ssh <mntner>@shell.fr-rbx1.burble.dn42 <br/> ssh <mntner>@shell.ca-bhs2.burble.dn42 | dn42             |Full shell account| See below |

### burble.dn42 shell access

Full shell accounts are available for all dn42 users. Usernames are 
constructed using the MNTNER name, lowercased and without the '-MNT' suffix. SSH public keys are imported automatically from the registry or alternatively, a password can be used instead.

See also the [burble.dn42 website](https://dn42.burble.com/services/shell/) for more details.

## Misc 

| Hostname / IP                                     | Remarks                                                                        |
| ------------------------------------------------- | ------------------------------------------------------------------------------ |
|[https://bbs.dn42](https://bbs.dn42), [https://dn42bbs.0b1.me](https://dn42bbs.0b1.me) via Clearnet | A general BBS powered by Flarum for virtually any topics. Maintained by nicholascw.|
| http://www.nop.dn42/                              | Basic "whatismyip" service               |
| http://freerouter.nop.dn42/                       | freeRouter main site                     |
| http://rtros.nop.dn42/                            | freeRouter distribution                  |
| http://wiki.dn42, http://internal.dn42            | This wiki!  Git Repo hosted on git.dn42  |
| http://jack.pyropeter.eu/dn42/routecount/         | Statistics about the number of v4/v6 routes seen by AS76115 (Since Aug. 2014)  |
| https://p.pebkac.dn42/                            | PasteBin Service (Netcat/Bash CLI Client) |
| https://sdr.pebkac.dn42/                          | OpenWebRX SDR Receiver, FM/VHF/UHF Analog & Digital |
| https://urandom.catgirls.dn42/                    | Message board |


### Usenet Servers / News
There are some News Servers available [here](/services/News)

## E-Mail
There is a list of E-Mail providers [here](/services/E-Mail-Providers)

# Other networks

**[List of other Overlay Networks](/Other)**

## Public Internet

 * https://mirror.frubar.net 100MBit
 * https://frucman.frubar.net
 

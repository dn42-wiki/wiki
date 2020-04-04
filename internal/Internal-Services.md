# Internal services

You are asked to show some creativity in terms of network usage and content. ;)

**More inspiration is collected [here](/internal/Historical-Services) and [here](/internal/ideas).**

## CA

xuu is maintaining an [[certificate authority]] for internal services.

## Network-related
  * See [[Looking Glasses]] for more network diagnostic tools
  * Map of the network: [map.nixnodes.net](http://map.nixnodes.net)
  * DN42 IP address lookup tool: [dn42.g-load.eu/ip](https://dn42.g-load.eu/ip/)
  * New DNS System monitoring: [grafana.burble.com/d/E4iCaHoWk/dn42-dns-status](https://grafana.burble.com/d/E4iCaHoWk/dn42-dns-status?orgId=1&refresh=1m)
  * DN42 Toplevel domain DNS monitoring: [gatuno.dn42/dns](http://gatuno.dn42/dns)
  * Free DNS Hosting. You can host any toplevel or subdomain from dn42: [gatuno.dn42/managed](http://gatuno.dn42/managed/)
  * What is my IP: [whatismyip.dn42](http://whatismyip.dn42/), [ip4.dn42](http://ip4.dn42/), [ip6.dn42](http://ip6.dn42/)

### Proving ASN ownership
Through this automated service you prove your ASN ownership to KIOUBIT-MNT who then automatically creates a "ownership verification signature".
This signature can be very easily verified by anyone. This removes the hassle from checking every different authentication method in the registry. This is particularly useful for automated setups.

Manual Verification: https://dn42.g-load.eu/verify/manual/

API: https://dn42.g-load.eu/verify/documentation.txt


## IRC

| Hostname / IP                 | SSL           | Remarks                                     |
|:------------------------------|:------------- |:--------------------------------------------|
| irc.hackint.dn42              |  Yes          | DN42                                        |
| irc.hackint.hack/dn42         |  Yes          | ChaosVPN                                    |
| irc.dn42                      |  Yes          | Internal IRC                                |

#### Clients

| Hostname / IP | Remarks |
|:--------------|:--------|
| https://lounge.burble.dn42 | [thelounge](https://thelounge.chat/) for lurking on #dn42, see [burble.dn42 services](https://dn42.burble.com/home/burble-dn42-services). |

## Search engines

| Hostname / IP                                     | Remarks                                                  |
|:------------------------------------------------- |:-------------------------------------------------------- |
| http://mhm.dn42/search                            | Hosted by toBee                                          |

## Images, E-Books, Videos and other Media

| Hostname / IP                                     | Remarks                                                  |
|:------------------------------------------------- |:-------------------------------------------------------- |
| http://img.dn42                                   | Imagehoster                                              |
| http://chan.dn42                                  | DN42-Chan, an imageboard                                 |
| http://j.munsternet.dn42                          | Jellyfin instance with movies and TV shows (test)
|

## Radio and Video Streaming

| Hostname / IP                                     | Remarks                                                  |
|:------------------------------------------------- |:-------------------------------------------------------- |
| http://stream.media.dn42/                         | icecast-relay, contact toBee for more streams            |
| https://invidious.doxz.dn42/                       | Invidious instance with proxy (Youtube)                  |

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
| http://seafile.dn42                                         |       |             | Opensource Dropbox, yay!                                                                        
| http://files.nop.dn42/                                      |       | max 1Mbit/s | download only                                  
| http://filer.mhm.dn42                                       |  4TB  | 1GBit       | 24/7/365                                                          

### Torrent Search Engine

- https://magnetic.dn42 (DHT Search Engine)

### BitTorrent tracker
- http://172.20.184.241/ (IPv4)
- http://[fd43:6d1:3ee2:1000:1]/ (IPv6)
- http://tracker.dn42/ (info page)

## Proxies

 See http://wiki.hamburg.ccc.de/ChaosVPN:Proxy

### Tor

Entry points to the Tor network are available on dn42. See [[Tor|internal/services/Tor]] for more details.

### Telegram

A MTProxy server is available at https://t.me/proxy?server=172.20.138.19&port=6667&secret=5bd3794d0fe2e00b985ffdac3fd17128.

## NTP

| Hostname / IP                                     | Remarks                             |
|:------------------------------------------------- |:----------------------------------- |
| ntp.martin89.dn42                                 | more than one A records/server      |
| tick.gotroot.dn42 (172.20.14.247)                 | Stratum 1, GPS, Vancouver Canada    |
| tock.gotroot.dn42 (172.20.14.250)                 | Stratum 2, Anycast on each node     |
| *.burble.dn42 | All burble.dn42 nodes are part of the NTP Pool and provide NTP over clearnet and DN42. See also [burble.dn42 services](https://dn42.burble.com/home/burble-dn42-services) |

## OS Mirror/Repository's

Repository Mirrors are listed on another page: [Repository Mirrors](/services/Repository-Mirrors)


## Gaming

| Hostname / IP                                     | Game                   | Remarks                    |
|:------------------------------------------------- |:---------------------- |:-------------------------- |
| hulk.mhm.dn42 (172.23.67.1)                       | Tetrinet               |                            |
| mc.nia.dn42 (172.20.168.131)                      | Minecraft              | 1.15.2, Optimized for CN   |

## Shell

Providers of shell access:

| Person        | Hostname                             | Net              | Description | Contact       |
|:------------- |:------------------------------------ |:---------------- |:----------- |:------------- |
| mc36          | telnet test.nop.dn42                  | dn42 only        |looking glass| -             |

## Misc 

| Hostname / IP                                     | Remarks                                                                        |
| ------------------------------------------------- | ------------------------------------------------------------------------------ | 
| http://teams.dn42[.us]/dn42                       | Mattermost (Slack clone) instance: get notifications for wiki/CA changes here  |
| http://nowhere.ws/dn42                            | Some random stuff concerning dn42, packages for Debian, e.g. Quagga        |
| https://paste.weiti.dn42                          | AES-encrypted pastebin-like (privatebin) |
| http://www.nop.dn42/                              | Basic "whatismyip" service               |
| http://freerouter.nop.dn42/                       | freeRouter main site                     |
| http://rtros.nop.dn42/                            | freeRouter distribution                  |
| https://git.dn42[.us]                             | Git Repository Hosting (Signup: email ssh pubkey to xuu@dn42.us)|               
| https://git.dn42[.us]/pubkeys/[username]          | Get ssh public keys from Git Users of git.dn42. |
| http://wiki.dn42, http://internal.dn42, [dn42.i2p](http://beb6v2i4jevo72vvnx6segsk4zv3pu3prbwcfuta3bzrcv7boy2q.b32.i2p/) (i2p), jsptropkiix3ki5u.onion  | This wiki! Web Hosted by [xuu](https://xuu.dn42). Git Repo hosted on git.dn42  |                
| http://jack.pyropeter.eu/dn42/routecount/         | Statistics about the number of v4/v6 routes seen by AS76115 (Since Aug. 2014)  |
| https://git.zotan.dn42                            | Git Repository Hosting, open signup (Powered by gitea)|               

### Usenet Servers / News
There are some News Servers available [here](/services/News)

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

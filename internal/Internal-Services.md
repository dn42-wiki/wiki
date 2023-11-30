# Internal services

You are asked to show some creativity in terms of network usage and content. ;)

## Search engine

There is a search engine at [search.dn42](https://search.dn42) that can also be used to discover services and content. It attempts to index all sites on DN42.

## Certificate Authority

xuu is maintaining an [certificate authority](/services/Certificate-Authority) for internal services.

n0emis maintains an [ACME server](https://acme.dn42) (with accompanying CA), compatible with any LetsEncrypt client like Certbot, Dehydrated or Caddy.

## Network-related

* See [Looking Glasses](/services/Looking-Glasses) for more network diagnostic tools
* Realtime network map: [map.dn42](http://map.dn42/) (DN42) or [map42.0x7f.cc](https://map42.0x7f.cc), [map.kuu.moe](https://map.kuu.moe) (IANA) _(Note: This is a direct copy of nixnodes map with some fixes and new functions since original map is no longer get maintained. This map is currently using MRT dump from GRC as source. We will pull new dumps from GRC every 15 minutes.)_
* Network Information Service: [info.nia.dn42](http://info.nia.dn42) (DN42) or [bgp42.strexp.net](https://bgp42.strexp.net) (IANA). Main functions including _network information_, _network map (from map.dn42, require WebGL)_, _network ranking (based on centrality)_, _ROA alerting_ and _path finder_.
* Yet Another network map: [map.jerry.dn42](https://map.jerry.dn42/) (DN42) or [map.meson.cc](https://map.meson.cc) (via clearnet) _(uses MRT dump as source, updated every 15 minutes.)_
* DN42 IP address lookup tool: [dn42.g-load.eu/lookup](https://dn42.g-load.eu/lookup/)
* Cloudflare-like cdn-cgi/trace: [map.jerry.dn42/cdn-cgi/trace](https://map.jerry.dn42/cdn-cgi/trace)
* New DNS System monitoring: [grafana.burble.com/d/E4iCaHoWk/dn42-dns-status](https://grafana.burble.com/d/E4iCaHoWk/dn42-dns-status?orgId=1&refresh=1m)
* whatsmyip:
    * ipv4+ipv6: [myip.dn42](http://myip.dn42/)
    * ipv4 only: [v4.myip.dn42](http://v4.myip.dn42/) or [172.20.0.81](http://172.20.0.81)
    * ipv6 only: [v6.myip.dn42](http://v6.myip.dn42/) or [fd42:d42:d42:81::1](http://[fd42:d42:d42:81::1]/)
    * API endpoints:
        * /raw: return your IP address as plain text
        * /api: JSON with your IP plus the details of the server you reached (location, ASN, etc.)
* Route Graphs: [routegraphs.highdef.dn42](http://routegraphs.highdef.dn42) (DN42), [routegraphs.highdef.network](https://routegraphs.highdef.network) (clearnet) - graph reachability from ASes to specific prefixes, using data from the dn42 GRC
* BGP flap detector (FlapAlertedPro by Kioubit) hosted by AS4242422092: https://flaps.pebkac.dn42

### GeoIP Services

[map.dn42](http://map.dn42/) provides a simple GeoIP service. This service uses rDNS records and country field in aut-num objects as a reference, currently designed for fun :P

#### API

Results are in JSON format.

```
http://ipip.map.dn42/whois?ip=[DN42_IP]&lang=en
http://ipip.map.dn42/whois?asn=AS[DN42_ASN]
```

#### Client

There is a client software using above apis to provide GeoIP-based traceroute.
It is a modified IPIP.NET Best Trace software with DN42 support injection.

Windows only, no virus scan report available, but our DLL source is provided with the modified client. It's highly recommended to run this tool in a sandbox.

**Since the original software is not open source, so use it at your own risk.**

Preview: <http://img.dn42/images/GEOTRACE42.jpg><br>
Link: <http://map.dn42/BestTrace42.zip>

### IX Services

|Name     |Wiki Page                          |AS Number |Related Link(s)                           |
|:--------|:----------------------------------|:---------|:-----------------------------------------|
|IXP-frnte|[IXP-frnte](/services/IXP-frnte.md)|4242421081|N/A                                       |
|IX2      |[IX2](/services/IX2.md)            |4242421951|N/A                                       |
|SERNET-IX|[SERNET-IX](/services/SERNET-IX.md)|4242422245|[SERNET-IX](https://blog.sherpherd.top/ix)|

### ASN Authentication Solution

Authenticate your users by having them verify their ASN ownership with KIOUBIT-MNT using their registry-provided methods in an automated way. An example of this is the automatic peering system for the Kioubit Network.
To use the service, please message Kioubit on IRC to have your domain activated and receive the required documentation.

## IRC

| Hostname / IP                 | SSL           | Remarks                                           |
|:------------------------------|:------------- |:--------------------------------------------------|
| irc.hackint.dn42              |  Yes          | DN42                                              |
| irc.hackint.hack/dn42         |  Yes          | ChaosVPN                                          |
| irc.dn42                      |  Yes          | Internal IRC                                      |
| irc.ty3r0x.dn42               |  Yes          | BonoboNET (ty3r0x.bnet)                           |
| irc.catgirls.dn42             |  Yes          | Karx IRC, clearnet karx.xyz/6697, dn42 v6 only    |

### Clients

| Hostname / IP | Remarks |
|:--------------|:--------|
| <https://lounge.burble.dn42> | [thelounge](https://thelounge.chat/) for lurking on #dn42, see [burble.dn42 services](https://dn42.burble.com/services/internal/#loungeburbledn42). |
| <https://irc.pebkac.dn42> | [thelounge](https://thelounge.chat/) for lurking on #dn42, ask TOMKAP-DN42 for an account |

## Images, E-Books, Videos and other Media

| Hostname / IP                                     | Remarks                                                  |
|:------------------------------------------------- |:-------------------------------------------------------- |
| <http://j.munsternet.dn42>                        | Jellyfin instance with movies and TV shows (test).       |

## Radio and Video Streaming

| Hostname / IP                                     | Remarks                                                        |
|:------------------------------------------------- |:-------------------------------------------------------------- |
| <https://live.jerry.dn42/>                        | Live audio stream powered by mpd                               |
| <http://radio.vr18.dn42/>                         | Online radio by vr18                                           |
| <https://dn42:dn42@tv.munsternet.dn42/playlist>   | TV Channels Streaming                                          |
| <http://icy.jones.dn42>                           | Home grown Icecast Radio covering a number of genres (HLS & Player coming soon [ish]!)          |
| <http://radio.vrpnet.org:8000/radio.ogg>          | Online radio by Pi3rrot on VRPNET                             |
| <http://webdj.nop.dn42/>                            | controller for Multicast stream: rtp://172.23.199.110@232.2.3.2:1234/ |
| <https://sdr.pebkac.dn42/>                          | OpenWebRX SDR Receiver, FM/VHF/UHF Analog & Digital |

## File Sharing

### FTP / HTTP

**FIXME**: Please add info about (approximate) bandwidth of the servers.

Repository Mirrors are listed on another page: [Repository Mirrors](/services/Repository-Mirrors)

| Hostname / IP                                       | Remarks                                        |
|:--------------------------------------------------- | ---------------------------------------------- |
| <http://files.nop.dn42>                             | download only, max 1Mbit/s                     |
| <http://rfc-editor.dn42>                            | download only, max 1Mbit/s                     |
| <http://freertr.dn42/>                              | freeRouter main site                           |
| <http://sources.nop.dn42>                           | freeRouter source tree                         |
| <http://rtros.nop.dn42/>                            | freeRouter distribution                        |

## VPN

DN42 Network Access over Automatic Wireguard VPN Service (IPv6 only, fd00::/8)
provided by TheQ at <https://dn42.0011.de/vpnusers>

## Proxies

See <http://wiki.hamburg.ccc.de/ChaosVPN:Proxy>

### Telegram

A MTProxy server is available at [mtp.jerry.dn42:8044](https://t.me/proxy?server=mtp.jerry.dn42&port=8044&secret=ee1419944c0a129dbba2beb2636fcf361a616e64726f69642e676f6f676c65736f757263652e636f6d).

## NTP

| Hostname / IP                                     | Remarks                             |
|:------------------------------------------------- |:----------------------------------- |
| *.burble.dn42 | All burble.dn42 nodes provide NTP over clearnet and DN42. See also [burble.dn42 public services](https://dn42.burble.com/services/public/) |

## Gaming

| Hostname / IP                                     | Game                   | Remarks                    |
|:------------------------------------------------- |:---------------------- |:-------------------------- |
| yuan.nia.dn42 (172.20.168.244)                    | Genshi Impact (Unofficial) | Latest development branch, Scripts included, Optimized for CN, Not stable yet  |
| mc.nico.dn42 | Minecraft | 1.16.5, [Forge Modded](https://bbs.dn42/d/17-modded-116-minecraft-server), IPv4 & IPv6, Central US Server |
| mc.jerry.dn42 & jerry.dn42                        | Minecraft              | latest, IPv4 & IPv6 |
| mc.razuritta.dn42                             | Minecraft | latest(1.20.1 atm), IPv4 and IPv6, map at https://mcmap.razuritta.dn42/ |
| mc.northrend.dn42 (172.20.222.240)		    | Minecraft		     | latest, IPv4 only   |
| redtrap.northrend.dn42 (172.20.222.252)	    | Minecraft		     | RedTrap DN42 Bridge, 1.8.8 - 1.19, IPv4 only   |
| ttd.jerry.dn42                                    | OpenTTD                | latest, IPv4 & IPv6 |
| stk.jerry.dn42:2759, stk.jerry.neo:2759           | SuperTuxKart           | latest, IPv4 only   |
| stk2.jerry.dn42:2759, stk2.jerry.neo:2759         | SuperTuxKart           | latest, IPv4 & IPv6 |
| factorio.catgirls.dn42 (IPv6 only)                | factorio               | Still in testing, expect downtime |
| The burble.dn42 shell servers include a number of classic text games, see [shell access](https://dn42.burble.com/services/shell/#classic-games) | Various | Log in to the shell servers for more |
| telnet tetris.dn42                                | tetris in your terminal| other games available on request, ping mc36 @ irc |
| <https://clicker.burble.dn42/>                    | Clicker/Idle           | Waste your time away with a dn42 themed browser based idle game |
| [monkic.mk16.de](https://monkic.mk16.de/), [dn42](https://monkic.bandura.dn42/) | Monkic (Game in German) |

## Voice chat

|Hostname / IP         | Remarks              |
|:---------------------|:---------------------|
| mumble://ty3r0x.dn42:64738 | Ty3r0X's Lair (men's club) |

## VOIP/SIP Endpoints

* burble.dn42 runs an [asterisk based VOIP service](https://dn42.burble.com/services/public/#voip) with various test extensions and real hardware modems for dialing in to dn42
* jerry.dn42 also runs an [asterisk based VOIP service](https://blog.jerry.dn42/dn42#Services_pbx) with live radio (see live.jerry.dn42 above), whois service, conference room and software modems for dialing in to dn42

## Challenges

Test out your skills with online challenges

| Start here                                        | Remarks                            |
|:------------------------------------------------- |:---------------------------------- |
| <https://burble.dn42/services/ping/>              | burble.dn42 ping challenge         |
| <http://kioubit.dn42/challenge/>                  | Kioubit.dn42 challenge             |

## Shell

Providers of shell access:

| Person        | Hostname                               | Net              | Description | Contact       |
|:------------- |:-------------------------------------- |:---------------- |:---------------- |:------------- |
| mc36          | `telnet test.nop.dn42`                   | dn42 only        |looking glass     | -             |
| JerryXiao     | `ssh lg@lg.jerry.dn42`                   | dn42 and icvpn   |looking glass     | -             |
| burble        | `ssh <mntner>@shell.fr-rbx1.burble.dn42` <br/> `ssh <mntner>@shell.ca-bhs2.burble.dn42` | dn42             | Full shell account | See below |

### burble.dn42 shell access

Full shell accounts are available for all dn42 users. Usernames are
constructed using the MNTNER name, lowercased and without the '-MNT' suffix. SSH public keys are imported automatically from the registry or alternatively, a password can be used instead.

See also the [burble.dn42 website](https://dn42.burble.com/services/shell/) for more details.

## Personal/network/blogs websites

| Website                                     | Description                                                                        |
| ------------------------------------------------- | ------------------------------------------------------------------------------ |
| <https://burble.dn42/>                              | burble.dn42 website |
| <http://www.marlinc.dn42/>                          | Marlinc website     |
| <http://vr18.dn42/>                                 | vr18.dn42 website   |
| <https://mk16de.bandura.dn42/> | Marek's site |
| <http://lpnet0.dn42/>                              | LAUNCHPAD-NETWORK official website |

## Pastebins

| Hostname / IP                                     | Remarks                                                                        |
| ------------------------------------------------- | ------------------------------------------------------------------------------ |
| <http://paste.nop.dn42>                             | yet another paste service                                        |
| <https://p.pebkac.dn42/>                            | PasteBin Service (Netcat/Bash CLI Client) |

## Forums / Message boards

| Hostname / IP                                     | Remarks                                                                        |
| ------------------------------------------------- | ------------------------------------------------------------------------------ |
| <https://urandom.catgirls.dn42/>                    | Message board |
| ------------------------------------------------- | ------------------------------------------------------------------------------ |
| <https://social.dn42/>                    | Pleroma Instance |

## Misc

| Hostname / IP                                     | Remarks                                                                        |
| ------------------------------------------------- | ------------------------------------------------------------------------------ |
| <http://wiki.dn42>, <http://internal.dn42>          | This wiki!  Git Repo hosted on git.dn42
| <http://www.nop.dn42/>                              | Basic "whatismyip" service                                       |
| <http://fun.nop.dn42/>                              | some funny images and videos                                     |
| <http://pvrp.nop.dn42/>                             | a path vector igp main site                                      |
| <http://lsrp.nop.dn42>                              | a link state igp main site                                       |
| <http://hwp0rn.nop.dn42>                            | girls with switches and routers, a hwpr0n.se mirror              |
| <http://ix.nop.dn42>                                | mcast-ix main site                                               |
| <http://mpls.dn42/>                                 | a brief description of MPLS technology                           |
| <http://speed.vr18.dn42/>                           | vr18 speedtest (100/20 mbit internet)                            |
| <http://it.vr18.dn42/>                              | handy tools for developers                                       |
| [dn42](https://draw.bandura.dn42/),[NeoNetwork](http://draw.bandura.neo/), [CRXN](http://draw.bandura.crxn/) | Excalidraw instance |
| <https://ntfy.weil-isso.dn42>                       | NTFY (Push Notify via REST-API)                                   |
| kms.weil-isso.dn42 (Port 1688)                      | Key Management Server (with Auto-Activation) for any Windows/Office (LAB AND TEST Purposes only) |

## Usenet Servers / News

There are some News Servers available [here](/services/News)

## E-Mail

There is a list of E-Mail providers [here](/services/E-Mail-Providers)

# Other networks

**[List of other Overlay Networks](/Other)**

## Public Internet

* <https://mirror.frubar.net> 100MBit
* <https://frucman.frubar.net>

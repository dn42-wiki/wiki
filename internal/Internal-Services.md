# Internal services

You are asked to show some creativity in terms of network usage and content. ;)

## Search engine

There is a search engine at [search.dn42](https://search.dn42) that can also be used to discover services and content. It attempts to index all sites on DN42.

A searx-ng instance is available at [search.androw.dn42](https://search.androw.dn42) to search the clearnet.

## Certificate Authority

xuu is maintaining an [certificate authority](/services/Certificate-Authority) for internal services.

Burble maintains an [ACME server](https://burble.dn42/services/acme/) (with accompanying CA), compatible with any LetsEncrypt client like Certbot, Dehydrated or Caddy.

> \*\*\*Currently Down as of 12/18/23 <br>
> n0emis maintains an [ACME server](https://acme.dn42) (with accompanying CA), compatible with any LetsEncrypt client like Certbot, Dehydrated or Caddy.

## Network-related

- See [Looking Glasses](/services/Looking-Glasses) for more network diagnostic tools
- Realtime network map: [map.dn42](https://map.dn42/) (DN42) or [map.iedon.net](https://map.iedon.net) (via clearnet) _(This map currently uses MRT dumps from GRC as its source. It refreshes whenever the collector provides a new MRT file, generally 10-15 minutes.)_
- Network Information Service: [info.nia.dn42](http://info.nia.dn42) (DN42) or [bgp42.strexp.net](https://bgp42.strexp.net) (via clearnet). Main functions including _network information_, _network map (from map.dn42, require WebGL)_, _network ranking (based on centrality)_, _ROA alerting_ and _path finder_.
- Yet Another network map: [map.jerry.dn42](https://map.jerry.dn42/) (DN42) or [map.meson.cc](https://map.meson.cc) (via clearnet) _(uses MRT dump as source, updated every 15 minutes.)_
- Various DN42-related tools: [dn42.g-load.eu/toolbox/](https://dn42.g-load.eu/toolbox/)
- Cloudflare-like cdn-cgi/trace: [map.jerry.dn42/cdn-cgi/trace](https://map.jerry.dn42/cdn-cgi/trace)
- New DNS System monitoring: [grafana.burble.com/d/E4iCaHoWk/dn42-dns-status](https://grafana.burble.com/d/E4iCaHoWk/dn42-dns-status?orgId=1&refresh=1m)
- whatsmyip:
  - ipv4+ipv6: [myip.dn42](http://myip.dn42/)
  - ipv4 only: [v4.myip.dn42](http://v4.myip.dn42/) or [172.20.0.81](http://172.20.0.81)
  - ipv6 only: [v6.myip.dn42](http://v6.myip.dn42/) or [fd42:d42:d42:81::1](http://[fd42:d42:d42:81::1]/)
  - API endpoints:
    - /raw: return your IP address as plain text
    - /api: JSON with your IP plus the details of the server you reached (location, ASN, etc.)
- Route Graphs: [routegraphs.highdef.dn42](http://routegraphs.highdef.dn42) (DN42), [routegraphs.highdef.network](https://routegraphs.highdef.network) (clearnet) - graph reachability from ASes to specific prefixes, using data from the dn42 GRC
- BGP flap detector (FlapAlertedPro by Kioubit) hosted by AS4242422092: [flaps.pebkac.dn42](https://flaps.pebkac.dn42)

### Speedtest

| Operator   | Location  | Max Speed   | Link                                                         |
| :--------- | :-------- | :---------- | ------------------------------------------------------------ |
| Burble     | Anycast   | Unknown     | [speedtest.burble.dn42](https://speedtest.burble.dn42/)      |
| Kioubit    | Nuremberg | 400 Mbps    | [kioubit.dn42/speedtest/](https://kioubit.dn42/speedtest/)   |
| Routedbits | Multiple  | Unknown     | [dn42.routedbits.io/nodes](https://dn42.routedbits.io/nodes) |
| vr18       | Unknwon   | 100/20 Mbps | [speed.vr18.dn42/](http://speed.vr18.dn42/)                  |

### Map.dn42 API Services

Data refreshes whenever the collector provides a new MRT file, generally 10-15 minutes.

| API              | URL                                                                                                                                                            | Description                                                                                                                                                              |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------- |
| `/map`           | [https://map.dn42/map](https://map.dn42/map) <br> [https://api.iedon.com/dn42/map](https://api.iedon.com/dn42/map)                                             | Raw binary map data                                                                                                                                                      |
| `/map?type=json` | [https://map.dn42/map?type=json](https://map.dn42/map?type=json) <br> [https://api.iedon.com/dn42/map?type=json](https://api.iedon.com/dn42/map?type=json)     | Raw map data, in JSON                                                                                                                                                    |
| `/asn/{asn}`     | [https://map.dn42/asn/4242422189](https://map.dn42/asn/4242422189) <br> [https://api.iedon.com/dn42/asn/4242422189](https://api.iedon.com/dn42/asn/4242422189) | Real-time node information from the map, including WHOIS data, in JSON                                                                                                   |
| `/ranking`       | [https://map.dn42/ranking](https://map.dn42/ranking) <br> [https://api.iedon.com/dn42/ranking](https://api.iedon.com/dn42/ranking)                             | DN42 Global Ranking, based on the map.dn42 index                                                                                                                         |
| `/myip/[raw      | api]`                                                                                                                                                          | [https://map.dn42/myip/](https://map.dn42/myip/) <br> [https://map.dn42/myip/api](https://map.dn42/myip/api) <br> [https://map.dn42/myip/raw](https://map.dn42/myip/raw) | What's My IP instance by map.dn42, including `netname` and `country` information if available |

### IX Services

| Name      | Wiki Page                        | AS Number  | Related Link(s)                            |
| :-------- | :------------------------------- | :--------- | :----------------------------------------- |
| IXP-frnte | [IXP-frnte](/services/IXP-frnte) | 4242421081 | N/A                                        |
| mcast-ix  | [mcast-ix](/services/mcast-ix)   | 4242421951 | N/A                                        |
| SERNET-IX | [SERNET-IX](/services/SERNET-IX) | 4242422245 | [SERNET-IX](https://blog.sherpherd.top/ix) |

### ASN Authentication solution

Authenticate your users by having them verify their ASN ownership with KIOUBIT-MNT using their registry-provided methods in an automated way. An example of this is the automatic peering system for the Kioubit Network. [Documentation](https://dn42.g-load.eu/about/authentication-services/)

## IRC

| Hostname / IP         | SSL | Remarks                                                                                                             |
| :-------------------- | :-- | :------------------------------------------------------------------------------------------------------------------ |
| irc.hackint.dn42      | Yes | DN42                                                                                                                |
| irc.hackint.hack/dn42 | Yes | ChaosVPN                                                                                                            |
| irc.dn42              | Yes | Internal IRC                                                                                                        |
| irc.ty3r0x.dn42       | Yes | BonoboNET (ty3r0x.bnet)                                                                                             |
| irc.catgirls.dn42     | Yes | Karx IRC, clearnet karx.xyz/6697, dn42 v6 only                                                                      |
| irc.replirc.dn42      | Yes | ReplIRC (started in 2022 on Replit (but no longer uses Replit), uses Solanum for servers), self-signed certificates |

### Clients

| Hostname / IP                | Remarks                                                                                                                                             |
| :--------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------- |
| <https://lounge.burble.dn42> | [thelounge](https://thelounge.chat/) for lurking on #dn42, see [burble.dn42 services](https://dn42.burble.com/services/internal/#loungeburbledn42). |
| <https://irc.pebkac.dn42>    | [thelounge](https://thelounge.chat/) for lurking on #dn42, ask TOMKAP-DN42 for an account                                                           |

## Images, E-Books, Videos and other Media

| Hostname / IP              | Remarks                                            |
| :------------------------- | :------------------------------------------------- |
| <http://j.munsternet.dn42> | Jellyfin instance with movies and TV shows (test). |

## Radio and Video Streaming

| Hostname / IP                                   | Remarks                                                                                |
| :---------------------------------------------- | :------------------------------------------------------------------------------------- |
| <https://live.jerry.dn42/>                      | Live audio stream powered by mpd                                                       |
| <http://radio.vr18.dn42/>                       | Online radio by vr18                                                                   |
| <https://dn42:dn42@tv.munsternet.dn42/playlist> | TV Channels Streaming                                                                  |
| <http://icy.jones.dn42>                         | Home grown Icecast Radio covering a number of genres (HLS & Player coming soon [ish]!) |
| <https://radio.vrpnet.dn42>                     | Online radio by Pi3rrot on VRPNET, using Icecast&Liquidsoap                            |
| <http://webdj.nop.dn42/>                        | controller for Multicast stream: rtp://172.23.199.110@232.2.3.2:1234/                  |
| <https://sdr.pebkac.dn42/>                      | OpenWebRX SDR Receiver, FM/VHF/UHF Analog & Digital (ask TOMKAP-DN42 for an account)   |
| <https://adsb.androw.dn42/>                     | ADS-B Receiver located in Paris                                                        |
| <http://kz.dn42:8888/6MusicProxy/index.m3u8>    | BBC Radio 6 Music Proxy service (320kbps AAC streams over HLS)                         |

## File Sharing

### FTP / HTTP

**FIXME**: Please add info about (approximate) bandwidth of the servers.

Repository Mirrors are listed on another page: [Repository Mirrors](/services/Repository-Mirrors)

| Hostname / IP             | Remarks                    |
| :------------------------ | -------------------------- |
| <http://files.nop.dn42>   | download only, max 1Mbit/s |
| <http://rfc-editor.dn42>  | download only, max 1Mbit/s |
| <http://freertr.dn42/>    | freeRouter main site       |
| <http://sources.nop.dn42> | freeRouter source tree     |
| <http://rtros.nop.dn42/>  | freeRouter distribution    |

## VPN

DN42 Network Access over Automatic Wireguard VPN Service (IPv6 only, fd00::/8)
provided by TheQ at <https://dn42.0011.de/vpnusers>

## Proxies

See <http://wiki.hamburg.ccc.de/ChaosVPN:Proxy>

### Telegram

A DN42 private Telegram app and server is available at [buzzgram.dn42](https://buzzgram.dn42), the app requires a new registration (Official Telegram account isn't valid), only support IPv6 at the moment.

A MTProxy server is available at [mtp.jerry.dn42:8044](https://t.me/proxy?server=mtp.jerry.dn42&port=8044&secret=ee1419944c0a129dbba2beb2636fcf361a616e64726f69642e676f6f676c65736f757263652e636f6d).

## NTP

| Hostname / IP  | Remarks                                                                                                                                    |
| :------------- | :----------------------------------------------------------------------------------------------------------------------------------------- |
| \*.burble.dn42 | All burble.dn42 nodes provide NTP over clearnet and DN42. See also [burble.dn42 public services](https://dn42.burble.com/services/public/) |

## Gaming

| Hostname / IP                                                                                                                                   | Game                       | Remarks                                                                                              |
| :---------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------- | :--------------------------------------------------------------------------------------------------- |
| yuan.nia.dn42 (172.20.168.244)                                                                                                                  | Genshi Impact (Unofficial) | Latest development branch, Scripts included, Optimized for CN, Not stable yet                        |
| 172.20.229.250                                                                                                                                  | Minecraft                  | Vanilla Server - latest(1.21.4)                                                                      |
| mc.jerry.dn42 & jerry.dn42                                                                                                                      | Minecraft                  | latest, IPv4 & IPv6                                                                                  |
| mc.razu.neo (formerly mc.razuritta.dn42)                                                                                                        | Minecraft                  | latest(1.20.4 atm), IPv4+IPv6                                                                        |
| mc.licen777.dn42 (172.23.197.197 & fde8:2e18:e127::300)                                                                                         | Minecraft                  | 1.16.5-1.21.3, (/gm 1, Cracked)                                                                      |
| fd11:ef8b:72ac::                                                                                                                                | Minecraft                  | 1.8                                                                                                  |
| ttd.jerry.dn42                                                                                                                                  | OpenTTD                    | latest, IPv4 & IPv6                                                                                  |
| stk.jerry.dn42:2759, stk.jerry.neo:2759                                                                                                         | SuperTuxKart               | latest, IPv4 only                                                                                    |
| stk2.jerry.dn42:2759, stk2.jerry.neo:2759                                                                                                       | SuperTuxKart               | latest, IPv4 & IPv6                                                                                  |
| factorio.catgirls.dn42 (IPv6 only)                                                                                                              | factorio                   | Still in testing, expect downtime                                                                    |
| nodecore.pivipi.dn42                                                                                                                            | Minetest                   | IPv6 only                                                                                            |
| The burble.dn42 shell servers include a number of classic text games, see [shell access](https://dn42.burble.com/services/shell/#classic-games) | Various                    | Log in to the shell servers for more                                                                 |
| telnet tetris.dn42                                                                                                                              | tetris in your terminal    | other games available on request, ping mc36 @ irc                                                    |
| [https://clicker.burble.dn42/](https://clicker.burble.dn42/)                                                                                    | Clicker/Idle               | Waste your time away with a dn42 themed browser based idle game                                      |
| [monkic.mk16.de](https://monkic.mk16.de/), [dn42](https://monkic.bandura.dn42/)                                                                 | Monkic (Game in German)    |
| yobanirot.dn42:7591                                                                                                                             | Minecraft and Minetest     | One more "tech" minecraft server, mods [here](http://yobanirot.dn42/mods.zip). Minetest on same port |

## Voice chat

| Hostname / IP              | Remarks                    |
| :------------------------- | :------------------------- |
| mumble://ty3r0x.dn42:64738 | Ty3r0X's Lair (men's club) |

## VOIP/SIP Endpoints

- burble.dn42 runs an [asterisk based VOIP service](https://dn42.burble.com/services/public/#voip) with various test extensions and real hardware modems for dialing in to dn42
- jerry.dn42 also runs an [asterisk based VOIP service](https://blog.jerry.dn42/dn42#Services_pbx) with live radio (see live.jerry.dn42 above), whois service, conference room and software modems for dialing in to dn42
- buzzster.dn42 runs an asterisk based VOIP service with shorts numbers, inbound and outbound calls and live radio. Request your short number on [KevvoGeek Telegram](https://t.me/KevvoGeek) for make and receive internal and external calls.

## Challenges

Test out your skills with online challenges

| Start here                           | Remarks                       |
| :----------------------------------- | :---------------------------- |
| <https://burble.dn42/services/ping/> | burble.dn42 ping challenge    |
| <http://kioubit.dn42/challenge/ch2/> | Kioubit.dn42 challenge (2024) |
| <http://kioubit.dn42/challenge/ch1/> | Kioubit.dn42 challenge (2023) |

## Shell

Providers of shell access:

| Person    | Hostname                                                                                | Net            | Description        | Contact   |
| :-------- | :-------------------------------------------------------------------------------------- | :------------- | :----------------- | :-------- |
| mc36      | `telnet test.nop.dn42`                                                                  | dn42 only      | looking glass      | -         |
| JerryXiao | `ssh lg@lg.jerry.dn42`                                                                  | dn42 and icvpn | looking glass      | -         |
| burble    | `ssh <mntner>@shell.fr-rbx1.burble.dn42` <br/> `ssh <mntner>@shell.ca-bhs2.burble.dn42` | dn42           | Full shell account | See below |

### burble.dn42 shell access

Full shell accounts are available for all dn42 users. Usernames are
constructed using the MNTNER name, lowercased and without the '-MNT' suffix. SSH public keys are imported automatically from the registry or alternatively, a password can be used instead.

See also the [burble.dn42 website](https://dn42.burble.com/services/shell/) for more details.

## Personal/network/blogs websites

| Website                        | Description                                       |
| ------------------------------ | ------------------------------------------------- |
| <https://burble.dn42/>         | burble.dn42 website                               |
| <http://www.marlinc.dn42/>     | Marlinc website                                   |
| <http://vr18.dn42/>            | vr18.dn42 website                                 |
| <https://mk16de.bandura.dn42/> | Marek's site                                      |
| <http://lpnet0.dn42/>          | LAUNCHPAD-NETWORK official website                |
| <http://blog.sherpherd.dn42/>  | Hawkins' Homepage                                 |
| <https://20plays.dn42/>        | 20plays' website                                  |
| <https://mdr.dn42/>            | Mark Dastmalchi-Round's website (clearnet mirror) |

## Pastebins

| Hostname / IP            | Remarks                                   |
| ------------------------ | ----------------------------------------- |
| <http://paste.nop.dn42>  | yet another paste service                 |
| <https://p.pebkac.dn42/> | PasteBin Service (Netcat/Bash CLI Client) |

## Forums / Message boards

| Hostname / IP                                             | Remarks                                                                             |
| --------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| <https://urandom.catgirls.dn42/>                          | Message board                                                                       |
| -------------------------------------------------         | ------------------------------------------------------------------------------      |
| <https://bbs.dn42>, <https://dn42bbs.0b1.me> via Clearnet | A general BBS powered by Flarum for virtually any topics. Maintained by nicholascw. |

## Misc

| Hostname / IP                              | Remarks                                                                                          |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------ |
| <http://wiki.dn42>, <http://internal.dn42> | This wiki! Git Repo hosted on git.dn42                                                           |
| <http://www.nop.dn42/>                     | Basic "whatismyip" service                                                                       |
| <http://fun.nop.dn42/>                     | some funny images and videos                                                                     |
| <http://pvrp.nop.dn42/>                    | a path vector igp main site                                                                      |
| <http://lsrp.nop.dn42>                     | a link state igp main site                                                                       |
| <http://hwp0rn.nop.dn42>                   | girls with switches and routers, a hwpr0n.se mirror                                              |
| <http://ix.nop.dn42>                       | mcast-ix main site                                                                               |
| <http://mpls.dn42/>                        | a brief description of MPLS technology                                                           |
| <http://speed.vr18.dn42/>                  | vr18 speedtest (100/20 mbit internet)                                                            |
| <http://it.vr18.dn42/>                     | handy tools for developers                                                                       |
| <https://ntfy.weil-isso.dn42>              | NTFY (Push Notify via REST-API)                                                                  |
| kms.weil-isso.dn42 (Port 1688)             | Key Management Server (with Auto-Activation) for any Windows/Office (LAB AND TEST Purposes only) |
| anycast.zookeeper.immibis.dn42 (port 2181) | Apache Zookeeper cluster (requires a client certificate for access - on request)                 |

## Usenet Servers / News

There are some News Servers available [here](/services/News)

## E-Mail

There is a list of E-Mail providers [here](/services/E-Mail-Providers)

# Other networks

**[List of other Overlay Networks](/Other)**

## Public Internet

- <https://mirror.frubar.net> 100MBit
- <https://frucman.frubar.net>

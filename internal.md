# Internal services

You are asked to show some creativity in terms of network usage and content. ;)

More ideas inspiration is collected on another [page](/ideas).

[[_TOC_]]

## Network-related

  * Polynome has some nice scripts and visualizations here: http://dataviz.polynome.dn42
  * http://172.23.174.1
  * See [[Looking Glasses]] for more network diagnostic tools

### DNS tunnel

This DNS tunnel service uses [Iodine](http://code.kryo.se/iodine/), and provides access to the dn42 network. Useful when you're on a shitty network (airport, train station) that still allows DNS.

Use the anycast DNS servers (172.22.0.53) inside your tunnel.

| Hostname    | Router IP      | Password |
|:----------- |:-------------- |:-------- |
| t.polyno.me | 172.23.185.193 | dn42     |

### DNS Tools

This tool allows you to lookup your dn42 domain name and check to see if your name servers are all working and have the correct information.

Select "Disable Recursion" to check only entries found in the registry or leave it off to check all (both are useful tests).

Currently this system only supports IPv4.

http://mwd.dn42/dns.php

MWD will also provide a secondary DNS server and/or cacti monitoring of your devices. Just ask on IRC. More info: http://mwd.dn42

## IRC

| Hostname                                       | IP          | Remarks  |
|:---------------------------------------------- |:----------- |:-------- |
|[irc.hackint.dn42](irc://irc.hackint.dn42/dn42) | 172.22.24.1 | DN42     |
|[irc.hackint.hack](irc://irc.hackint.hack/dn42) | 172.31.0.30 | ChaosVPN |

## Search engine

 * [Web search engine](http://search.dn42) (172.23.184.1) - a few chosen HTTP domains are crawled  (taken from the wiki). The previous method, "crawl everything available from the wiki", generated too much data because of FTPs.

## Radio and Video Streaming

| Hostname / IP                             | Remarks                                  |
|:----------------------------------------- |:---------------------------------------- |
| http://10.11.10.30:8000                   | Freimusik
| http://stream.laxu.dn42:8000              | [xenim Streams](http://streams.xenim.de)
| http://sprawl.smrsh.dn42:8000/            | [smrsh radio](http://smrsh.net/radio)
| http://10.112.0.6:8000/mpd.ogg, http://radio.ffhh:8000/mpd.ogg | Freifunk Hamburg radio, yeay 8bit music!

## File sharing

**FIXME**: Please add info about (approximate) bandwidth of the servers.

### FTP / HTTP

| Hostname / IP                                 | Space | Speed | Remarks                    |
|:--------------------------------------------- |:----- |:----- |:-------------------------- |
| http://filer.nihilus.dn42, http://172.22.92.2 |       | ~60kbps | mostly up
| ftp://cochimetl.tim.dn42, nfs://cochimetl.tim.dn42/data/ftp | ~3TB | ~700kbps
| http://seafile.dn42                           |       |       | Opensource Dropbox, yay!   
| ftp://descent.derf.dn42 (172.23.225.35)       |  3TB  | 60kbit/s | download only
| http://files.feuerrot.dn42                    |  6TB  | 1Gbit | http, ftp, nfs, rsync
| ftp://vsynology.dev.ffc (10.8.6.13)           | 150G  | 20Mbit/s | just drop your nzb/torrent file and be patient; clean up your stuff!
| http://filer1.grmml.dn42 (172.23.149.21)     |  4TB  | 200Mbit/s | download only
| http://10.196.0.100:18455                     | 9 TB  |       | only download
| sftp://anonsftp:Iich0zieC3retaid@files.crest.dn42:2212/ | 12TB | 1Gb/s | incoming writable |

#### Down?

| Hostname / IP                                 | Space | Speed | Remarks                    |
|:--------------------------------------------- |:----- |:----- |:-------------------------- |
| http://turing.il.maxx.dn42, http://172.22.42.2 | ~6.5TB | ~400kbit | WebDAV enabled, up 24/7z

## Images and Media

| Hostname / IP                                  | Remarks                       |
|:---------------------------------------------- |:----------------------------- |
| http://img.dn42                                | Imagehoster                   |
| http://chan.dn42                               | DN42-Chan, an imageboard      |
| http://media.dn42                              | A Mediagoblin instance (Login: dn42:dn42dn42)       |
reachable from inside DN42

## Proxies

 See http://wiki.hamburg.ccc.de/ChaosVPN:Proxy

### Tor

| Hostname / IP                  | Bandwidth          |             Nickname        |
| ------------------------------ | ------------------ | ------------ |
| socks5://lian.0l.dn42:9050     | 600 kb/s | [nulll](https://atlas.torproject.org/#details/84F41A116AD7F1E038781413E0B4ADE4494BA38A)

### Hochschulbibliothekszentrum des Landes Nordrhein-Westfalen
Bodems (AS76124) is announcing 193.30.112.0/24 via his DFN-Node, so you can access the "[Digibib](http://www.digibib.net/jumpto?LOCATION=Bi10&D_SERVICE=TEMPLATE&D_SUBSERVICE=DIGILINK_BROWSE&DP_FUNC=CategoryView&DP_FILTER=All&DP_CID=14211)" through DN42 with a valid IP. For some parts (like VDE norms) you will need Citrix Receiver.

## NTP

| Hostname / IP                                  | Remarks                       |
|:---------------------------------------------- |:----------------------------- |
| ntp.e-utp.dn42 (172.22.165.50)         | Stratum 1, GPS+NMEA
| ntp1.nixnodes.dn42 (172.22.177.123)    |
| ntp2.nixnodes.dn42 (172.22.177.124)    | 

## Crypto coins

| Hostname / IP                                  | Remarks                       |
|:---------------------------------------------- |:----------------------------- |
| bitcoin.e-utp.dn42 (RR: 172.22.165.50, 172.22.165.34) | 8333 for Bitcoin, 9333 for Litecoin

## Misc 

| Hostname / IP                                             | Remarks                        |
| --------------------------------------------------------- | ------------------------------ | 
| http://nowhere.ws/dn42                                    | Some random stuff concerning dn42, packages for Debian, e.g. Quagga 
| https://paste.synhacx.dn42          | AES-encrypted pastebin-like ([zerobin](https://github.com/sebsauvage/ZeroBin))
| http://ip.synhacx.dn42 | Basic "whatismyip" service ([description](http://synhacx.dn42/showmyip))
| http://tor.mirror.martin89.dn42                           | Tor Project Homepage mirror
| http://debian.mirror.martin89.dn42                        | Debian Wheezy mirror
| nntp://news.blacksheep.dn42                               | Martin's newsgroup server (ping MB-DN42 for a rw account or a nntp/uucp feed)
| mumble://shard.smrsh.dn42:64738                           | [Mumble](http://mumble.sourceforge.net/) Voice Chat
| http://wiki.dn42, http://internal.dn42 | This wiki! Web Hosted by [xuu](https://xuu.dn42). Git Repo hosted by welterde |

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
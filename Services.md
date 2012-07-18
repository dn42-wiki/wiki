# TOP SECRET 

# Services 

You are asked to show some creativity in terms of network usage (content). ;)

## Chat

 * [hackint](irc://172.22.24.1/dn42)
 * [alice (links to public chat cloud)](irc://172.22.43.1/dn42)
 * [NNNC (links to public chat cloud)](irc://irc6.srn.dn42/dn42)

## Outback adventures 

 * [Iodine (DNS tunnel)](http://iodine.amadeus.dn42/)
 * [SMS gateway](http://sms.amadeus.dn42/)

## Search Engine 

 * [[http://yacy.script.dn42:8080/]] - internal dn42 search engine [alpha status, contact script@hackint for feedback], also indexes *.ffa

## File sharing 

 * [[http://ahuizotl.tim.dn42]] - ffsearch (currently down)

### NFS 

 * "mount -t nfs 172.22.45.3:/home/jchome/data" (rw for all)
 * "mount -t nfs 172.22.94.65:/media/media" (currently down until I have internet access again)
 * (currently down) "mount -t nfs 172.22.4.1:/home/gvu" (ro, 100Mbit, torrent-dumpsite, contact me for anything, latest episodes of some assorted tv-series ([http://172.22.4.1:8090/0-TV-Series 0-TV])) 

### FTP 

 * [[ftp://ftp.crest.dn42]] - 1.5TB to share. ro for all + upload. (slow, but up)
 * [[ftp://172.22.152.2]] - 100 GB DSL 16k up at day, down at night.
 * [[ftp://server0.adiblol.dn42/shared/music_hq/]] - read-only, 32GiB of FLAC lossless music - mainly grunge and rock. Streaming allowed! 4Mbps
 * [[ftp://filer.nihilus.dn42]] - [[ftp://172.22.92.2]] - ro for all + upload

### HTTP 
 
 * [[http://filer0.helios.dn42/]] - currently 686GB; only download. And just by IP http://172.22.195.17 (up again soon (technical issues); contact helios to get access.)
 * (currently down) [[http://172.22.4.1:8090/]], [[http://linus15.htwm.spaceboyz.net:8090/]] (v6 only), WebDAV enabled, 100MBit, torrent-dumpsite, contact me for anything, latest episodes of some assorted TV-Series ([0-TV](http://172.22.4.1:8090/0-TV-Series))
 * [[http://dn42.e-utp.net/]] - small dump site - 1Gbit
 * [[http://fido.pl.dn42.e-utp.net/share/]] - ~1TB, ~400kbit but up 24/7, allow from 172.22.0.0/15
 * [[http://172.22.8.202]] - small dump site
 * [[http://filer.nihilus.dn42/]] - [[http://172.22.92.2/]] - slow but mostly up
 * (new host!) [[http://turing.il.maxx.dn42/]], [[http://172.22.42.2/]] - WebDAV enabled, ~6.5TB, ~400kbit but up 24/7
 * [[https://dn42:Lee2phah@mirror.frubar.net/it.crowd/it.crowd.index.php]] - IT Crowd Mirror with all episodes, 100MBit
 * [[http://share.script.dn42/]] - IPv4 + IPv6, a couple of GiB music and movies, superslow but up 24/7
 * [[http://172.22.70.3]] - 1.7TB - movies/series/music - 400kBit - mostly up

### DC++ 

 * [[dchub://172.22.34.1:3325]] (not working?) - read about DC++ at [[http://en.wikipedia.org/wiki/Direct_Connect_%28file_sharing%29]]

### Streaming 

 * [[http://10.8.6.1:8000/]] - javaBeats.FM Internet Radio :: Streaming 24/7 Techno, Trance, Breakbeats, and More!
 * [[http://10.11.10.30:8000/]] - Freimusik

### Torrent 

 * [[http://chaosbay.hq.c3d2.de]]

## Web 
### Imageboard 

 * [[http://chan.dn42/]] dn42chan ([[http://chan.allo.dn42/]])

### One-Click Hosting 

 * [[http://img.allo.dn42/]] imagehosting (only jpg/gif/png up to 2 mb allowed.)

### ping check & traceroute from AS 64752 

  * [[http://allo.dn42/]]

### Pastebin 

  * Pastebox [[https://pastebox.martin89.dn42/]]

### Graph 

  * Tinc Graph [[http://graph.martin89.dn42/tinc.svg]]

## community aerial system 

  * [[http://172.22.159.2:3000/Extern/]] (currently down; Includes Freesat)

## Looking Glasses 

  * AS64692 [[http://zeus.nihilus.dn42/dn42/routes.cgi]]
  * AS64825 [[http://nagual.tim.dn42/]]
  * AS76102 [[http://www.mtak.dn42/lg/]] or [[http://172.22.139.137/lg/]]

## Misc 

  * [[http://nowhere.ws/dn42/]] - some random stuff concerning dn42, packages for debian, e.g. quagga
  * [[http://www.srn.dn42/crazydns/]] - description of the DNS system proposed by somerandomnick
  * 172.22.113.129 Tor Socks on Port 9050 
  * 172.22.113.129 DNS over Tor on Port 53 (No IPv6)
  * [[http://www.mtak.dn42/whois_5.0.10+dn42_i386.deb]] .deb whois with built-in dn42 hints (towards welterde's whois server)

## Services at Freifunk Augsburg 

We have a plugin that enables us to announce services in the mesh. So instead of listing them here again just have a look at [[http://10.11.0.8/cgi-bin-services.html]] to see what we have to offer. (Upload depends, but most probably dsl speed only)

# Ideas 

We could make the services zeroconf-browseable with the .dn42 TLD: [[http://wm161.net/2007/08/23/bind-and-zeroconf/]] Prerequisite: don't just delegate to one person.

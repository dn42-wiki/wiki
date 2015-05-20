# Looking Glasses

This is the list of **looking glasses** available for the dn42 network.  Some only display BGP information, while some others allow interactive queries (traceroute, details of a route, BGP-map visualisation, etc).

When a looking glass is described as `IPv4` or `IPv6`, it refers to the information displayed (or queried) by the looking glass, not to the reachability of the looking glass service itself.

## Reachable from the Internet & dn42

Please sort by AS number.

* AS 64701: http://dn42.netrik.de/de-home/ or http://172.22.232.5/de-home/ → interactive, bgpmap
* AS 64720: http://lg.prauscher.de or http://lg.prauscher.dn42
* AS 64737 + others: http://sour.is (IPv4 & IPv6) If you would like to submit your own site AS route information contact xuu@sour.is. 
* AS 64766: http://ix.ucis.dn42/routes.php or http://ix.ucis.nl/routes.php → interactive (traceroute)
* AS 76103: http://lg.nixnodes.dn42 or http://lg.nixnodes.net (IPv4) → interactive (traceroute)
  * http://map.nixnodes.net or http://map.nixnodes.dn42 →  interactive BGP-graph
* AS 4242420013: http://dn42.netrik.de/de-fra1/ or http://172.22.232.5/de-fra1/ → interactive, bgpmap
* AS 4242420101: http://edge1.core.chaos-darmstadt.de or http://lg.cda.dn42
* AS 4242420184: http://peerfinder.polynome.dn42 or http://peerfinder.polyno.me : it can be used as a distributed looking glass if you give it a dn42 address.
* AS 4242420300: http://lg-fr-rbx.wolke7.me or http://lg-fr-rbx.wolke7.dn42

**BROKEN**
* AS 64692: http://zeus.nihilus.dn42/dn42/routes.cgi or http://zeus.nowhere.ws/dn42/routes.cgi (IPv4 & IPv6) → non-interactive (simply displays all known routes) 

## Reachable only from within dn42

Please sort by AS number.

* AS 64835: http://lg.nordkapp-5.dn42 or http://172.22.235.4 (IPv4) → interactive
* AS 76142: http://lg.ffdn.dn42 (IPv4) → interactive (traceroute, BGP-map)
* AS 4242420022: http://mhm.dn42:5001
* AS 4242420123: http://lg.grmml.dn42/ (IPv4 & IPv6) → interactive (ping currently not working)
* AS 4242420184: http://dataviz.polynome.dn42/dn42/lastseen/ (IPv4) → non-interactive ("BGP last seen" service: keeps an history of previously announced BGP prefixes) **currently down**
* AS 4242420200: http://lg.punkt.dn42 (IPv4) → interactive (traceroute, BGP-map)
* AS 4242420321: http://lg.dn42 (IPv4 & IPv6) → interactive (traceroute, BGP-map)
* AS 4242422342: http://lg.gbe.dn42 → semi-interactive (no traceroute, no ping)
* AS 4242420820: http://node2-lg.servers.dn42 -> interactive (traceroute, BGP-map)

## Reachable only from the Internet

Please sort by AS number.

* AS 65529: http://bgp.freifunk-bielefeld.de/ulg/ulg.py → interactive (BGP-map)
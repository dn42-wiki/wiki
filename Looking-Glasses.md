# Looking Glasses

This is the list of **looking glasses** available for the dn42 network.  Some only display BGP information, while some others allow interactive queries (traceroute, details of a route, BGP-map visualisation, etc).

When a looking glass is described as `IPv4` or `IPv6`, it refers to the information displayed (or queried) by the looking glass, not to the reachability of the looking glass service itself.

## Reachable from the Internet & dn42

Please sort by AS number.

* AS 64692: http://zeus.nihilus.dn42/dn42/routes.cgi or http://zeus.nowhere.ws/dn42/routes.cgi (IPv4 & IPv6) → non-interactive (simply displays all known routes)
* AS 64766: http://ix.ucis.dn42/routes.php or http://ix.ucis.nl/routes.php → interactive (traceroute)
* AS 76103: http://lg.nixnodes.dn42 or http://lg.nixnodes.net (IPv4) → interactive (traceroute)
  * http://nixnodes.net/dn42/graph/ → interactive BGP-graph

## Reachable only from within dn42

Please sort by AS number.

* AS 64825: http://nagual.tim.dn42 or http://172.22.225.1 (IPv4) → interactive
* **[broken]** AS 76102: http://www.mtak.dn42/lg/ or http://172.22.139.137/lg/
* AS 76142: http://lg.ffdn.dn42 (IPv4) → interactive (traceroute, BGP-map)
* AS 76142: http://dataviz.polynome.dn42/dn42/lastseen/ (IPv4) → non-interactive ("BGP last seen" service: keeps an history of previously announced BGP prefixes)

## Reachable only from the Internet

Please sort by AS number.

* AS 65529: http://vpn1.freifunk-bielefeld.de/ulg/ulg.py (IPv4) → interactive (BGP-map)
* AS 65529: http://vpn1.freifunk-bielefeld.de/ulgv6/ulg.py (IPv6) → interactive (BGP-map)

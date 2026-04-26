# Application Programming Interfaces (APIs)
This page can be useful if you are trying to automate something or if you are trying to retrieve data programmatically.

## What is my IP
- whatsmyip:
  - ipv4+ipv6: [myip.dn42](http://myip.dn42/)
  - ipv4 only: [v4.myip.dn42](http://v4.myip.dn42/) or [172.20.0.81](http://172.20.0.81)
  - ipv6 only: [v6.myip.dn42](http://v6.myip.dn42/) or [fd42:d42:d42:81::1](http://[fd42:d42:d42:81::1]/)
  - Api endpoints:
    - /raw: return your IP address as plain text
    - /api: JSON with your IP plus the details of the server you reached (location, ASN, etc.)
  - with geolocation provided by [DN42-Geoip](https://github.com/Xe-iu/dn42-geoip) and autonomous system info provided by [DN42-GeoASN](https://github.com/rdp-studio/dn42-geoasn): [myip.launchpadx.dn42](https://myip.launchpadx.dn42)
    - Geolocation API
      - API endpoints:
        - IPv4: v4.myip.launchpadx.dn42
        - IPv6: v6.myip.launchpadx.dn42
        - Dual-stack: api.myip.launchpadx.dn42
      - [API documentation](https://github.com/Xe-iu/dn42-geoip/blob/main/api/README.md)
    - ASN API
      - API endpoints:
        - IPv4: asn4.myip.launchpadx.dn42
        - IPv6: asn6.myip.launchpadx.dn42
        - Dual-stack: asn.myip.launchpadx.dn42
  - [whatismyip.dn42](http://whatismyip.dn42)
    - API endpoint:
      - IPv4: [http://ipv4.whatismyip.dn42:8080/ipinfo](http://ipv4.whatismyip.dn42:8080/ipinfo)
      - IPv6: [http://ipv6.whatismyip.dn42:8080/ipinfo](http://ipv6.whatismyip.dn42:8080/ipinfo)


## Other network-related

- Cloudflare-like cdn-cgi/trace: [map.jerry.dn42/cdn-cgi/trace](https://map.jerry.dn42/cdn-cgi/trace)

### Map.dn42 API Services

Data refreshes whenever the collector provides a new MRT file, generally every 10-15 minutes.

| API                | URL | Description |
| ------------------ | --- | ----------- |
| `/map`             | [https://map.dn42/map](https://map.dn42/map) <br> [https://api.iedon.com/dn42/map](https://api.iedon.com/dn42/map)                                                       | Raw binary map data, [See binary format](https://iedon.net/post/4)   |
| `/map?type=json`   | [https://map.dn42/map?type=json](https://map.dn42/map?type=json) <br> [https://api.iedon.com/dn42/map?type=json](https://api.iedon.com/dn42/map?type=json)               | Raw map data, in JSON |
| `/asn/{asn}`       | [https://map.dn42/asn/4242422189](https://map.dn42/asn/4242422189) <br> [https://api.iedon.com/dn42/asn/4242422189](https://api.iedon.com/dn42/asn/4242422189)           | Real-time node information from the map, including WHOIS data, in JSON |
| `/ranking`         | [https://map.dn42/ranking](https://map.dn42/ranking) <br> [https://api.iedon.com/dn42/ranking](https://api.iedon.com/dn42/ranking)                                       | DN42 Global Ranking, based on the map.dn42 index |
| `/myip/[raw｜api]` | [https://map.dn42/myip/](https://map.dn42/myip/) <br> [https://map.dn42/myip/api](https://map.dn42/myip/api) <br> [https://map.dn42/myip/raw](https://map.dn42/myip/raw) | What's My IP instance by map.dn42, including `netname` and `country` information if available |


## ASN Authentication Solution
Authenticate your users by having them verify their ASN ownership with KIOUBIT-MNT using their registry-provided methods in an automated way. An example of this is the automatic peering system for the Kioubit Network.
[Documentation](https://dn42.g-load.eu/about/authentication-services/)

## Registry REST API

[dn42regsrv](https://git.burble.com/burble.dn42/dn42regsrv) is a REST API for the DN42 registry that provides a bridge between interactive applications and the registry.

As well as the main REST API to the DN42 registry, the server can also generate ROA tables and provides a small web application for exploring registry data.

A public instance of the API and associated explorer web app is available at the following URLs:

<https://explorer.burble.com/> (public internet link)  
<https://explorer.burble.dn42/> (DN42 link)

<https://explorer.dn42.pebkac.gr/> (public internet link)  
<https://explorer.pebkac.dn42/> (DN42 link)
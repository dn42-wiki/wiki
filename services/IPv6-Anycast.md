We provide some anycast services over IPv6.

## Anycast address space

**fd42:d42:d42::/48** is reserved for anycast services.

Each anycast service runs on a dedicated /64 in this range.  This way, nobody needs to update filters.

Remember, if you announce an anycast /64, then you need to provide **all** services within this /64. It's probably simpler to only provide one service for each /64.

## Anycast services

| **Name**               | **Service address**       | **Protocol/port** | **Comment**                   | 
| ---------------------- | ------------------------- | ----------------- | ----------------------------- |
| Recursive DNS resolver | `fd42:d42:d42:53::1/64`   | UDP/53            | `.` and `dn42.` [Providers][] |
| Whois Database         | `fd42:d42:d42:43::1/64`   | TCP/43            |                               |
| TOR SOCKS5 Proxy       | `fd42:d42:d42:9050::1/64` | TCP/9050          |                               |        
| internal Wiki          | `fd42:d42:d42:80::1/64`   | TCP/80, TCP/443   |                               |


[Providers]: Providing-Anycast-DNS#Persons-providing-anycast-DNS-for-IPv6

### Future services

- streaming
- other kind of DNS (authoritative-only, recursive for `dn42` only)
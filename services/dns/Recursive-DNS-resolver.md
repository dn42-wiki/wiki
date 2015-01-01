If you want to run your own recursive DNS server, you must find upstream servers that are authoritative for the dn42 zones.

You may use some servers listed in the [[table of anycast servers|Providing-Anycast-DNS#Persons-providing-anycast-DNS]], or just use `172.22.119.160` and `172.22.119.163` (ns{1,2}.fritz.dn42).

## Configuration

### Unbound

Configuration for `unbound.conf`

```
server:
        local-zone: "22.172.in-addr.arpa." nodefault
        local-zone: "23.172.in-addr.arpa." nodefault

stub-zone:
        name: "dn42"
        stub-prime: yes
        stub-addr: 172.22.119.160
        stub-addr: 172.22.119.163

stub-zone:
        name: "22.172.in-addr.arpa"
        stub-prime: yes
        stub-addr: 172.22.119.160
        stub-addr: 172.22.119.163

stub-zone:
        name: "23.172.in-addr.arpa"
        stub-prime: yes
        stub-addr: 172.22.119.160
        stub-addr: 172.22.119.163
```
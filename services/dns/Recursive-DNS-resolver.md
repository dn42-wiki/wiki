If you want to run your own recursive DNS server, you must find upstream servers that are authoritative for the dn42 zones.

You may use some servers listed in the [table of anycast servers](/services/dns/Providing-Anycast-DNS#Persons-providing-anycast-DNS), or just use `172.22.119.160` and `172.22.119.163` (ns{1,2}.fritz.dn42).

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

### Unbound with root-hints
Alternatively you can put dn42 root servers in the root-hints file for recursive resolving.

```
# /etc/unbound/unbound.conf.d/dn42.conf 
server:
        # DNSSEC validation will fail
        val-permissive-mode: yes
        # recursive queries for everyone
        access-control: 0.0.0.0/0 allow
        # dn42 root servers
        root-hints: /etc/unbound/dn42.hints
	# enable IPv6
	do-ip6: yes
	# allow reverse lookup of rfc1918 space, which includes the DN42 address space
	unblock-lan-zones: yes
	insecure-lan-zones: yes

remote-control:
        control-enable: no
```

The `/etc/unbound/dn42.hints` file:
```
.                                         NS    a.root-servers.dn42.
a.root-servers.dn42.           3600000    A     172.22.177.6
.                                         NS    m.root-servers.dn42.
m.root-servers.dn42.           3600000    A     172.23.67.67
.                                         NS    t.root-servers.dn42.
t.root-servers.dn42.           3600000    A     172.22.102.141
.                                         NS    x.root-servers.dn42.
x.root-servers.dn42.           3600000    A     172.22.141.1
```

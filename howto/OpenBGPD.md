This guide describes a simple configuration for [OpenBGPD](https://openbgpd.org) running on [OpenBSD](https://openbsd.org).
The [portable version](https://openbgpd.org/ftp.html) should run with little to no configuration changes on other operating systems as well.

Other than the
[`bgpd.conf(5)`](https://man.openbsd.org/bgpd.conf.5) and
[`bgpd(8)`](http://man.openbsd.org/bgpd.8) man pages and the
OpenBSD
[`/etc/examples/bgpd.conf`](http://cvsweb.openbsd.org/cgi-bin/cvsweb/~checkout~/src/etc/examples/bgpd.conf?rev=HEAD&content-type=text/plain&only_with_tag=MAIN),
you might also find useful reference or ideas in the
[Bird2](/howto/Bird2) page (even if you don't use Bird), as
it likely presents the most widespread dn42 router setup.

# Example configuration
When copying from the below configuration, be sure to at
least replace the various `<PLACEHOLDER>`s with your own
numbers.

Concrete configuration examples can also be found elsewhere,
e.g.:

[https://smrk.net/text/openbsd-dn42-setup.txt](https://smrk.net/text/openbsd-dn42-setup.txt)

[https://kaizo.org/2024/01/03/openbsd-bgpd/](https://kaizo.org/2024/01/03/openbsd-bgpd/)

Given OpenBGPD's limited support for multiprotocol sessions
(no extended next hop (RFC8950)) and
[some](https://marc.info/?l=openbgpd-users&m=159983144408845&w=2)
[issues](https://marc.info/?l=openbgpd-users&m=165605427017298&w=2)
with IPv6 link-local nexthops, we configure separate IPv4
and IPv6 sessions for each peer, and for IPv6 we adjust
nexthop to a "global" address (i.e., one from our dn42 IPv6
allocation) assigned to each peering (Wireguard) interface
(each interface gets its own).

To avoid burning a dn42 IPv4 address for each peering, we
put the router's dn42 IPv4 address on a loopback interface
and have `bgpd` bind to that address (`local-address` in
`bgpd.conf`) when opening IPv4 BGP sessions; each peering
interface gets an IPv4 address from an RFC1918 subnet
(192.168.42/24), and a static route to the corresponding
peer via that address.

## `/etc/hostname.lo42`

```conf
inet <YOUR-ROUTER-DN42-IPv4>
inet6 <YOUR-ROUTER-DN42-IPv6>
# add a fallback route for our prefixes to discard traffic to
# targets without a more specific route
!route -qn add -blackhole <YOUR-DN42-IPv4-PREFIX> 127.0.0.1
!route -qn add -blackhole <YOUR-DN42-IPv6-PREFIX> ::1
```

## `/etc/hostname.wg1234`
(one example; similar for each peer)

```conf
inet 192.168.42.1/32
inet6 fe80::1 64 # this is the address your peer will want to know
# (and connect to); the following address is only really needed
# to provide a non-link-local IPv6 address for the nexthop setting;
# you can pick it "arbitrarily" from your dn42 IPv6 allocation
inet6 <YOUR-DN42-IPv6-OF-PEER1-INTERFACE> 64
group my_dn
wgport 21234 wgkey <PRIVKEY-BASE64>
wgpeer <PEER1-PUBKEY-BASE64> \
  wgdescr "dn42 peer1" \
  wgaip fe80::/64 wgaip fd00::/8 wgaip 10.0.0.0/8 wgaip 172.20.0.0/14 \
  wgendpoint <PEER1-HOSTNAME-OR-IP> 24321
up
# add a static IPv4 route to the peer
!route -nq add <PEER1-IPv4> 192.168.42.1
```

## `/etc/pf.conf`
(only the dn42-related snippet)

```conf
pass in quick proto {icmp icmp6} max-pkt-rate 30/3
dn42_self = <YOUR-ROUTER-DN42-IPv4>
table <dn42etc> const {172.20/14 172.31/16 10/8 fd00::/8}
table <dn42peers> const {<PEER1-IPv4> fe80::/64}
pass in quick on egress proto udp to port 21234
pass out quick on my_dn proto tcp to <dn42peers> port bgp !received-on any
pass in quick on my_dn proto tcp from <dn42peers> \
  to {$dn42_self (my_dn)} port bgp
# block everything (except for ICMP above) destined to the
# router itself; only dn42 transit and BGP sessions are allowed
block in log quick on my_dn to {$dn42_self (my_dn)}
# 'no state' as we might not see both directions of transit traffic
pass on my_dn from <dn42etc> to <dn42etc> no state
```

## `/etc/bgpd.conf`
```conf
ASN = "<YOUR-AS-NUMBER>"
ID = "<YOUR-ROUTER-DN42-IPv4>"

AS $ASN
router-id $ID

# list of networks that may be originated by our ASN
prefix-set mydn42 {
	<YOUR-DN42-IPv4-PREFIX>
	<YOUR-DN42-IPv6-PREFIX>
}

# https://dn42.eu/howto/Bird2#example-configuration
prefix-set dn42etc {
	172.20.0.0/14 prefixlen 21 - 29	# dn42
	172.20.0.0/24 prefixlen >= 28	# dn42 Anycast
	172.21.0.0/24 prefixlen >= 28	# dn42 Anycast
	172.22.0.0/24 prefixlen >= 28	# dn42 Anycast
	172.23.0.0/24 prefixlen >= 28	# dn42 Anycast
	172.31.0.0/16 or-longer		# ChaosVPN
	10.100.0.0/14 or-longer		# ChaosVPN
	10.127.0.0/16 or-longer		# neonetwork
	10.0.0.0/8    prefixlen 15 - 24	# Freifunk.net
	fd00::/8      prefixlen 44 - 64	# dn42
}

# https://dn42.burble.com/services/public/#roa-data
# https://dn42.burble.com/roa/dn42_roa_obgpd_46.conf
# see the crontab snippet and an update script further below
include "/var/db/openbgpd/dn42_roa_obgpd_46.conf"

network prefix-set mydn42 set {
	# https://dn42.dev/howto/BGP-communities
	# e.g., for Germany this could read
	# community 64511:41
	# community 64511:1276
	community 64511:<READ-THE-LINK-ABOVE>
	community 64511:<READ-THE-LINK-ABOVE>
	large-community $ASN:1:1
}

listen on $ID
listen on <PEER1-IPv6-LOCAL> # e.g. fe80::1%wg1234

group dn42peers {
	# RFC7454 sec. 8
	# (currently no peer sends more than 800 prefixes for
	# a single address family; increase this if using
	# multi-protocol BGP (or when the network grows)!)
	max-prefix 1000 restart 60
	neighbor <PEER1-IPv4> {
		descr peer1_4
		remote-as <PEER1-ASN>
		local-address $ID
	}
	neighbor <PEER1-IPv6-REMOTE> { # e.g. fe80::2%wg1234
		descr peer1_6
		remote-as <PEER1-ASN>
		set nexthop <YOUR-DN42-IPv6-OF-PEER1-INTERFACE>
	}
}

# deny EBGP UPDATEs to our own originated prefixes
deny quick from ebgp prefix-set mydn42 or-longer
# filter out overlong paths
deny quick from any max-as-len 10

allow from group dn42peers prefix-set dn42etc ovs valid
allow to group dn42peers prefix-set dn42etc

# scrub communities relevant to our ASN from EBGP neighbors
# (RFC7454 sec. 11)
# match from ebgp set { community delete $ASN:* }
match from ebgp set { large-community delete $ASN:*:* }

# honor requests to gracefully shutdown BGP sessions (RFC8326)
match from any community GRACEFUL_SHUTDOWN set { localpref 0 }
```

## ROA

The `roa-set` for route origin validation (`ovs valid` in
the config above) can be generated from the dn42 registry;
here we use
[data](https://dn42.burble.com/services/public/#roa-data)
conveniently provided by BURBLE-MNT.

If using the update script below, don't forget to create the
`/var/db/openbgpd/` directory first.

### `/root/openbgpd-roa-update.sh`
```sh
#!/bin/sh

die() {
  >&2 printf '%s: %s\n' "${0##*/}" "$*"
  exit 1
}

# Unfortunately, burble regenerates the ROA files (hourly?)
# even when nothing changed, so If-Modified-Since doesn't
# help (similar story for .meta).

metafile=/var/db/openbgpd/registry.meta
err=$(ftp -o "$metafile" \
  https://explorer.burble.com/api/registry/.meta 2>&1 >/dev/null) ||
  die "/api/registry/.meta download failed: $err"

if ! cmp -s "$metafile" "$metafile".old >/dev/null 2>&1; then
  mv "$metafile" "$metafile".old
  roafile=/var/db/openbgpd/dn42_roa_obgpd_46.conf
  if err=$(ftp -To "$roafile".new \
    https://dn42.burble.com/roa/dn42_roa_obgpd_46.conf \
    2>&1 >/dev/null); then
    mv "$roafile".new "$roafile"
    bgpctl reload
  else
    die "ROA download failed: $err"
  fi
else
  logger -cisp user.info "${0##*/}: registry unchanged, not reloading"
fi
```

### `/var/cron/tabs/root`
```conf
~       *       *       *       *       -ns /root/openbgpd-roa-update.sh
```

# Looking glass
This is mostly OpenBSD specific since [bgplg(8)](http://man.openbsd.org/bgplg.8) and [httpd(8)](http://man.openbsd.org/httpd.8) ship as part of the operating system.
The **bgplg** manual contains the steps and example [httpd.conf(5)](http://man.openbsd.org/httpd.conf.5) required to enable the looking glass.

See <https://lg-dn42.vinishor.xyz/bgplg> (IPv6) for a
running instance operating within DN42.

# VyOS 1.5.x circinus

VyOS is an open source software router. It is feature rich and supports
multiple deployment options such as physical hardware (old PCs) or a VPC/VM.

VyOS offers three release channels:

- **Rolling release** – bleeding-edge nightly builds with all the latest
  features (including WireGuard), but no stability guarantees: anything may
  change and experimental features can be added or removed at any time. Best
  for development and testing only.
- **VyOS Stream** – a quarterly technology-preview / quality-gate on the way
  to the 1.5 (circinus) LTS. It carries most of the upcoming LTS features but
  is considerably more stable than the rolling release, since features are
  only backported once their design is settled. Stream images are named after
  their release year and month, e.g. `vyos-2026.03-generic-amd64.iso`.
- **LTS** – the stable long-term-support release, available to subscribers
  (or via the no-cost subscription for those who qualify, or by building from
  source).

This guide was written and tested on **VyOS Stream (circinus), version
2026.03**.

Rolling images: <https://vyos.net/get/nightly-builds/><br>
VyOS Stream images: <https://vyos.net/get/stream/>

This guide uses the following example values throughout. Replace the
**operator-assigned** ones with your own DN42 registry allocation; the
**DN42-wide** ranges are fixed and must be left as-is.

| Example value            | Meaning                                  | Change it?                 |
| ------------------------ | ---------------------------------------- | -------------------------- |
| `4242421234`             | **Your** ASN                             | Yes                        |
| `4242424242`             | **Your peer's** ASN                      | Yes                        |
| `172.20.20.0/27`         | **Your assigned IPv4** (`inetnum`)       | Yes                        |
| `fd88:9deb:a69e::/48`    | **Your assigned IPv6** (`inet6num`)      | Yes                        |
| `10.0.0.0/8`, `172.20.0.0/14`, `172.31.0.0/16` | Whole DN42 IPv4 space  | No, leave as-is            |
| `fd00::/8`               | Whole DN42 IPv6 space                    | No, leave as-is            |
| `172.20.0.53`, `172.23.0.53` | DN42 recursive resolvers             | No, leave as-is            |

## Firewall

### Firewall Baseline (stateful global)

By default, VyOS performs **stateless** filtering: with no ruleset nothing is filtered, and once a drop-by-default ruleset is applied, return traffic of established sessions is **not** allowed back automatically. We enable stateful inspection globally so we don't have to repeat established/related rules in every ruleset.

```sh
set firewall global-options state-policy established action 'accept'
set firewall global-options state-policy related action 'accept'
```

### Edge: accept invalid

We also accept `invalid` packets on our network's edge. This should **not** become common practice elsewhere. It is specific to a DN42 BGP edge, where conntrack-untracked traffic (including BGP) can otherwise be wrongly classified as `invalid` and dropped.

```sh
set firewall global-options state-policy invalid action 'accept'
```

### Per-WireGuard-interface rulesets

These create **in** (transit) and **local** (to-the-router) baseline templates to be applied to every WireGuard interface facing a peer.

Only the two `My-Assigned-Space-*` lines need editing to match your own registry prefixes; the `Allowed-Transit-*` ranges are the whole of DN42 and stay as-is.

```sh
# --- Groups v4 ---
set firewall group network-group Allowed-Transit-v4 network '10.0.0.0/8'
set firewall group network-group Allowed-Transit-v4 network '172.20.0.0/14'
set firewall group network-group Allowed-Transit-v4 network '172.31.0.0/16'
set firewall group network-group My-Assigned-Space-v4 network '172.20.20.0/27'

# --- Groups v6 ---
set firewall group ipv6-network-group Allowed-Transit-v6 network 'fd00::/8'
set firewall group ipv6-network-group My-Assigned-Space-v6 network 'fd88:9deb:a69e::/48'

# --- Inbound / transit v4 ---
set firewall ipv4 name Tunnels_In_v4 default-action 'drop'
set firewall ipv4 name Tunnels_In_v4 default-log
set firewall ipv4 name Tunnels_In_v4 rule 68 action 'drop'
set firewall ipv4 name Tunnels_In_v4 rule 68 description 'Block Traffic to Operator Assigned IP Space'
set firewall ipv4 name Tunnels_In_v4 rule 68 destination group network-group 'My-Assigned-Space-v4'
set firewall ipv4 name Tunnels_In_v4 rule 68 log
set firewall ipv4 name Tunnels_In_v4 rule 70 action 'accept'
set firewall ipv4 name Tunnels_In_v4 rule 70 description 'Allow Peer Transit'
set firewall ipv4 name Tunnels_In_v4 rule 70 destination group network-group 'Allowed-Transit-v4'
set firewall ipv4 name Tunnels_In_v4 rule 70 source group network-group 'Allowed-Transit-v4'
set firewall ipv4 name Tunnels_In_v4 rule 99 action 'drop'
set firewall ipv4 name Tunnels_In_v4 rule 99 description 'Black Hole'
set firewall ipv4 name Tunnels_In_v4 rule 99 log

# --- Inbound / transit v6 ---
set firewall ipv6 name Tunnels_In_v6 default-action 'drop'
set firewall ipv6 name Tunnels_In_v6 default-log
set firewall ipv6 name Tunnels_In_v6 rule 68 action 'drop'
set firewall ipv6 name Tunnels_In_v6 rule 68 description 'Block Traffic to Operator Assigned IP Space'
set firewall ipv6 name Tunnels_In_v6 rule 68 destination group network-group 'My-Assigned-Space-v6'
set firewall ipv6 name Tunnels_In_v6 rule 68 log
set firewall ipv6 name Tunnels_In_v6 rule 70 action 'accept'
set firewall ipv6 name Tunnels_In_v6 rule 70 description 'Allow Peer Transit'
set firewall ipv6 name Tunnels_In_v6 rule 70 destination group network-group 'Allowed-Transit-v6'
set firewall ipv6 name Tunnels_In_v6 rule 70 source group network-group 'Allowed-Transit-v6'
set firewall ipv6 name Tunnels_In_v6 rule 99 action 'drop'
set firewall ipv6 name Tunnels_In_v6 rule 99 description 'Black Hole'
set firewall ipv6 name Tunnels_In_v6 rule 99 log

# --- Local / to the router v4 ---
set firewall ipv4 name Tunnels_Local_v4 default-action 'drop'
set firewall ipv4 name Tunnels_Local_v4 rule 50 action 'accept'
set firewall ipv4 name Tunnels_Local_v4 rule 50 protocol 'icmp'
set firewall ipv4 name Tunnels_Local_v4 rule 61 action 'accept'
set firewall ipv4 name Tunnels_Local_v4 rule 61 description 'Allow BGP'
set firewall ipv4 name Tunnels_Local_v4 rule 61 destination port '179'
set firewall ipv4 name Tunnels_Local_v4 rule 61 protocol 'tcp'
set firewall ipv4 name Tunnels_Local_v4 rule 99 action 'drop'
set firewall ipv4 name Tunnels_Local_v4 rule 99 description 'Black Hole'
set firewall ipv4 name Tunnels_Local_v4 rule 99 log

# --- Local / to the router v6 ---
set firewall ipv6 name Tunnels_Local_v6 default-action 'drop'
set firewall ipv6 name Tunnels_Local_v6 rule 50 action 'accept'
set firewall ipv6 name Tunnels_Local_v6 rule 50 protocol 'ipv6-icmp'
set firewall ipv6 name Tunnels_Local_v6 rule 61 action 'accept'
set firewall ipv6 name Tunnels_Local_v6 rule 61 description 'Allow BGP'
set firewall ipv6 name Tunnels_Local_v6 rule 61 destination port '179'
set firewall ipv6 name Tunnels_Local_v6 rule 61 protocol 'tcp'
set firewall ipv6 name Tunnels_Local_v6 rule 99 action 'drop'
set firewall ipv6 name Tunnels_Local_v6 rule 99 description 'Black Hole'
set firewall ipv6 name Tunnels_Local_v6 rule 99 log
```

## WireGuard

### Setup Keys

You can generate one keypair and reuse it for every WireGuard peering, or generate a different one per peering.

```sh
generate pki wireguard key-pair

# Output give public keys like: UcqcZsJvq1MlYgo3gObjaJ8FH+N7wkfV+EH3YDAMyRE=
```

To generate **and install** a unique keypair on an interface in one shot (from `configure` mode, top level):

A WireGuard interface name on VyOS must be `wg` followed by digits only (letters are rejected, so `wgpeer` won't work). The name has no protocol meaning, but since interface names must be unique, the DN42 convention is to name each tunnel after the peer's ASN (for example `wg4242424242`, or a short form). That way the name tells you who's on the other end. Don't name a tunnel after your own ASN: it would collide the moment you add a second peer and wouldn't identify anything.

```sh
run generate pki wireguard key-pair install interface wg4242
# 1 value(s) installed. Use "compare" to see pending changes, and "commit" to apply.
# Corresponding public-key to use on peer system is: 'UcqcZsJvq1MlYgo3gObjaJ8FH+N7wkfV+EH3YDAMyRE='
```

To retrieve the public key later (op-mode):

```sh
run show interfaces wireguard wg4242 public-key

# Output example:
# UcqcZsJvq1MlYgo3gObjaJ8FH+N7wkfV+EH3YDAMyRE=
```

### Configure the peer's tunnel

This example assumes your ASN is `4242421234` and your peer's ASN is `4242424242`.
Interface naming follows the DN42 convention `wg<peer-ASN-last-4-digits>` -> `wg4242`.

```sh
set interfaces wireguard wg4242 description 'AS4242424242 - My Peer'

# DN42 port convention:
#   - YOUR listen port      = 2 + last 4 digits of your PEER's ASN  -> 2 + 4242 = 24242
#   - the port you point to = 2 + last 4 digits of YOUR  ASN        -> 2 + 1234 = 21234
set interfaces wireguard wg4242 port '24242'

# NOTE: the private key is already installed by
#   'run generate pki wireguard key-pair install interface wg4242'
# Do NOT re-set it here, or you will overwrite your real key.

# Your link-local IPv6 (the one you give to the auto-peering portal).
# Arbitrary, but must be inside fe80::/64, e.g. derived from your ASN.
set interfaces wireguard wg4242 address 'fe80::1234/64'

# (No IPv4 on the tunnel: with extended-next-hop enabled, your DN42 IPv4
#  lives on the LAN interface instead, anchored by a blackhole route,
#  see the BGP prerequisites section below.)

# Your peer's clearnet endpoint, must be an IP literal, not a DNS name.
# Resolve the peer's FQDN first and use the resulting IP.
set interfaces wireguard wg4242 peer mypeer address '<peer endpoint IP>'
set interfaces wireguard wg4242 peer mypeer port '21234'

# Allow everything and rely on the firewall + BGP for control
set interfaces wireguard wg4242 peer mypeer allowed-ips '0.0.0.0/0'
set interfaces wireguard wg4242 peer mypeer allowed-ips '::/0'

# Your peer's WireGuard public key (from the auto-peering result)
set interfaces wireguard wg4242 peer mypeer public-key '<peer wireguard public key>'

# Helps the BGP session come up reliably
set interfaces wireguard wg4242 peer mypeer persistent-keepalive '60'

# Remove the auto-generated link-local so the BGP session sources from YOUR
# chosen link-local only. With two link-locals on the interface, BGP may source
# from the wrong one and the peer won't recognize the session.
set interfaces wireguard wg4242 ipv6 address no-default-link-local
```

**Placeholders to replace with your own values:**

| Placeholder                   | Meaning                                               |
| ----------------------------- | ----------------------------------------------------- |
| `wg4242`                      | interface name = `wg` + your peer's last 4 ASN digits |
| `24242`                       | your listen port = `2` + peer's last 4 ASN digits     |
| `fe80::1234/64`               | your link-local (the one you gave the portal)         |
| `<peer endpoint IP>`          | peer's clearnet endpoint, as an IP literal            |
| `21234`                       | peer's port = `2` + your last 4 ASN digits            |
| `<peer wireguard public key>` | from the auto-peering result                          |

### Apply the firewall rulesets to the tunnel

```sh
# Group every DN42 peer tunnel together (add future wgXXXX interfaces here)
set firewall group interface-group DN42-Peers interface 'wg4242'

# Transit traffic (forward hook) -> Tunnels_In_*
set firewall ipv4 forward filter rule 10 description 'DN42 peers transit'
set firewall ipv4 forward filter rule 10 action 'jump'
set firewall ipv4 forward filter rule 10 inbound-interface group 'DN42-Peers'
set firewall ipv4 forward filter rule 10 jump-target 'Tunnels_In_v4'
set firewall ipv6 forward filter rule 10 description 'DN42 peers transit'
set firewall ipv6 forward filter rule 10 action 'jump'
set firewall ipv6 forward filter rule 10 inbound-interface group 'DN42-Peers'
set firewall ipv6 forward filter rule 10 jump-target 'Tunnels_In_v6'

# Traffic to the router itself, e.g. BGP (input hook) -> Tunnels_Local_*
set firewall ipv4 input filter rule 10 description 'DN42 peers to router'
set firewall ipv4 input filter rule 10 action 'jump'
set firewall ipv4 input filter rule 10 inbound-interface group 'DN42-Peers'
set firewall ipv4 input filter rule 10 jump-target 'Tunnels_Local_v4'
set firewall ipv6 input filter rule 10 description 'DN42 peers to router'
set firewall ipv6 input filter rule 10 action 'jump'
set firewall ipv6 input filter rule 10 inbound-interface group 'DN42-Peers'
set firewall ipv6 input filter rule 10 jump-target 'Tunnels_Local_v6'
```

You will also want a plain accept for your own LAN, both for traffic transiting the router and for traffic to the router itself:

```sh
set firewall ipv4 forward filter rule 20 description 'LAN forward'
set firewall ipv4 forward filter rule 20 action 'accept'
set firewall ipv4 forward filter rule 20 inbound-interface name 'eth2'
set firewall ipv4 input filter rule 20 description 'LAN to router'
set firewall ipv4 input filter rule 20 action 'accept'
set firewall ipv4 input filter rule 20 inbound-interface name 'eth2'
set firewall ipv6 forward filter rule 20 description 'LAN forward v6'
set firewall ipv6 forward filter rule 20 action 'accept'
set firewall ipv6 forward filter rule 20 inbound-interface name 'eth2'
set firewall ipv6 input filter rule 20 description 'LAN to router v6'
set firewall ipv6 input filter rule 20 action 'accept'
set firewall ipv6 input filter rule 20 inbound-interface name 'eth2'
```

## BGP

### Prerequisites: anchor your prefixes (blackhole + LAN address)

BGP only advertises a prefix from a `network` statement if a route of the **exact
same length** already exists in the routing table. Instead of a dummy interface,
we anchor each registry prefix with a static **blackhole** route. A blackhole
route is always present regardless of interface or tunnel state, so BGP keeps
advertising your aggregate even when a link flaps.

Your actual DN42 addresses live on your **LAN-facing interface**, where your
clients and services sit. Traffic to real hosts follows the connected route;
traffic to unused addresses in the range falls through to the blackhole and is
dropped, so you never leak or loop packets for hosts that don't exist.

```sh
# DN42 addresses on the LAN interface (adjust eth2 to your LAN NIC).
# The /27 gives you the IPv4 LAN; the /64 is one segment carved from your /48.
set interfaces ethernet eth2 description 'LAN services DN42'
set interfaces ethernet eth2 address '172.20.20.1/27'
set interfaces ethernet eth2 address 'fd88:9deb:a69e:1::1/64'

# Always-up aggregate routes so BGP can advertise the exact registry prefixes,
# independent of any interface or tunnel state.
set protocols static route 172.20.20.0/27 blackhole
set protocols static route6 fd88:9deb:a69e::/48 blackhole
```

Here the IPv4 LAN is a `/27` matching your `inetnum`, and the IPv6 `/48`
blackhole matches your `inet6num` exactly, which is what the BGP `network`
statements need. The `/64` on the LAN is just one usable segment of that `/48`
for client autoconfiguration.

Verify:

```sh
run show interfaces ethernet eth2
run show ip route 172.20.20.0/27
run show ipv6 route fd88:9deb:a69e::/48
```

### Initial Router Setup

```sh
# Your ASN
set protocols bgp system-as '4242421234'

# Advertise your EXACT registry prefixes (a more-specific subnet may be filtered by peers)
set protocols bgp address-family ipv4-unicast network '172.20.20.0/27'
set protocols bgp address-family ipv6-unicast network 'fd88:9deb:a69e::/48'

# Router-id = your lowest DN42 IPv4
set protocols bgp parameters router-id '172.20.20.1'
```

### Neighbor: MP-BGP over IPv6 link-local + extended next-hop

One peer-group carries the address families, the extended-next-hop capability, and `remote-as external` (every DN42 peer is a different external AS).

```sh
# Shared peer-group for all DN42 link-local MP-BGP peers
set protocols bgp peer-group dn42 remote-as 'external'
set protocols bgp peer-group dn42 address-family ipv4-unicast
set protocols bgp peer-group dn42 address-family ipv6-unicast
set protocols bgp peer-group dn42 capability extended-nexthop

# The peer, addressed on the peer's link-local, scoped to the tunnel interface.
# Replace fe80::5678 with the link-local your peer's auto-peering portal gave you
# (this is the PEER's link-local, not yours).
set protocols bgp neighbor fe80::5678 interface source-interface 'wg4242'
set protocols bgp neighbor fe80::5678 peer-group 'dn42'
```

Address the peer's link-local **explicitly** as shown above. Avoid unnumbered
peering (`neighbor wg4242 interface v6only`): unnumbered relies on the peer
sending IPv6 Router Advertisements to auto-discover its link-local, and some
peers (for example those running BIRD) do not send them, so the session stays
`Idle` and never sends a single packet.

Also, do not add `remote-as` on the neighbor when the peer-group already sets
`remote-as external`; VyOS rejects it with `cannot override remote-as of
peer-group`.

### Verify

```sh
run show bgp summary          # both address families
run show bgp ipv4 summary
run show bgp ipv6 summary
```

A healthy session shows a number (received prefixes) under `State/PfxRcd`, not `Connect`/`Active`:

```txt
Neighbor        V         AS   MsgRcvd   MsgSent   TblVer  InQ OutQ  Up/Down State/PfxRcd   PfxSnt Desc
fe80::5678      4 4242424242      1921      1850     1103    0    0 00:00:08         1097     1098 N/A
```

Once the session is `Established`, confirm end-to-end routing to DN42. Because the
router has no DN42 IPv4 on the tunnel, source pings from your DN42 LAN address:

```sh
run ping 172.20.0.53 source-address 172.20.20.1 count 3
run ping fd42:d42:d42:53::1 source-address fd88:9deb:a69e:1::1 count 3
```

A reply (with `ttl` < 64, showing it crossed DN42 hops) confirms the extended-next-hop
is working, IPv4 destinations resolved via an IPv6 next-hop over the tunnel.

## RPKI / ROA Filtering

On DN42, RPKI lets your router reject route announcements whose origin AS doesn't match the registry (likely hijacks). You run a small **RTR cache** that downloads the DN42 ROA table, and point VyOS at it.

Cloudflare's GoRTR is archived; DN42 now uses **StayRTR** (`rpki/stayrtr`), a
drop-in fork with the same flags, and the ROA file is still published by Burble.
StayRTR has no `-verify` flag (GoRTR had one): a stray `-verify=false` makes the
container exit immediately (`Exited (2)`) and just print its help. Valid StayRTR
args: `-cache <url> -checktime=false [-bind :8082]`.

### Option A: Run StayRTR as a container on VyOS

```sh
# op-mode: pull the image
run add container image rpki/stayrtr
```

```sh
# config-mode: an internal-only network for the cache
set container network rpki prefix '172.16.2.0/24'

# the StayRTR container, note: flags go in `arguments`, not `command`
set container name stayrtr image 'rpki/stayrtr'
set container name stayrtr arguments '-cache https://dn42.burble.com/roa/dn42_roa_46.json -checktime=false -bind :8082'
set container name stayrtr network rpki address '172.16.2.10'
set container name stayrtr restart 'on-failure'
```

### Option B: Run StayRTR on a separate host (Docker)

```sh
docker run -d --name dn42-rpki --restart=always -p 8082:8082 \
  rpki/stayrtr -cache https://dn42.burble.com/roa/dn42_roa_46.json \
  -checktime=false -bind :8082
```

### Point VyOS at the cache

```sh
set protocols rpki cache 172.16.2.10 port '8082'
set protocols rpki cache 172.16.2.10 preference '1'
```

*(Use the container IP `172.16.2.10` for Option A, or the separate host's IP for Option B.)*

Verify the cache is connected and the table is populated:

```sh
run show rpki cache-connection
run show rpki prefix-table
```

After pointing VyOS at the cache, you may need to force FRR to (re)connect. Otherwise it may stay
on "No connection" because it gave up while the container was starting:

```sh
run reset rpki
```

Then re-verify:

```sh
run show rpki cache-connection      # expect: (connected)
run show rpki prefix-table          # expect: thousands of entries
```

### Filtering route-map (RPKI + DN42 range sanity, combined)

A single route-map does both jobs: drop RPKI-`invalid` prefixes **and** only accept prefixes that fall inside DN42's address space (a bogon guard RPKI alone doesn't give you).

The `le '32'` on IPv4 is deliberate: DN42's recursive resolvers are announced as
host routes (`172.20.0.53/32`, `172.23.0.53/32`). A stricter bound would silently
drop every `/32`, so those resolvers never enter your table and DNS later fails
for no obvious reason. Keep `le '32'` (and `le '64'` on IPv6) so host routes are
accepted.

```sh
# --- Prefix-lists: DN42's allocated ranges ---
set policy prefix-list DN42-v4 rule 10 action 'permit'
set policy prefix-list DN42-v4 rule 10 prefix '172.20.0.0/14'
set policy prefix-list DN42-v4 rule 10 le '32'
set policy prefix-list DN42-v4 rule 20 action 'permit'
set policy prefix-list DN42-v4 rule 20 prefix '10.0.0.0/8'
set policy prefix-list DN42-v4 rule 20 le '32'

set policy prefix-list6 DN42-v6 rule 10 action 'permit'
set policy prefix-list6 DN42-v6 rule 10 prefix 'fd00::/8'
set policy prefix-list6 DN42-v6 rule 10 le '64'

# --- One combined filter, used both import and export ---
set policy route-map DN42-PEERING rule 5 action 'deny'
set policy route-map DN42-PEERING rule 5 description 'Reject RPKI invalid (possible hijack)'
set policy route-map DN42-PEERING rule 5 match rpki 'invalid'
set policy route-map DN42-PEERING rule 20 action 'permit'
set policy route-map DN42-PEERING rule 20 description 'Accept DN42 IPv4 ranges'
set policy route-map DN42-PEERING rule 20 match ip address prefix-list 'DN42-v4'
set policy route-map DN42-PEERING rule 21 action 'permit'
set policy route-map DN42-PEERING rule 21 description 'Accept DN42 IPv6 ranges'
set policy route-map DN42-PEERING rule 21 match ipv6 address prefix-list 'DN42-v6'
set policy route-map DN42-PEERING rule 99 action 'deny'
set policy route-map DN42-PEERING rule 99 description 'Drop everything else'
```

The same map works for both address families: a `match ip address` rule simply
doesn't match IPv6 routes (and vice-versa), so each AF falls through to its own
rule. The `le '32'` / `le '64'` also caps absurdly long prefixes; tighten or
loosen to taste.

### Apply the filter to the peer-group (not per neighbor)

Because every DN42 peer shares the `dn42` peer-group, you apply the filter **once** and all peers inherit it:

```sh
set protocols bgp peer-group dn42 address-family ipv4-unicast route-map import 'DN42-PEERING'
set protocols bgp peer-group dn42 address-family ipv4-unicast route-map export 'DN42-PEERING'
set protocols bgp peer-group dn42 address-family ipv6-unicast route-map import 'DN42-PEERING'
set protocols bgp peer-group dn42 address-family ipv6-unicast route-map export 'DN42-PEERING'
commit
```

### Apply & verify

Route-maps only affect new updates, so refresh the session, then check.

```sh
vtysh -c "clear bgp * soft"
show bgp ipv4 summary
show bgp ipv6 summary
vtysh -c "show bgp ipv4 unicast 172.20.0.0/14 longer-prefixes"   # spot-check learned routes
```

Your `State/PfxRcd` count may drop slightly after filtering, that's the invalid/out-of-range prefixes being rejected, which is exactly the point.

To inspect what a specific neighbor sent you before filtering (handy when a route
you expect is missing), enable soft-reconfiguration inbound and look at the
filtered routes:

```sh
set protocols bgp neighbor fe80::5678 address-family ipv4-unicast soft-reconfiguration inbound
commit
```

```sh
vtysh -c "show bgp ipv4 unicast neighbors fe80::5678 filtered-routes"
```

## DNS Resolution (resolving `.dn42`)

Once routing works, you'll want to resolve DN42 names like `git.dn42`. DN42 runs
recursive resolvers (for example `172.20.0.53` and `172.23.0.53`) that also
resolve the clearnet, so they can serve as your only upstream if you want
everything to go through DN42.

We run a small DNS forwarder on the router itself. LAN clients query the router,
and the router forwards to the DN42 resolvers.

Two things to get right, both reflected in the config below:

Queries the router generates itself leave through the WireGuard tunnel, but the
kernel has no DN42 IPv4 on that interface, so by default it picks your public WAN
IP as the source. The DN42 resolver then has nowhere to send its reply and the
query times out. `source-address` forces the forwarder to source from your DN42
LAN address.

Use the **global** `name-server` form, not conditional `domain` forwarding.
VyOS turns `domain` entries into pdns `forward-zones=` (RD=0), which asks the
upstream only for what it is authoritative for. The DN42 resolvers are recursive,
not authoritative for `.dn42`, so they answer `REFUSED`, surfaced to the client
as `SERVFAIL`. Global `name-server` entries become `forward-zones-recurse`
(RD=1, the `+` prefix in the generated zone file), which is what recursive
upstreams expect. Since we want all DNS to go through DN42 anyway, the global
form is both simpler and correct.

### Config

```sh
# Listen on the router's DN42 LAN address; only the LAN may query us
set service dns forwarding listen-address '172.20.20.1'
set service dns forwarding allow-from '172.20.20.0/27'

# Source outgoing queries from our DN42 address
set service dns forwarding source-address '172.20.20.1'

# Global upstreams = DN42 recursive resolvers (no 'domain' entries).
# These resolve both .dn42 and the clearnet.
set service dns forwarding name-server '172.20.0.53'
set service dns forwarding name-server '172.23.0.53'

# DN42 resolvers are not DNSSEC-signed to the ICANN root; validation would
# SERVFAIL .dn42, so turn it off on the forwarder.
set service dns forwarding dnssec 'off'
commit
```

### Verify

From the router, query your own forwarder explicitly (not the system resolver):

```sh
dig @172.20.20.1 git.dn42 A      # expect NOERROR + an answer
dig @172.20.20.1 example.com A   # clearnet must work too
```

If you get `SERVFAIL`, check the generated zone file: the `.dn42` upstream must be
the catch-all `+.` line (recursive). A separate `dn42=` line without the `+`
prefix means a stray `domain` entry slipped in and should be removed.

```sh
sudo cat /run/pdns-recursor/recursor.forward-zones.conf
# +.=172.20.0.53:53, 172.23.0.53:53     <- good, recursive
# dn42=172.20.0.53:53, ...              <- bad, non-recursive, remove the 'domain' entry
```

To confirm the source address is correct, capture an outgoing query; the source
must be your DN42 address, not your public WAN IP:

```sh
sudo tcpdump -ni any port 53 and host 172.20.0.53
# 172.20.20.1.xxxxx > 172.20.0.53.53   <- good
# <public IP>.xxxxx > 172.20.0.53.53   <- bad, source-address not applied
```

### Hand DNS to LAN clients

Point your DHCP clients at the forwarder and drop the clearnet resolvers (which
don't know `.dn42`):

```sh
set service dhcp-server shared-network-name LAN subnet 172.20.20.0/27 option name-server '172.20.20.1'
commit
```

With a single peer, all DNS (clearnet included) now depends on that BGP session
and the DN42 resolvers. If the peer drops, the LAN loses DNS entirely. Add a
second peer (and ideally resolvers reachable over different paths) before relying
on this, or keep one clearnet resolver lower in the list as a fallback, at the
cost of it not resolving `.dn42`.

## Credits

This How-To has to be considered a work-in-progress by **mathys-lopinto**
The doc was updated with the help of Claude AI.

It's based on the original VyOS How-To made by **Matwolf** and **bri**: [How-To/vyos1.4.x](/howto/vyos1.4.x).

The commands in this page have been adapted to be compatible with the new version of VyOS 1.5.x (circinus) and to include configurations for IPv6 (MP-BGP over link-local and extended next-hop).

If you have any questions or suggestions please reach out.

## See also

[WireGuard](https://docs.vyos.io/en/latest/configuration/interfaces/wireguard.html) and [BGP](https://docs.vyos.io/en/latest/configuration/protocols/bgp.html) in the official VyOS documentation.
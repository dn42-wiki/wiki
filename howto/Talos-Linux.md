# Talos Linux

This guide sets up a DN42 router on a [Talos Linux](https://www.talos.dev/) node, using WireGuard tunnels and BIRD. It covers what's specific to Talos; for BIRD itself, see the [Bird2](/howto/Bird2) guide.

Talos is different from a normal Linux distribution: there is no shell, no SSH and no package manager. You configure everything with YAML documents, sent to the node with `talosctl`. Software that isn't part of Talos, like BIRD, comes in as a *system extension* baked into the Talos image.

Tested with Talos v1.15.0-alpha.2 and the `siderolabs/bird2` 2.18 extension.

## Before you start

You need:

* A registered DN42 AS, prefix and ROA, see [Getting Started](/howto/Getting-Started). This guide uses **AS4242421234** and **fd00:1234:5678::/48** as examples; replace them with your own everywhere.
* A Talos node you can manage with `talosctl`, with a public IPv6 address peers can reach.
* `wg` (wireguard-tools) on your workstation, to generate keys.
* At least one peer, see [Wireguard](/howto/wireguard) for what to exchange with them.

The steps:

1. Add BIRD to the Talos image
2. Create a WireGuard tunnel to your peer
3. Configure BIRD
4. Check that it works
5. Optional, for Kubernetes nodes: move DN42 into its own routing table (VRF)

## 1. Add BIRD to the Talos image

Talos images are built by the [Image Factory](https://factory.talos.dev/) from a *schematic* that lists the extensions. Create `schematic.yaml` (keep any extensions you already use):

```yaml
customization:
  systemExtensions:
    officialExtensions:
      - siderolabs/bird2
```

Register it; the response contains the schematic ID:

```
curl -X POST --data-binary @schematic.yaml https://factory.talos.dev/schematics
```

Upgrade the node to an installer image built from that ID. This reboots the node. Use your Talos version, and the installer type that matches your node (`metal-installer` for bare metal):

```
talosctl -n <node> upgrade --image factory.talos.dev/metal-installer/<schematic id>:v1.15.0-alpha.2
```

After the reboot, `talosctl -n <node> get extensions` lists `bird2`. BIRD itself starts once it has a config (step 3).

## 2. Create a WireGuard tunnel

Generate a key pair. Send the public key to your peer, together with your endpoint (`[<node IPv6>]:<port>`) and your link-local address:

```
wg genkey | tee private.key | wg pubkey > public.key
```

Create `wg-example.yaml`, one per peer. The link-local addresses are agreed with the peer; WireGuard endpoints must be IP addresses, not hostnames.

```yaml
apiVersion: v1alpha1
kind: WireguardConfig
name: wg-example
privateKey: <contents of private.key>
listenPort: 21234
addresses:
  - address: fe80::1234/64 # your side of the tunnel
peers:
  - publicKey: <peer public key>
    endpoint: "[2001:db8::1]:21234" # the peer's endpoint
    persistentKeepaliveInterval: 25s
    allowedIPs:
      - fe80::/10
      - fd00::/8
---
# Don't let the peer configure this node through router advertisements or redirects.
machine:
  sysctls:
    net.ipv6.conf.wg-example.accept_ra: "0"
    net.ipv6.conf.wg-example.accept_redirects: "0"
```

Apply it:

```
talosctl -n <node> patch machineconfig --patch @wg-example.yaml
```

The private key is now in your machine config. If you keep your config in git, encrypt it, for example with SOPS.

Check the tunnel with `talosctl -n <node> get links wg-example`, and by pinging the peer's link-local address from your peer's side.

## 3. Configure BIRD

BIRD's config is passed in an `ExtensionServiceConfig` document. The config below is a trimmed-down version of the one in the [Bird2](/howto/Bird2) guide, which explains the filters and options in more detail. Create `bird.yaml`:

```yaml
apiVersion: v1alpha1
kind: ExtensionServiceConfig
name: bird2
configFiles:
  - mountPath: /usr/local/etc/bird.conf
    content: |
      define OWNAS = 4242421234;
      define OWNNETv6 = fd00:1234:5678::/48;
      define OWNNETSETv6 = [fd00:1234:5678::/48+];

      # Any unique IPv4 address. Use your DN42 IPv4 address if you have one.
      router id 10.12.34.1;
      log stderr all;

      protocol device {
        scan time 10;
      }

      function is_self_net_v6() -> bool {
        return net ~ OWNNETSETv6;
      }

      function is_valid_network_v6() -> bool {
        return net ~ [ fd00::/8{44,64} ];
      }

      # ROAs, see "ROA validation" below
      roa6 table dn42_roa6;

      protocol rpki rtr {
        roa6 { table dn42_roa6; };
        remote 2001:db8::8282 port 8282;
        refresh 600;
        retry 30;
        expire 7200;
      }

      # Announce your own prefix.
      protocol static {
        route OWNNETv6 unreachable;
        ipv6 {
          import all;
          export none;
        };
      }

      # Install learned routes into the kernel.
      protocol kernel {
        scan time 20;
        learn off;
        ipv6 {
          import none;
          export filter {
            if source = RTS_STATIC then reject;
            accept;
          };
        };
      }

      template bgp dnpeers {
        local as OWNAS;
        path metric on;
        ipv6 {
          # re-check received routes when the ROAs change
          import table on;
          import filter {
            if !is_valid_network_v6() || is_self_net_v6() then reject;
            if roa_check(dn42_roa6, net, bgp_path.last) != ROA_VALID then {
              print "[dn42] ROA check failed for ", net, " ASN ", bgp_path.last;
              reject;
            }
            accept;
          };
          export filter {
            if is_valid_network_v6() && source ~ [RTS_STATIC, RTS_BGP] then accept;
            reject;
          };
          import limit 9000 action block;
        };
      }

      # One per peer: their link-local address and AS, over their tunnel.
      protocol bgp example from dnpeers {
        neighbor fe80::4321 as 4242424321;
        interface "wg-example";
      }
```

Apply it:

```
talosctl -n <node> patch machineconfig --patch @bird.yaml
```

BIRD can't reload its config: every change to this document restarts it, which briefly drops all BGP sessions.

### ROA validation

BIRD checks routes against the DN42 ROAs, which it fetches from an *RTR server* (`remote` in the `rpki` block). Talos can't run a cron job to download a ROA file, so run a small RTR server somewhere your node can reach, for example [stayrtr](https://github.com/bgp/stayrtr) with the DN42 feed:

```
stayrtr -cache=https://dn42.burble.com/roa/dn42_roa_46.json -checktime=false
```

It listens on port 8282. Until BIRD has ROAs, every route fails the check and is rejected.

### Reaching DN42 from your LAN

Your LAN router needs a route for `fd00::/8` to the Talos node, and your LAN hosts need addresses from your DN42 prefix.

## 4. Check that it works

```
talosctl -n <node> service ext-bird2
talosctl -n <node> logs ext-bird2
```

The service should be `Running`. The log shows rejected routes as `ROA check failed` lines, which is normal: not every route in DN42 has a valid ROA.

For the full picture, use `birdc`. Its control socket is `/var/run/bird/bird.ctl` on the node. On a Kubernetes node, from a debug pod:

```
kubectl debug node/<node> -it --image=alpine --profile=sysadmin
apk add bird2
birdc -s /host/var/run/bird/bird.ctl show protocols
```

The BGP protocol should show `Established`.

## 5. Optional: a VRF on Kubernetes nodes

With the setup above, DN42 routes go into the node's main routing table. That's fine on a dedicated router. On a Kubernetes node, the node and all its pods then reach DN42 directly. And because pod and service networks are often ULAs too (`fd00::/8`), a peer announcing an overlapping prefix could pull cluster traffic into a tunnel.

A VRF gives DN42 its own routing table, so the node and its pods never see DN42 routes. It isn't sealed off: it has exactly two doors, and both lead to your LAN router.

```
LAN host
  │  default route
  ▼
LAN router ── route fd00::/8 → Talos node
  │
  ▼  arrives on eth0
Talos node ── rule: "came in on eth0, going to fd00::/8 → use the DN42 table"   ← way in
  │
  ▼  DN42 table: routes from BIRD
wg-example ──► DN42

reply from DN42
  │
  ▼  arrives on wg-example → DN42 table
Talos node ── DN42 table: "your /48 → LAN router via eth0"                     ← way out
  │
  ▼
LAN router ──► LAN host
```

For background, see the Linux kernel's [VRF documentation](https://docs.kernel.org/networking/vrf.html), Talos's [VRFConfig](https://docs.siderolabs.com/talos/v1.15/reference/configuration/network/vrfconfig) and [RoutingRuleConfig](https://docs.siderolabs.com/talos/v1.15/reference/configuration/network/routingruleconfig) references, and BIRD's [`vrf` option](https://bird.network.cz/?get_doc&v=20&f=bird-3.html#proto-vrf).

Anything in the DN42 table that isn't your prefix or a DN42 route is dropped. So DN42 can't reach the node or its pods, and your LAN router's firewall sees all DN42 traffic in both directions. Pods on the node reach DN42 like any other LAN host: via the LAN router, which sends it back through the way in.

Add these documents, replacing `eth0`, the router's link-local address and the pod range with yours. Talos only accepts table numbers 1–252.

```yaml
apiVersion: v1alpha1
kind: VRFConfig
name: vrf-dn42
table: "42"
links:
  - wg-example # every tunnel
---
# Way out: your prefix goes back to the LAN router.
apiVersion: v1alpha1
kind: LinkConfig
name: eth0
routes:
  - destination: fd00:1234:5678::/48
    gateway: fe80::1 # LAN router
    table: "42"
---
# Way in: DN42 traffic from the LAN router uses the DN42 table...
apiVersion: v1alpha1
kind: RoutingRuleConfig
name: "210"
iifName: eth0
dst: fd00::/8
table: "42"
---
# ...except traffic for the cluster's own ULA ranges (lower number = checked first).
apiVersion: v1alpha1
kind: RoutingRuleConfig
name: "200"
iifName: eth0
dst: fd10:1111:2222::/48 # pod range
table: main
---
# Drop everything else in the DN42 table. Without this, lookups that miss it
# fall through to the main table. Talos can't add "unreachable" routes to a
# VRF table, so the default route points into a dummy link, which drops it.
apiVersion: v1alpha1
kind: DummyLinkConfig
name: dn42-null
routes:
  - gateway: fe80::1
    metric: 4278198272
    table: "42"
```

Then tell BIRD to use the VRF: add these lines to the `protocol kernel` block and to the `template bgp dnpeers` block, and reject the cluster ranges in the import filter.

```
protocol kernel {
  vrf "vrf-dn42";
  kernel table 42;
  ...
}

template bgp dnpeers {
  vrf "vrf-dn42";
  ...
}
```

```
# the cluster's pod and service ranges
function is_cluster_net_v6() -> bool {
  return net ~ [ fd10:1111:2222::/48+, fd10:3333:4444::/112+ ];
}

  if !is_valid_network_v6() || is_self_net_v6() || is_cluster_net_v6() then reject;
```

Leave `net.ipv4.tcp_l3mdev_accept` and `udp_l3mdev_accept` at their default of `0`, so services on the node can't be reached from the VRF.

## Why not Talos's built-in BGP?

Talos has its own BGP support (`BGPInstanceConfig`), but it doesn't work for DN42 yet:

* it has no route filters or ROA validation, so it accepts anything a peer sends ([siderolabs/talos#14252](https://github.com/siderolabs/talos/issues/14252));
* it can't peer over a WireGuard tunnel. The tunnel itself is fine, but a neighbor is either a `link` or an `address`, not both like BIRD's `neighbor fe80::… interface "wg-…"`. With `link:` Talos waits for neighbor discovery, which WireGuard doesn't do; with a link-local `address:` the session comes up, but routes with a link-local next hop aren't installed.

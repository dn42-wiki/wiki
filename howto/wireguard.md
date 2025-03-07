To quote the [homepage](https://www.wireguard.io/):

> WireGuard is an extremely simple yet fast and modern VPN that utilizes state-of-the-art cryptography. It aims to be faster, simpler, leaner, and more useful than IPSec, while avoiding the massive headache. It intends to be considerably more performant than OpenVPN. WireGuard is designed as a general purpose VPN for running on embedded interfaces and super computers alike, fit for many different circumstances. Initially released for the Linux kernel, it plans to be cross-platform and widely deployable. It is currently under heavy development, but already it might be regarded as the most secure, easiest to use, and simplest VPN solution in the industry.

# Example configuration for dn42

Wireguard is a Layer3 VPN. In theory it allows multiple peers to be served with one interface/port, but it does internal routing based on the peer's public key. This means **you will need one interface per peering** on dn42
to allow your BGP daemon instead to do routing. This approach is comparable to [OpenVPN p2p tunnels](/howto/openvpn).

First generate on each peer public and private keys.

```sh
$ wg genkey | tee privatekey | wg pubkey > publickey
```

## Configuration

```conf
# tunnel.conf
[Interface]
PrivateKey = <private_key>
ListenPort = <YOUR_LOCAL_UDP_PORT>

[Peer]
PublicKey = <public_key_of_your_peer>
# OPTIONAL, its also possible to define a pre-shared key for additional security
PresharedKey = <pre-shared key>
# at least one peer needs to provide this one
Endpoint = <end_point_hostname_or_ip:port>
# in theory this could be restricted to dn42 networks,
# however it is easier to do this with iptables/bgp filters/routing table 
# instead just like for openvpn-based peerings
AllowedIPs = 0.0.0.0/0,::/0
```

**Make sure that your AllowedIPs include the full dn42 ranges (`172.20.0.0/14`, `fd00::/8`) and not just your peer's next hop IPs!** AllowedIPs functions as a data plane restriction on which target IPs can go over each WireGuard tunnel. If this is misconfigured, you may see errors such as: `ping: sendmsg: Destination address required`.

## Configure tunnel:

Wireguard comes with its own interface type. 
It supports link-local addresses for IPv6 and single /32 addresses for IPv4, which can be used for peering.

```sh
$ ip link add dev <interface_name> type wireguard
$ wg setconf <interface_name> tunnel.conf
# both side pick a different link-local ipv6 address
$ ip addr add fe80::<some_random_suffix>/64 dev <interface_name>
# choose the first ip from your subnet and the second one from the peer
$ ip addr add 172.xx.xx.xx/32 peer 172.xx.xx.xx/32 dev <interface_name>
$ ip link set <interface_name> up
```

<!-- Nurtic-Vibe has another [script](https://git.dn42.us/Nurtic-Vibe/grmml-helper/src/master/create_wg.sh) to interactively automate the peering process. -->

Maybe you should check the MTU to your peer with e.g. `ping -s 1472 <end_point_hostname_or_ip>`. If your output looks like `From gateway.local (192.168.0.1) icmp_seq=1 Frag needed and DF set (mtu = 1440)` substract `80` from the MTU and set it via `ip link set dev <interface_name> mtu <calculated_mtu>`

## Testing

```sh
ping fe80::<your_peers_suffix>%<interface_name>
```

(For older iputils, use `ping6`.)

Afterwards configure your [BGP session](/howto/Bird2) as usual

## Debugging

The wireguard kernel module on linux has support for enabling dynamic debugging. This can be useful to help track down some problems, e.g. if public keys don't match. 

Debug messages are logged via dmesg and can be enabled using:

```sh
$ echo 'module wireguard +p' > /sys/kernel/debug/dynamic_debug/control
```

To disable debug:

```sh
$ echo 'module wireguard -p' > /sys/kernel/debug/dynamic_debug/control
```

## wg-quick

[wg-quick](https://git.zx2c4.com/wireguard-tools/about/src/man/wg-quick.8) is a script that is shipped with Wireguard to help users bring up tunnels in some common use cases. 

> It is designed for users with simple needs, and users with more advanced needs are highly encouraged to use a more specific tool, a more complete network manager, or otherwise just use wg(8) and ip(8), as usual.

The script makes some changes that are not valid when used for DN42 tunnels, and which must be worked around:

- By default, the script will add a routing policy that routes the 'AllowedIP' ranges through the tunnel. In DN42, route selection is managed by BGP so the routing policy *must* be removed to avoid problems. This is achieved by adding the '_Table = off_' directive. 

  - **Warning: a common pattern for DN42 tunnels is to use `AllowedIPs = 0.0.0.0/0` or `AllowedIPs = ::/0` then use firewall rules to limit source and destination addresses. If you do not add 'Table = off' this could cause you to route clearnet traffic via your peer and potentially lose connectivity to your node!**

- It is common in DN42 to use Point-to-Point addressing schemes on tunnel interfaces (that is, using IPv4/32 and IPv6/128 addresses); this is not supported by wg-quick. To configure PTP addresses you must add a '_PostUp_' statement. On Linux, this will typically be done using `ip` from `iproute2`.

An example wg-quick script that incorporates the above two workarounds is below, where `<MyIPv[46]>` are the DN42 IP addresses of your node and `<PeerIPv[46]>` are the IP addresses for your peer. 

```conf
[Interface]
PrivateKey = <your private key>
Address = <your link-local address, if any>
PostUp = /sbin/ip addr add dev %i <MyIPv4>/32 peer <PeerIPv4>/32
PostUp = /sbin/ip addr add dev %i <MyIPv6>/128 peer <PeerIPv6>/128
Table = off

[Peer]
Endpoint = <your peer's wireguard endpoint>
PublicKey = <your peer's public key>
AllowedIPs = 172.16.0.0/12, 10.0.0.0/8, fd00::/8, fe80::/10
```
Use `which ip` to get the full path to your ip binary.

## systemd-networkd

Example configuration for systemd-networkd.

peer.netdev
```conf
[NetDev]
Name=<ifname>
Kind=wireguard

[WireGuard]
PrivateKey=<your private key>
ListenPort=<your listen port>

[WireGuardPeer]
PublicKey=<peer public key>
# OPTIONAL, pre-shared key
PresharedKey=<pre-shared key>
Endpoint=<peer host and port, e.g. 1.2.3.4:9876>
AllowedIPs=fe80::/64
AllowedIPs=fd00::/8
AllowedIPs=0.0.0.0/0
```

peer.network
```conf
[Match]
Name=<ifname>

[Network]
DHCP=no
IPv6AcceptRA=false
IPForward=yes
IPv4ReversePathFilter=no  # required if sysctl item `net.ipv4.conf.default.rp_filter` is not 0


# for networkd >= 244 KeepConfiguration stops networkd from
# removing routes on this interface when restarting
KeepConfiguration=yes

# for networkd < 244 the CriticalConnection parameter achieves
# the same thing
[DHCP]
CriticalConnection=true

# if using link local addresses for peering
[Address]
Address=fe80::xx:xx:xx:xx/64

# if using IPv6 point to point
[Address]
Address=<your ipv6 address>/128
Peer=<your peer's IPv6 address>/128

# IPv4 point to point
[Address]
Address=<your IPv4 address>/32
Peer=<your peer's IPv4 address>/32
```

## Dynamics IP

As wireguard are only resolving the hostname to IP only on start, dynamics DNS will stop working after a while without further configuration. The Following is a [script](https://github.com/WireGuard/wireguard-tools/blob/master/contrib/reresolve-dns/reresolve-dns.sh) from wireguard which will "re-resolve" the DNS and update the wireguard. 

You can add cron entries to periodically  "re-resolve" the DNS:
```sh
* * * * * /path-to-the-script/reresolve-dns.sh
```

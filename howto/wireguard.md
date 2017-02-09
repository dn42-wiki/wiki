To quote the [homepage](https://www.wireguard.io/):

> WireGuard is an extremely simple yet fast and modern VPN that utilizes state-of-the-art cryptography. It aims to be faster, simpler, leaner, and more useful than IPSec, while avoiding the massive headache. It intends to be considerably more performant than OpenVPN. WireGuard is designed as a general purpose VPN for running on embedded interfaces and super computers alike, fit for many different circumstances. Initially released for the Linux kernel, it plans to be cross-platform and widely deployable. It is currently under heavy development, but already it might be regarded as the most secure, easiest to use, and simplest VPN solution in the industry.

# Example configuration for dn42

Wireguard is a Layer3 VPN. In theory it allows multiple peers to be served with one interface/port, but it does internal routing based on the peer's public key. This means you will need one interface per peering on dn42
to allow your BGP daemon instead to do routing. This approach is comparable to [OpenVPN p2p tunnels](/howto/openvpn).

First generate on each peer public and private keys.

```
$ wg genkey | tee privatekey | wg pubkey > publickey
```

## Configuration

```
# tunnel.conf
[Interface]
PrivateKey = <private_key>
ListenPort = <YOUR_LOCAL_UDP_PORT>

[Peer]
PublicKey = <public_key_of_your_peer>
# at least one peer needs to provide this one
Endpoint = <end_point_hostname_or_ip:port>
# in theory this could be restricted to dn42 networks,
# however it is easier to do this with iptables/bgp filters/routing table 
# instead just like for openvpn-based peerings
AllowedIPs = 0.0.0.0/0,::/0
```

## Configure tunnel:

Wireguard comes with its own interface type. 
It supports link-local addresses ipv6 and single /32 addresses for ipv4, which can be used for peering.

```
$ ip link add dev <interface_name> type wireguard
$ wg setconf <interface_name> tunnel.conf
# both side pick a different link-local ipv6 address
$ ip addr add fe80::<some_random_suffix>/64 dev <interface_name>
# choose the first ip from your subnet and the second one from the peer
$ ip addr add 172.xx.xx.xx/32 peer 172.xx.xx.xx/32 dev <interface_name>
$ ip link set <interface_name> up
```

Mic92 uses this [script](https://github.com/Mic92/bird-dn42/tree/master/wireguard) to automate this

Maybe you should check the MTU to your peer with e.g. `ping -s 1472 <end_point_hostname_or_ip>`. If your output looks like `From gateway.local (192.168.0.1) icmp_seq=1 Frag needed and DF set (mtu = <MTU>)` substract `80` from the MTU and set it via `ip link set dev <interface_name> mtu <calculated_mtu>`

## Testing

```
ping fe80::<your_peers_suffix>%<interface_name>
```

(For older iputils, use `ping6`.)

Afterwards configure your [BGP session](/howto/Bird) as usual



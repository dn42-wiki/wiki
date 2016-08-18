To quote the [homepage](https://www.wireguard.io/):

> WireGuard is an extremely simple yet fast and modern VPN that utilizes state-of-the-art cryptography. It aims to be faster, simpler, leaner, and more useful than IPSec, while avoiding the massive headache. It intends to be considerably more performant than OpenVPN. WireGuard is designed as a general purpose VPN for running on embedded interfaces and super computers alike, fit for many different circumstances. Initially released for the Linux kernel, it plans to be cross-platform and widely deployable. It is currently under heavy development, but already it might be regarded as the most secure, easiest to use, and simplest VPN solution in the industry.

# Example configuration for dn42

Wireguard is a Layer3 VPN. In theory it allows multiple peers to be served with one interface/port, but it does internal routing based on the public key of the peers. This means you will need one interface per peering on dn42
to allow your BGP deamon instead to do routing. This approach is comparable to [openvpn p2p tunnels](/howto/openvpn).

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
Endpoint = <end_post_hostname_or_ip:port>
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

## Testing

```
ping6 fe80::<you_peers_suffix> -I <interface_name>
```

or with new iputils without ping6

```
ping fe80::<you_peers_suffix>%<interface_name>
```

Afterwards configure you [BGP](/howto/Bird) as usual



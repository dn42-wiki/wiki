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
It supports link-local addresses for IPv6 and single /32 addresses for IPv4, which can be used for peering.

```
$ ip link add dev <interface_name> type wireguard
$ wg setconf <interface_name> tunnel.conf
# both side pick a different link-local ipv6 address
$ ip addr add fe80::<some_random_suffix>/64 dev <interface_name>
# choose the first ip from your subnet and the second one from the peer
$ ip addr add 172.xx.xx.xx/32 peer 172.xx.xx.xx/32 dev <interface_name>
$ ip link set <interface_name> up
```
  
Nurtic-Vibe has another [script](https://git.dn42.us/Nurtic-Vibe/grmml-helper/src/master/create_wg.sh) to interactively automate the peering process.

Maybe you should check the MTU to your peer with e.g. `ping -s 1472 <end_point_hostname_or_ip>`. If your output looks like `From gateway.local (192.168.0.1) icmp_seq=1 Frag needed and DF set (mtu = 1440)` substract `80` from the MTU and set it via `ip link set dev <interface_name> mtu <calculated_mtu>`

## Testing

```
ping fe80::<your_peers_suffix>%<interface_name>
```

(For older iputils, use `ping6`.)

Afterwards configure your [BGP session](/howto/Bird) as usual

## wg-quick

[wg-quick](https://git.zx2c4.com/wireguard-tools/about/src/man/wg-quick.8) is a script that is shipped with Wireguard to help users bring up tunnels in some common use cases. 

> It is designed for users with simple needs, and users with more advanced needs are highly encouraged to use a more specific tool, a more complete network manager, or otherwise just use wg(8) and ip(8), as usual.

The script makes some changes that are not valid when used for DN42 tunnels, and which must be worked around:

- By default, the script will add a routing policy that routes the 'AllowedIP' ranges through the tunnel. In DN42, route selection is managed by BGP so the routing policy *must* be removed to avoid problems. This is achieved by adding the '_Table = off_' directive. 

  - **Warning: a common pattern for DN42 tunnels is to use `AllowedIPs = 0.0.0.0/0` or `AllowedIPs = ::/0` then use firewall rules to limit source and destination addresses. If you do not add 'Table = off' this could cause you to route clearnet traffic via your peer and potentially lose connectivity to your node!**

- It is common in DN42 to use Point-to-Point addressing schemes on tunnel interfaces (that is, using IPv4/32 and IPv6/128 addresses); this is not supported by wg-quick. To configure PTP addresses you must add a '_PostUp_' statement that first removes the addresses that wg-quick has configured and then re-add them. On Linux, this will typically be done using `ip` from `iproute2`.

An example wg-quick script that incorporates the above two workarounds is below, where `<MyIPv[46]>` are the DN42 IP addresses of your node and `<PeerIPv[46]>` are the IP addresses for your peer. 

```
[Interface]
PrivateKey = <your private key>
Address = <MyIPv4>/32, <MyIPv6>/128
PostUp = /sbin/ip addr del dev wg0 <MyIPv4>/32 && /sbin/ip addr add dev wg0 <MyIPv4>/32 peer <PeerIPv4>/32 && /sbin/ip addr del dev wg0 <MyIPv6>/128 && /sbin/ip addr add dev wg0 <MyIPv6>/128 peer <PeerIPv6>/128
Table = off
 
[Peer]
Endpoint = <your peer's wireguard endpoint>
PublicKey = <your peer's public key>
AllowedIPs = 172.16.0.0/12
AllowedIPs = 10.0.0.0/8
AllowedIPs = fd00::/8
AllowedIPs = fe80::/10
```
Use `which ip` to get the full path to your ip binary.


# systemd-networkd configuration example
This is the config that is used on ZOTAN Networks (AS4242422341). Full network configuration available on [my Git](https://git.zotan.dn42/zotan/dn42) (dn42) or alternatively [my Git](https://git.prod.zotan.network/zotan/dn42) (clear)


# Configuration

## loopback device (lo.network)
```
[Match]
Name=lo

[Network]
Address=fdff:b02d:2ef7::2/128
```

## wireguard netdev (dn42p1.netdev)
```
[NetDev]
Name = dn42p1
Kind = wireguard
Description = WireGuard

[WireGuard]
ListenPort = 42421
PrivateKeyFile = /etc/wireguard/private.key

[WireGuardPeer]
PublicKey = <peer wg pubkey>
Endpoint = <peer wg endpoint>:<peer wg port>
AllowedIPs = 172.16.0.0/12,10.0.0.0/8,fd00::/8,fe80::/10,ff00::/8
```

## wireguard network (dn42p1.network)
```
[Match]
Name = dn42p1

[Address]
Address = fe80::2342/128 # arbitrary, doesn't need to be unique for each interface
Peer = <peer tunnel linklocal address>/128

[Address]
Address = <your DN42 ipv4>/32
Peer = <peer DN42 ipv4>/32

```

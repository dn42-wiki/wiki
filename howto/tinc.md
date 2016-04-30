[Tinc](http://www.tinc-vpn.org/) is a meshing VPN daemon. It allows multiple parties to connect and discover each other independently, while minimizing points of failure. Tinc will use a bunch of nodes to build the network graph, which in return all nodes use to learn addresses for each other. If nodes want to reach each other they establish a direct connection to each other. If that is not possible traffic may be routed via a shared neighbour. Tinc is most notably powering the Freifunk communitys [ICVPN](https://github.com/freifunk/icvpn) (in L2/Switch-Mode) and [ChaosVPN](http://wiki.hamburg.ccc.de/ChaosVPN) (in L3/Router-Mode).

Tinc primarily operates in two modes: router and switch. A third mode, the hub mode, exists, but it's just a dumb router mode that keeps no routing table and broadcasts everything - don't use it.
In Router mode each peer announces the addresses/subnets it serves. Tinc will spawn an interface on which it will act as a L3 network, routing according to announcements. This is the default mode, but it is unsuitable for dn42, because you cannot influence how tinc will route to a certain network. In Switch mode tinc will act like a L2 network, in which the routing table reflects the peers mac addresses.

One advantage of tinc is that you can have multiple peering over the same VPN configuration by opening multiple connections.


## Configuration

Example `/etc/tinc/dn42_yourpeer/tinc.conf`:

```
Interface = dn42_yourpeer
Name = your_host
# Only switch mode is feasible for dn42 peerings, since in router mode tinc takes care of routing decisions on its own
Mode = switch
# To discover other hosts, it is required to initially specify a number of hosts to connect to. ConnectTo can be specified multiple times.
ConnectTo = remote_host
# In newer versions (>= 1.1) you can use AutoConnect instead
#AutoConnect = yes
```

Tinc requires to add manually ip addresses and routes to the tap/tun interfaces. On startup it will execute `/etc/tinc/tinc-up` if it exists **and** is executable:

Example `/etc/tinc/dn42_yourpeer/tinc-up`:

**Linux/iproute2**
```
#!/bin/sh

# set the interface up
ip link set dev $INTERFACE up

# add transfer networks
ip -4 addr add 172.16.0.1/30 dev $INTERFACE scope link
ip -6 addr add fe80::1/64 dev $INTERFACE

# add routes
ip -4 route add 172.16.0.1/30 dev $INTERFACE table peers
```

For authentication tinc uses public key authentication instead of certificates or pre-shared keys.
For each key tinc should connect to or allow to connect, a file with the name of the peer in tincd -n twwh -K
is required. To generate a public/private key pair use:

```
$ tincd -K
```

Import for each other party the key like this `/etc/tinc/dn42_yourpeer/hosts/<peername>`:

```
# address/port are optional, in case they're missing you only expect connections from that host
Address = <fqdn/ip_addr>
Port = <port|655>
-----BEGIN RSA PUBLIC KEY-----
MIIBCgKCAQEAoGeD5b1HKW2UAFpIPayxsOOYx5qC0oHrJnvcPH33jnDBGiOYJ9ma
QZErWdF0Qsnqh/wJE6i569fzKWOUdLHrN5dVzD/Q5zjMOwJf3rmcerS0oAFTxKDj
pkw2kKcLA/lSNMIN//W66mM258BLo1XgEraUx5RcJ4hTxawhNTn0NTJVCbfUX6e5
tcJpbgbYRzBTUPdSL3OB8k0qlmFI2ZYTnCzOSpgxRQARIB1ecoqOYVxQISK2pzxi
MHQQlVbquwldaKiVoj7tD7PFW4oQxpiMHZnHIA6dnZCsT3ktTOzCjhf2XMi8o8u5
P9C5dYrmVWrVAWQznlbuq/w1z+PrTYquoQIDAQAB
-----END RSA PUBLIC KEY-----
```

## Fun with tinc-pre

The current development version (which is pretty stable by the way), allow to bootstrap networks using invitation urls. Instead of rsa keys it uses ed25519 keys. To keep backwards compatibility with the tinc 1.0 release you need rsa keys, if you don't need that only generate ed25519 keys. It also introduces the tinc binary in addition to tincd, which allows tinc to be configured via an readline interface.

Installation:
* Archlinux: install [tinc-pre](https://aur.archlinux.org/packages/tinc-pre) from AUR
* Debian: follow these [instructions](https://gist.github.com/mweinelt/efff4fb7eba1ee41ef2d) to get a package
* Freebsd: Use this [port repo](https://github.com/Mic92/ports/tree/master/security/tinc)

Set up a new tinc network
```
# tinc init dn42_yourpeer
```

Invite your peering partner. Tinc will try print the invition which you need to copy to your peering partner.
```
$ tinc invite yourpeer
<ip-or-address>/nIRp5pJCnfnhuV13JUomscGs1q5HqEbz3AydZer7wRaMcpUB
```

On the other node you can join by using:

```
$ tinc join <invitation-url>
```

This node will then automatically generate configuration, private/public keys and will exchange this key with the other node on connection.

Remember to still set up your **tinc-up** script.
[Tinc](http://www.tinc-vpn.org/) is a meshing VPN daemon. It allows multiple parties to connect and discover each other independently, while minimizing points of failure. Tinc will use a bunch of nodes to build the network graph, which in return all nodes use to learn addresses for each other. If nodes want to reach each other they establish a direct connection to each other. If that is not possible traffic may be routed via a shared neighbour. Tinc is most notably powering the Freifunk communitys [ICVPN](https://github.com/freifunk/icvpn) (in L2/Switch-Mode) and ChaosVPN (in L3/Router-Mode).

Tinc primarily operates in two modes: router and switch. A third mode, the hub mode, exists, but it's just a dumb router mode that keeps no routing table and broadcasts everything - don't use it.
In Router mode each peer announces the addresses/subnets it serves. Tinc will spawn an interface on which it will act as a L3 network, routing according to announcements. This is the default mode, but it is unsuitable for dn42, because you cannot influence how tinc will route to a certain network. In Switch mode tinc will act like a L2 network, in which the routing table reflects the peers mac addresses.

One advantage of tinc is that you can have multiple peering over the same VPN configuration by opening multiple connections.


## Configuration

Example `/etc/tinc/tinc.conf`:

```
Name = host1
Mode = switch
# To discover other hosts, 
# it is required to initially 
# specify a number of hosts to connect to.
# ConnectTo can be specified multiple times.
ConnectTo = host2
```

Tinc requires to add manually ip addresses and routes to the tap/tun interfaces. On startup it will execute `/etc/tinc/tinc-up` if it exists and is executable:

Example `/etc/tinc/tinc-up`:
```
#!/bin/sh

# these lines differs depending on the operating system in use
# on linux the following will work.
# INTERFACE is an environmental variable set by tinc, when executing this script
ip link set dev $INTERFACE up
# to peer over tinc it is convenient to an transfer net, which is exclusively on this link;
# this way you don't have to specify routes for each peer.
# the transfer network does not need to be part of dn42, 
# you can also pick a network from 192.168.0.0/14 range
ip addr add dev $INTERFACE 192.168.41.1/24 scope link
# for ipv6 you can use fixed link-local addresses 
ip addr add dev $INTERFACE fe80::1/64
```

For authentication tinc uses public key authentication instead of certificates or pre-shared keys.
For each key tinc should connect to or allow to connect, a file with the name of the peer in tincd -n twwh -K
is required. To generate a public/private key pair use:

```
$ tincd -K
```

Import for each other party the key like this `/etc/tinc/hosts/<peername>`:

```
# Address and Port can be also skipped,
# in this case the other side has to make an attempt to connect.
Address = <ip_or_dns_name>
Port = <port_if_different_from_655>
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

The current development version (which is pretty stable by the way), allow to bootstrap networks using by invitation urls. Instead of rsa keys it uses additionally ed25519 keys. It also introduces a tinc command in addition to tincd, which allows tinc to be configured via an readline interface:

On one node which is already part of the network use:
```
$ tinc invite foo
<ip-or-address>/nIRp5pJCnfnhuV13JUomscGs1q5HqEbz3AydZer7wRaMcpUB
```

On the other node you can join by using:

```
$ tinc join <invitation-url>
```

This node will then automatically generate configuration, private/public keys and will exchange this key with the other node on connection.
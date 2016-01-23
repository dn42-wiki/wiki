[Tinc](http://www.tinc-vpn.org/) is a mesh-able vpn. It allows multiple parties to connect and discover each other independently, with no single point of failure. Tinc will try to find the shortest path to the other side but can also tunnel traffic over a third node, if 2 peers cannot reach each other directly. Tinc is in use by the Freifunk community where it powers [ICVPN](https://github.com/freifunk/icvpn).
It is also used for [ChaosVPN](https://wiki.hamburg.ccc.de/ChaosVPN).

Tinc operates in 2 modes: router and switch. In Router mode each peer announce a subnet it serves. Tinc will act as a Layer3 network. This is the default mode, but unsuitable for dn42, because you cannot influence how tinc will route to a certain network. In Switch mode tinc will act like a Layer2 network. Each peer gets a MAC address assigned.

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

Installation:

* Archlinux: install [tinc-pre](https://aur.archlinux.org/packages/tinc-pre) from AUR
* Debian: follow these [instructions](https://gist.github.com/mweinelt/efff4fb7eba1ee41ef2d) to get a package
* Freebsd: Use this [port repo](https://github.com/Mic92/ports/tree/master/security/tinc)

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
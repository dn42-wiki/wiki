# GRE+IPsec

## Why GRE?
* [GRE](https://en.wikipedia.org/wiki/Generic_routing_encapsulation) provides universal encapsulation on top of IP.
* It has a smaller header than UDP.
* GRE tunnels are processed in-kernel on *nix systems.
* It's supported by hardware routers.

## Why IPsec?
* GRE provides no encryption and authentication of it's own.
* IPsec in implemented in-kernel on FreeBSD and Linux with multithreaded encryption resulting in a lower latency than userspace VPN daemons using tun/tap interfaces.

## Problems with GRE
* GRE is defined directly on top of IP.
* Broken NAPT implementations will stop GRE tunnels.

## Problems with IPsec
* ESP is defined directly on top of IP.
* NAT support was added as an aftertought to IPsec.
* IKEv1 is too complex.
* Racoon has useless error messages.

## Requirements for sane operation
* Identify your peers by X.509 certificates
* At least one peer should operate his own (Sub-)CA.

## How to configure a GRE tunnel on FreeBSD
See [GRE on FreeBSD](/howto/GRE-on-FreeBSD).

## How to configure IPsec on FreeBSD
See [IPsec on FreeBSD](/howto/IPsec-on-FreeBSD).

## How to configure GRE + IPsec on Debian
See [GRE + IPsec on Debian](/howto/IPsecWithPublicKeys/GRE-plus-IPsec-Debian).

# dn42 on OpenWRT

This page gives hints on how to participate to dn42 with an OpenWRT router. It assumes Attitude Adjustment (12.09), but you can adapt it for other versions.

The intended target is a home router, acting as the default gateway for its LAN clients. The goal is to have one or more dn42 peers, announce the LAN subnet with BGP, and thus transparently provide dn42 access to the LAN clients.

This documentation assumes that the LAN is addressed in the dn42 space (`172.22.0.0/15`), but it's not a big deal to add NAT if it's not.

## Configuration

### Peerings

Nothing fancy: use GRE tunnels, openvpn, anything. Don't forget to install the relevant packages with `opkg` (`kmod-gre` for instance).

You can't manage GRE tunnels with OpenWRT, so just create them in `/etc/rc.local` (and assign addresses if needed).

### BGP

`quagga` and `bird` are both packaged in OpenWRT. Note that quagga is split in many packages, you probably need `quagga-bgpd`, `quagga-vtysh` and `quagga-zebra`.

### Interface definition

This is needed so that OpenWRT is aware of the new interfaces (for firewall and stuff).

In `/etc/config/network`, add entries for each dn42 interface:

    config interface dn42peer1
        option ifname tun-peer1
        option proto none

### Firewall

There are two goals:

  - Allowing traffic from LAN to dn42 (and maybe from dn42 to LAN too)
  - If you have more than one peer, allowing traffic from dn42 to dn42 (forwarding)

### DNS


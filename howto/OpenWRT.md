# dn42 on OpenWRT

This page gives hints on how to participate to dn42 with an OpenWRT router. It assumes Attitude Adjustment (12.09), but you can adapt it for other versions.

The intended target is a home router, acting as the default gateway for its LAN clients. The goal is to have one or more dn42 peers, announce the LAN subnet with BGP, and thus transparently provide dn42 access to the LAN clients.

This documentation assumes that the LAN is addressed in the dn42 space (`172.22.0.0/15`), but it's not a big deal to add NAT if it's not.

## Initial configuration



## Peerings

Nothing fancy: use GRE tunnels, openvpn, anything. Don't forget to install the relevant packages with `opkg` (`kmod-gre` for instance).

You can't manage GRE tunnels with OpenWRT, so just create them in `/etc/rc.local` (and assign addresses if needed).

## BGP

`quagga` and `bird` are both packaged in OpenWRT. Note that quagga is split in many packages, you probably need `quagga-bgpd`, `quagga-vtysh` and `quagga-zebra`.

Of course, you should announce the prefix of your home network.

## Interface definition

This is needed so that OpenWRT is aware of the new interfaces (for firewall and stuff).

In `/etc/config/network`, add entries for each dn42 interface:

```conf
config interface dn42peer1
    option ifname tun-peer1
    option proto none
```

## Firewall

There are two goals:

  - Allowing traffic from LAN to dn42 (and maybe from dn42 to LAN too)
  - If you have more than one peer, allowing traffic from dn42 to dn42 (forwarding)

Everything is done in `/etc/config/firewall`.

### Zone declaration

```conf
config zone
    option name             dn42
    option network          'dn42peer1 dn42peer2 dn42peer3'
    option input            REJECT
    option output           ACCEPT
    option forward          REJECT
```

If you need to NAT your home network into dn42, you probably just need to add: 

```conf
option masq             1
```

### dn42 ↔ LAN forwarding

```conf
config forwarding                   
    option src              lan
    option dest             dn42
```

If you're confident enough, you can also forward dn42 into your LAN:

```conf
config forwarding                   
    option src              dn42
    option dest             lan
```

Or you can forward only certain ports, to certain hosts, etc (standard `config rule` stuff)

### dn42 ↔ dn42 forwarding

This is more tricky. In theory, all you have to do is to set

```conf
option forward          ACCEPT
```

in the definition of the zone. However, due to a bug in Attitude Adjustment (see <https://dev.openwrt.org/ticket/12945>), this will allow forwarding **everything everywhere**.

You have to use this patch: <https://dev.openwrt.org/changeset/35484> (monkeypatching the relevant files in `/lib` should work).

## DNS

See [DNS Configuration](/services/dns/Configuration). This will use the anycast dn42 DNS server to resolve `dn42` and relevant reverse domains.

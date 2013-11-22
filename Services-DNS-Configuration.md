# Forwarder setup

Configuration of common resolver softwares, to forward DNS queries for `.dn42` (and reverse DNS) to `172.22.0.53`.

## BIND

If you already run a local DNS server, you can tell it to query the dn42 anycast servers for the relevant domains
by adding the following to /etc/bind/named.conf.local 

```
zone "dn42" {
  type forward;
  forwarders { 172.22.0.53; };
};
zone "22.172.in-addr.arpa" {
  type forward;
  forwarders { 172.22.0.53; };
};
zone "23.172.in-addr.arpa" {
  type forward;
  forwarders { 172.22.0.53; };
};
```

## dnsmasq

If you are running dnsmasq under openwrt, you just have to add 

```
config dnsmasq
        option boguspriv '0'
        option rebind_protection '1'
        list rebind_domain 'dn42'
        list server '/dn42/172.22.0.53'
        list server '/22.172.in-addr.arpa/172.22.0.53'
        list server '/23.172.in-addr.arpa/172.22.0.53'
```

to `/etc/config/dhcp` and run `/etc/init.d/dnsmasq` restart. After that you are able to resolve `.dn42` 
with the anycast DNS-Server, while your normal requests go to your standard DNS-resolver.

For normal dnsmasq use

```
server=/dn42/172.22.0.53
server=/22.172.in-addr.arpa/172.22.0.53
server=/23.172.in-addr.arpa/172.22.0.53
```
in `dnsmasq.conf`.

## PowerDNS recursor
Add this to /etc/powerdns/recursor.conf (at least in Debian)

```
dont-query=127.0.0.0/8, 10.0.0.0/8, 192.168.0.0/16, ::1/128, fe80::/10
forward-zones= dn42=172.22.0.53,22.172.in-addr.arpa=172.22.0.53,23.172.in-addr.arpa=172.22.0.53
```

## MaraDNS
Put this in your mararc:

```
ipv4_alias["dn42_root"] = "172.22.0.53"
root_servers["dn42."] = "dn42_root"
root_servers["22.172.in-addr.arpa."] = "dn42_root"
root_servers["23.172.in-addr.arpa."] = "dn42_root"
```

## Unbound

`unbound.conf` for forwarding requests to `172.22.0.53`.


```
server:
      local-zone: "22.172.in-addr.arpa." nodefault
      local-zone: "23.172.in-addr.arpa." nodefault
      local-zone: "d.f.ip6.arpa." nodefault

forward-zone: 
      name: "dn42"
      forward-addr: 172.22.0.53

forward-zone: 
      name: "22.172.in-addr.arpa"
      forward-addr: 172.22.0.53

forward-zone: 
      name: "23.172.in-addr.arpa"
      forward-addr: 172.22.0.53

forward-zone:
      name: "d.f.ip6.arpa"
      forward-addr: 172.22.0.53
```
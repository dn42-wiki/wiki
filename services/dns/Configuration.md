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

Attention: If you go with the default config you'll have to disable "boguspriv" in the first dnsmasq config section.

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
forward-zones-recurse=dn42=172.22.0.53,hack=172.22.0.53,ffhh=172.22.0.53,ffac=172.22.0.53,020=172.22.0.53,adm=172.22.0.53,ffa=172.22.0.53,ffhb=172.22.0.53,ffc=172.22.0.53,ffda=172.22.0.53,ffdh=172.22.0.53,ff3l=172.22.0.53,fffl=172.22.0.53,ffffm=172.22.0.53,fffr=172.22.0.53,fffd=172.22.0.53,ffgl=172.22.0.53,fflln=172.22.0.53,ffbcd=172.22.0.53,ffbgl=172.22.0.53,ffgoe=172.22.0.53,ffgt=172.22.0.53,ffh=172.22.0.53,helgo=172.22.0.53,ffhef=172.22.0.53,ffj=172.22.0.53,ffka=172.22.0.53,ffki=172.22.0.53,ffhl=172.22.0.53,fflux=172.22.0.53,ffms=172.22.0.53,mueritz=172.22.0.53,ffnord=172.22.0.53,ffnw=172.22.0.53,ffoh=172.22.0.53,ffpb=172.22.0.53,ffpi=172.22.0.53,ffrade=172.22.0.53,ffrgb=172.22.0.53,ffrg=172.22.0.53,rzl=172.22.0.53,ffsaar=172.22.0.53,fftr=172.22.0.53,fftdf=172.22.0.53,ffwk=172.22.0.53,ffgro=172.22.0.53,ffwk=172.22.0.53,ffwp=172.22.0.53,ffw=172.22.0.53,20.172.in-addr.arpa=172.22.0.53,22.172.in-addr.arpa=172.22.0.53,23.172.in-addr.arpa=172.22.0.53,31.172.in-addr.arpa=172.22.0.53,c.f.ip6.arpa=172.22.0.53
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
      domain-insecure: "dn42"
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

## JunOS (SRX 12.1X46)
Should also work in 12.1X44 and 12.1X45. After making the changes below you may need to run:
```
restart named-service
```
Config (vlan.0 is presumed to be your LAN/Trust interface)
```
system {
   services {
      dns {
         dns-proxy {
            interface {
               vlan.0;
            }
	    default-domain dn42 {
	       forwarders {
	          172.22.0.53;
	       }
	    }
	    default-domain 22.172.in-addr.arpa {
               forwarders {
                  172.22.0.53;
               }
            }                       
            default-domain 23.172.in-addr.arpa {
               forwarders {
                  172.22.0.53;
               }
            }
	 }
      }
   }
}
```
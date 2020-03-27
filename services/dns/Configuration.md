# Forwarder setup

Configuration of common resolver softwares to forward DNS queries for `.dn42` (and reverse DNS) IPv4 and IPv6 anycast services.

You can use any *.delegation-servers.dn42 (where * is a letter) for resolving .dn42 domains. The current list is:

* b.delegation-servers.dn42 fd42:4242:2601:ac53::1, 172.20.129.1
* j.delegation-servers.dn42 fd42:5d71:219:1:a526:d935:281e:22d6, 172.20.1.254

The most up-to-date information is available at the [DN42 registry](https://git.dn42.us/dn42/registry/src/master/data/dns/delegation-servers.dn42)

All the examples here list 172.20.129.1, but you can use any other *.delegation-servers.dn42

## BIND

If you already run a local DNS server, you can tell it to query the dn42 anycast servers for the relevant domains
by adding the following to /etc/bind/named.conf.local

```
zone "dn42" {
  type forward;
  forwarders { 172.20.129.1; fd42:4242:2601:ac53::1; };
};
zone "20.172.in-addr.arpa" {
  type forward;
  forwarders { 172.20.129.1; fd42:4242:2601:ac53::1; };
};
zone "22.172.in-addr.arpa" {
  type forward;
  forwarders { 172.20.129.1; fd42:4242:2601:ac53::1; };
};
zone "23.172.in-addr.arpa" {
  type forward;
  forwarders { 172.20.129.1; fd42:4242:2601:ac53::1; };
};
```

**Note**: With DNSSEC enabled, bind might refuse to accept query results from the dn42 zone: `validating dn42/SOA: got insecure response; parent indicates it should be secure`.

## dnsmasq

If you are running dnsmasq under openwrt, you just have to add 

```
config dnsmasq
        option boguspriv '0'
        option rebind_protection '1'
        list rebind_domain 'dn42'
        list server '/dn42/172.20.129.1'
        list server '/20.172.in-addr.arpa/172.20.129.1'
        list server '/21.172.in-addr.arpa/172.20.129.1'
        list server '/22.172.in-addr.arpa/172.20.129.1'
        list server '/23.172.in-addr.arpa/172.20.129.1'
        list server '/d.f.ip6.arpa/fd42:4242:2601:ac53::1'

```

to `/etc/config/dhcp` and run `/etc/init.d/dnsmasq restart`. After that you are able to resolve `.dn42` 
with the anycast DNS-Server, while your normal requests go to your standard DNS-resolver.

Attention: If you go with the default config you'll have to disable "boguspriv" in the first dnsmasq config section.

For normal dnsmasq use

```
server=/dn42/172.20.129.1
server=/20.172.in-addr.arpa/172.20.129.1
server=/21.172.in-addr.arpa/172.20.129.1
server=/22.172.in-addr.arpa/172.20.129.1
server=/23.172.in-addr.arpa/172.20.129.1
server=/d.f.ip6.arpa/fd42:4242:2601:ac53::1
```
in `dnsmasq.conf`.

## PowerDNS recursor
Add this to /etc/powerdns/recursor.conf (at least in Debian and CentOS), the **forward-zone-recurse** is _**one line**_.

```
dont-query=127.0.0.0/8, 10.0.0.0/8, 192.168.0.0/16, ::1/128, fe80::/10
forward-zones-recurse=dn42=172.20.129.1,hack=172.20.129.1,ffhh=172.20.129.1,ffac=172.20.129.1,020=172.20.129.1,adm=172.20.129.1,ffa=172.20.129.1,ffhb=172.20.129.1,ffc=172.20.129.1,ffda=172.20.129.1,ffdh=172.20.129.1,ff3l=172.20.129.1,fffl=172.20.129.1,ffffm=172.20.129.1,fffr=172.20.129.1,fffd=172.20.129.1,ffgl=172.20.129.1,fflln=172.20.129.1,ffbcd=172.20.129.1,ffbgl=172.20.129.1,ffgoe=172.20.129.1,ffgt=172.20.129.1,ffh=172.20.129.1,helgo=172.20.129.1,ffhef=172.20.129.1,ffj=172.20.129.1,ffka=172.20.129.1,ffki=172.20.129.1,ffhl=172.20.129.1,fflux=172.20.129.1,ffms=172.20.129.1,mueritz=172.20.129.1,ffnord=172.20.129.1,ffnw=172.20.129.1,ffoh=172.20.129.1,ffpb=172.20.129.1,ffpi=172.20.129.1,ffrade=172.20.129.1,ffrgb=172.20.129.1,ffrg=172.20.129.1,rzl=172.20.129.1,ffsaar=172.20.129.1,fftr=172.20.129.1,fftdf=172.20.129.1,ffwk=172.20.129.1,ffgro=172.20.129.1,ffwk=172.20.129.1,ffwp=172.20.129.1,ffw=172.20.129.1,20.172.in-addr.arpa=172.20.129.1,22.172.in-addr.arpa=172.20.129.1,23.172.in-addr.arpa=172.20.129.1,31.172.in-addr.arpa=172.20.129.1,c.f.ip6.arpa=172.20.129.1
```

## MaraDNS
Put this in your mararc:

```
ipv4_alias["dn42_root"] = "172.20.129.1"
root_servers["dn42."] = "dn42_root"
root_servers["20.172.in-addr.arpa."] = "dn42_root"
root_servers["22.172.in-addr.arpa."] = "dn42_root"
root_servers["23.172.in-addr.arpa."] = "dn42_root"
```

## Unbound

Make sure DNSSEC is disabled (`auto-trust-anchor-file` is not set):

```
server:
      domain-insecure: "dn42"
      domain-insecure: "20.172.in-addr.arpa"
      domain-insecure: "21.172.in-addr.arpa"
      domain-insecure: "22.172.in-addr.arpa"
      domain-insecure: "23.172.in-addr.arpa"
      domain-insecure: "d.f.ip6.arpa"
      local-zone: "20.172.in-addr.arpa." nodefault
      local-zone: "21.172.in-addr.arpa." nodefault
      local-zone: "22.172.in-addr.arpa." nodefault
      local-zone: "23.172.in-addr.arpa." nodefault
      local-zone: "d.f.ip6.arpa." nodefault

forward-zone: 
      name: "dn42"
      forward-addr: fd42:4242:2601:ac53::1
      forward-addr: 172.20.129.1

forward-zone: 
      name: "20.172.in-addr.arpa"
      forward-addr: fd42:4242:2601:ac53::1
      forward-addr: 172.20.129.1

forward-zone: 
      name: "21.172.in-addr.arpa"
      forward-addr: fd42:4242:2601:ac53::1
      forward-addr: 172.20.129.1

forward-zone: 
      name: "22.172.in-addr.arpa"
      forward-addr: fd42:4242:2601:ac53::1
      forward-addr: 172.20.129.1

forward-zone: 
      name: "23.172.in-addr.arpa"
      forward-addr: fd42:4242:2601:ac53::1
      forward-addr: 172.20.129.1

forward-zone:
      name: "d.f.ip6.arpa"
      forward-addr: fd42:4242:2601:ac53::1
      forward-addr: 172.20.129.1
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
              172.20.129.1;
	      fd42:4242:2601:ac53::1;
           }
        }
        default-domain 20.172.in-addr.arpa {
               forwarders {
                  172.20.129.1;
		  fd42:4242:2601:ac53::1;
               }
            }
        default-domain 22.172.in-addr.arpa {
               forwarders {
                  172.20.129.1;
		  fd42:4242:2601:ac53::1;
               }
            }
            default-domain 23.172.in-addr.arpa {
               forwarders {
                  172.20.129.1;
                  fd42:4242:2601:ac53::1;
               }
            }
         }
      }
   }
}
```

## MS DNS
Add a "Conditional Forward" (de: "Bedingte Weiterleitung") for each of "dn42", "20.172.in-addr.arpa", "22.172.in-addr.arpa", "23.172.in-addr.arpa" using 172.20.129.1 as forwarder. Ignore the error message that the server is not authoritative.
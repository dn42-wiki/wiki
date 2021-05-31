# Forwarder setup

Configuration of common resolver softwares to forward DNS queries for `.dn42` (and reverse DNS) IPv4 and IPv6 anycast services.

You can use any *.recursive-servers.dn42 (where * is a letter) for resolving .dn42 domains. The current list is available at the [DN42 registry](https://git.dn42.dev/dn42/registry/src/master/data/dns/recursive-servers.dn42) or through querying SRV records of recursive-servers.dn42:

````sh
drill -D SRV _dns._udp.recursive-servers.dn42. @172.20.0.53
````

Two independent anycast services are also provided:

| Name | IPv4 | IPv6 |
|---|---|---|
| a0.recursive-servers.dn42 | 172.20.0.53 | fd42:d42:d42:54::1 |
| a3.recursive-servers.dn42 | 172.23.0.53 | fd42:d42:d42:53::1 |

All the examples here list 172.20.0.53/fd42:d42:d42:54::1, but users are encouraged to configure
multiple services from *.recursive-servers.dn42 for redundancy. 

## Note on ICVPN Zones

DN42 is [interconnected](/internal/Interconnections) with the Inter City VPN or in short "ICVPN". The registry of the ICVPN includes all the DNS information such as the Top level domains (TLDs) used inside ICVPN and the reverse DNS for the IP ranges of the ICVPN. Additionally, it includes the TLDs of some other networks that are interconnected with dn42 and share some of the IP space of ICVPN. The ICVPN [repository](https://github.com/freifunk/icvpn-meta) includes a handy script to automatically generate all the required zones.

## BIND

If you already run a local DNS server, you can tell it to query the dn42 anycast servers for the relevant domains
by adding the following to /etc/bind/named.conf.local

````
zone "dn42" {
  type forward;
  forwarders { 172.20.0.53; fd42:d42:d42:54::1; };
};
zone "20.172.in-addr.arpa" {
  type forward;
  forwarders { 172.20.0.53; fd42:d42:d42:54::1; };
};
zone "21.172.in-addr.arpa" {
  type forward;
  forwarders { 172.20.0.53; fd42:d42:d42:54::1; };
};
zone "22.172.in-addr.arpa" {
  type forward;
  forwarders { 172.20.0.53; fd42:d42:d42:54::1; };
};
zone "23.172.in-addr.arpa" {
  type forward;
  forwarders { 172.20.0.53; fd42:d42:d42:54::1; };
};
zone "10.in-addr.arpa" {
  type forward;
  forwarders { 172.20.0.53; fd42:d42:d42:54::1; };
};
zone "d.f.ip6.arpa" {
  type forward;
  forwarders { 172.20.0.53; fd42:d42:d42:54::1; };
};

# for reverse dns to work the following option must be set:
options {
  # [...]

  # disable the integrated handling of RFC1918 and non-assigned IPv6 space reverse dns
  empty-zones-enable no;

  # [...]
};
````

**Note**: With DNSSEC enabled, bind might refuse to accept query results from the dn42 zone: `validating dn42/SOA: got insecure response; parent indicates it should be secure`.

To disable DNSSEC validation only for certain TLDs include the following in the options section:
````
options {
  # [...]
  
  validate-except {
    "dn42";
    "20.172.in-addr.arpa";
    "21.172.in-addr.arpa";
    "22.172.in-addr.arpa";
    "23.172.in-addr.arpa";
    "10.in-addr.arpa";
    "d.f.ip6.arpa";
  };

  # [...]
};
````

## dnsmasq

If you are running dnsmasq under openwrt, you just have to add 

````
config dnsmasq
        option boguspriv '0'
        option rebind_protection '1'
        list rebind_domain 'dn42'
        list server '/dn42/172.20.0.53'
        list server '/20.172.in-addr.arpa/172.20.0.53'
        list server '/21.172.in-addr.arpa/172.20.0.53'
        list server '/22.172.in-addr.arpa/172.20.0.53'
        list server '/23.172.in-addr.arpa/172.20.0.53'
        list server '/10.in-addr.arpa/172.20.0.53'
        list server '/d.f.ip6.arpa/fd42:d42:d42:54::1'

````

to `/etc/config/dhcp` and run `/etc/init.d/dnsmasq restart`. After that you are able to resolve `.dn42` 
with the anycast DNS-Server, while your normal requests go to your standard DNS-resolver.

Attention: If you go with the default config you'll have to disable "boguspriv" in the first dnsmasq config section.

For normal dnsmasq use

````
server=/dn42/172.20.0.53
server=/20.172.in-addr.arpa/172.20.0.53
server=/21.172.in-addr.arpa/172.20.0.53
server=/22.172.in-addr.arpa/172.20.0.53
server=/23.172.in-addr.arpa/172.20.0.53
server=/10.in-addr.arpa/172.20.0.53
server=/d.f.ip6.arpa/fd42:d42:d42:54::1
````
in `dnsmasq.conf`.

## PowerDNS recursor
Add this to /etc/powerdns/recursor.conf (at least in Debian and CentOS), the **forward-zone-recurse** is _**one line**_.

````
dont-query=127.0.0.0/8, 192.168.0.0/16, ::1/128, fe80::/10
forward-zones-recurse=dn42=172.20.0.53,hack=172.20.0.53,ffhh=172.20.0.53,ffac=172.20.0.53,020=172.20.0.53,adm=172.20.0.53,ffa=172.20.0.53,ffhb=172.20.0.53,ffc=172.20.0.53,ffda=172.20.0.53,ffdh=172.20.0.53,ff3l=172.20.0.53,fffl=172.20.0.53,ffffm=172.20.0.53,fffr=172.20.0.53,fffd=172.20.0.53,ffgl=172.20.0.53,fflln=172.20.0.53,ffbcd=172.20.0.53,ffbgl=172.20.0.53,ffgoe=172.20.0.53,ffgt=172.20.0.53,ffh=172.20.0.53,helgo=172.20.0.53,ffhef=172.20.0.53,ffj=172.20.0.53,ffka=172.20.0.53,ffki=172.20.0.53,ffhl=172.20.0.53,fflux=172.20.0.53,ffms=172.20.0.53,mueritz=172.20.0.53,ffnord=172.20.0.53,ffnw=172.20.0.53,ffoh=172.20.0.53,ffpb=172.20.0.53,ffpi=172.20.0.53,ffrade=172.20.0.53,ffrgb=172.20.0.53,ffrg=172.20.0.53,rzl=172.20.0.53,ffsaar=172.20.0.53,fftr=172.20.0.53,fftdf=172.20.0.53,ffwk=172.20.0.53,ffgro=172.20.0.53,ffwk=172.20.0.53,ffwp=172.20.0.53,ffw=172.20.0.53,20.172.in-addr.arpa=172.20.0.53,21.172.in-addr.arpa=172.20.0.53,22.172.in-addr.arpa=172.20.0.53,23.172.in-addr.arpa=172.20.0.53,31.172.in-addr.arpa=172.20.0.53,10.in-addr.arpa=172.20.0.53,c.f.ip6.arpa=172.20.0.53
````

## MaraDNS
Put this in your mararc:

````
ipv4_alias["dn42_root"] = "172.20.0.53"
root_servers["dn42."] = "dn42_root"
root_servers["20.172.in-addr.arpa."] = "dn42_root"
root_servers["21.172.in-addr.arpa."] = "dn42_root"
root_servers["22.172.in-addr.arpa."] = "dn42_root"
root_servers["23.172.in-addr.arpa."] = "dn42_root"
root_servers["10.in-addr.arpa."] = "dn42_root"
````

## Unbound

Make sure to disable `auto-trust-anchor-file` and manually configure `trust-anchor-file` to 
point to a file with DNSKEY records for dn42.

````
server:
      local-zone: "20.172.in-addr.arpa." nodefault
      local-zone: "21.172.in-addr.arpa." nodefault
      local-zone: "22.172.in-addr.arpa." nodefault
      local-zone: "23.172.in-addr.arpa." nodefault
      local-zone: "10.in-addr.arpa." nodefault
      local-zone: "d.f.ip6.arpa." nodefault

forward-zone: 
      name: "dn42"
      forward-addr: fd42:d42:d42:54::1
      forward-addr: 172.20.0.53

forward-zone: 
      name: "20.172.in-addr.arpa"
      forward-addr: fd42:d42:d42:54::1
      forward-addr: 172.20.0.53

forward-zone: 
      name: "21.172.in-addr.arpa"
      forward-addr: fd42:d42:d42:54::1
      forward-addr: 172.20.0.53

forward-zone: 
      name: "22.172.in-addr.arpa"
      forward-addr: fd42:d42:d42:54::1
      forward-addr: 172.20.0.53

forward-zone: 
      name: "23.172.in-addr.arpa"
      forward-addr: fd42:d42:d42:54::1
      forward-addr: 172.20.0.53

forward-zone: 
      name: "10.in-addr.arpa"
      forward-addr: fd42:d42:d42:54::1
      forward-addr: 172.20.0.53

forward-zone:
      name: "d.f.ip6.arpa"
      forward-addr: fd42:d42:d42:54::1
      forward-addr: 172.20.0.53
````

## JunOS (SRX 12.1X46)
Should also work in 12.1X44 and 12.1X45. After making the changes below you may need to run:
````
restart named-service
````
Config (vlan.0 is presumed to be your LAN/Trust interface)
````
system {
   services {
      dns {
         dns-proxy {
            interface {
               vlan.0;
            }
        default-domain dn42 {
           forwarders {
              172.20.0.53;
	      fd42:d42:d42:54::1;
           }
        }
        default-domain 20.172.in-addr.arpa {
               forwarders {
                  172.20.0.53;
		  fd42:d42:d42:54::1;
               }
            }
        default-domain 21.172.in-addr.arpa {
               forwarders {
                  172.20.0.53;
		  fd42:d42:d42:54::1;
               }
            }
        default-domain 22.172.in-addr.arpa {
               forwarders {
                  172.20.0.53;
		  fd42:d42:d42:54::1;
               }
            }
            default-domain 23.172.in-addr.arpa {
               forwarders {
                  172.20.0.53;
                  fd42:d42:d42:54::1;
               }
            }
            default-domain 10.in-addr.arpa {
               forwarders {
                  172.20.0.53;
                  fd42:d42:d42:54::1;
               }
            }
         }
      }
   }
}
````

## MS DNS
Add a "Conditional Forward" (de: "Bedingte Weiterleitung") for each of "dn42", "20.172.in-addr.arpa", "21.172.in-addr.arpa", "22.172.in-addr.arpa", "23.172.in-addr.arpa", "10.in-addr.arpa" using 172.20.0.53 as forwarder. Ignore the error message that the server is not authoritative.
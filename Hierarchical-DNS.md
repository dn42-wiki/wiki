DNS in the global internet is designed as a tree starting from "." and traveling outward in layers. Currently in DN42 dns is flat. This leads to issues when trying to debug problems and makes it difficult to delegate to subnets smaller than /24. Another problem that arises is having the root dns setup as an anycast. If one of the anycast roots is having problems it creates inconsistent errors for some users. This has led to the problem of when a user has a poorly configured anycast available to create their own root anycast. 

The purpose of this project is to create a system of high quality dns roots. With them in place, an anycast resolver would only need to be a simple caching server that uses the roots to query. 

## Hierarchy in DN42

 - . (dot)
   - arpa
     - in-addr
       - 172
           - 20 (Reserved for future use)
           - 21 (Reserved for future use)
           - 22
           - 23
   - dn42
     - {{dn42 domain names}}
   - {{Future Top Level Domains?}}
   - {{hack, ano, bit or other non-iana roots?}}

Note: With this system it could be possible to merge the IANA root file.. but it would be beyond the scope of this project. 

## Servers

For all of these servers they have a specific IP assigned, are not anycasted, only respond to their authoritative zones, and do not allow recursion. 

**{{name}}.root-servers.dn42** - This server is authoritative for "." (dot) only.

**{{name}}.dn42-servers.dn42** - This server is authoritative for "dn42" only. This would be where the records for all forward dns nameservers would be. Similar to our current root setup.

**{{name}}.arpa-servers.arpa** - This server is authoritative for "arpa", "in-addr", and each of the 172 zones for dn42 ip space. For non dn42 ip space NS records to the respective darknet would need to be registered. 

**{{name}}.zone-servers.arpa** - This server is authoritative for any RFC 2317 delegations. When generating for the rdns zones if nameservers are registered for a /25-28 inetnum the NS should point here to allow for proper delegation.

**{{name}}.zone-servers.dn42** - This server holds the zone files for the various *-servers.* 

## Example DNS setup

A prototype system of root dns has been setup in 172.22.141.0/28 to demonstrate.

```
$ dig +nodnssec +trace @172.22.141.1 xuu.zone-servers.dn42

; <<>> DiG 9.9.5-3ubuntu0.2-Ubuntu <<>> +trace @172.22.141.1 xuu.zone-servers.dn42 +nodnssec
; (1 server found)
;; global options: +cmd
.                       300     IN      NS      souris.root-servers.dn42.
;; Received 65 bytes from 172.22.141.1#53(172.22.141.1) in 2 ms

dn42.                   300     IN      NS      souris.dn42-servers.dn42.
;; Received 128 bytes from 172.22.141.1#53(souris.root-servers.dn42) in 0 ms

zone-servers.dn42.      300     IN      NS      souris.zone-servers.dn42.
;; Received 115 bytes from fdea:a15a:77b9:4444::2#53(souris.dn42-servers.dn42) in 0 ms

xuu.zone-servers.dn42.  300     IN      A       172.22.141.180
;; Received 66 bytes from fdea:a15a:77b9:4444::3#53(souris.zone-servers.dn42) in 0 ms

```


## Example Configs

```
::::::::::::::
db.dot
::::::::::::::
$TTL 5m
.     IN SOA  souris.root-servers.dn42. xuu.dn42.us.  ( 2015050100 5m 15m 1w 5m )
.     IN NS   souris.root-servers.dn42.

dn42. IN NS   souris.dn42-servers.dn42.
arpa. IN NS   souris.arpa-servers.arpa.

souris.root-servers.dn42. IN A 172.22.141.1
souris.root-servers.dn42. IN AAAA fdea:a15a:77b9:4444::1

souris.dn42-servers.dn42. IN A 172.22.141.2
souris.dn42-servers.dn42. IN AAAA fdea:a15a:77b9:4444::2



::::::::::::::
db.arpa
::::::::::::::
$TTL 300
arpa.     IN SOA  souris.arpa-servers.arpa. xuu.dn42.us.  ( 2015050100 5m 15m 1w 5m )
          IN NS   souris.arpa-servers.arpa. 

arpa-servers.arpa.   IN NS   souris.zone-servers.dn42.
20.172.in-addr.arpa. IN NS   souris.root.dn42.
22.172.in-addr.arpa. IN NS   souris.root.dn42.
23.172.in-addr.arpa. IN NS   souris.root.dn42.

souris.arpa-servers.arpa. IN A    172.22.141.4
souris.arpa-servers.arpa. IN AAAA fdea:a15a:77b9:4444::4

::::::::::::::
db.dn42
::::::::::::::
$TTL 300
dn42.     IN SOA  souris.dn42-servers.dn42. xuu.dn42.us.  ( 2015050700 5m 15m 1w 5m )
dn42.     IN NS   souris.dn42-servers.dn42. 

xuu.dn42.          IN NS   souris.root.dn42.
root.dn42.         IN NS   souris.root.dn42.

root-servers.dn42. IN NS   souris.zone-servers.dn42.
dn42-servers.dn42. IN NS   souris.zone-servers.dn42.
zone-servers.dn42. IN NS   souris.zone-servers.dn42.

souris.root.dn42. IN A 172.22.141.180
souris.root.dn42. IN AAAA fdea:a151:77b9:53::1

souris.dn42-servers.dn42. IN A    172.22.141.2
souris.dn42-servers.dn42. IN AAAA fdea:a15a:77b9:4444::2

souris.zone-servers.dn42. IN A    172.22.141.3
souris.zone-servers.dn42. IN AAAA fdea:a15a:77b9:4444::3

::::::::::::::
db.dn42-servers.dn42
::::::::::::::
$TTL 300
dn42-servers.dn42.     IN SOA  souris.zone-servers.dn42. xuu.dn42.us.  ( 2015050100 5m 15m 1w 5m )
dn42-servers.dn42.     IN NS   souris.zone-servers.dn42. 

dn42-servers.dn42.     IN A    172.22.141.2
dn42-servers.dn42.     IN AAAA fdea:a15a:77b9:4444::2

souris.dn42-servers.dn42. IN A    172.22.141.2
souris.dn42-servers.dn42. IN AAAA fdea:a15a:77b9:4444::2


::::::::::::::
db.root-servers.dn42
::::::::::::::
$TTL 5m
root-servers.dn42.  IN SOA  souris.zone-servers.dn42. xuu.dn42.us.  ( 2015050100 5m 15m 1w 5m )
root-servers.dn42.  IN NS   souris.zone-servers.dn42.

root-servers.dn42.     IN A    172.22.141.1
root-servers.dn42.     IN AAAA fdea:a15a:77b9:4444::1

souris IN A 172.22.141.1
souris IN AAAA fdea:a15a:77b9:4444::1


::::::::::::::
db.zone-servers.dn42
::::::::::::::
$TTL 5m
zone-servers.dn42. IN SOA  souris.zone-servers.dn42. xuu.dn42.us.  ( 2015050100 5m 15m 1w 5m )
                   IN NS   souris.zone-servers.dn42. 

zone-servers.dn42.     IN A    172.22.141.3
zone-servers.dn42.     IN AAAA fdea:a15a:77b9:4444::3

souris IN A 172.22.141.3
souris IN AAAA fdea:a15a:77b9:4444::3
xuu    IN A 172.22.141.180
```
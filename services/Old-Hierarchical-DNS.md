This information is now **deprecated**. Please check [New DNS](/services/New-DNS) for the current architecture.

***

DNS in the global internet is designed as a tree starting from "." and traveling outward in layers. Currently in DN42 dns is flat. This leads to issues when trying to debug problems and makes it difficult to delegate to subnets smaller than /24. Another problem that arises is having the root dns setup as an anycast. If one of the anycast roots is having problems it creates inconsistent errors for some users. This has led to the problem of when a user has a poorly configured anycast available to create their own root anycast. 

The purpose of this project is to create a system of high quality dns roots. With them in place, an anycast resolver would only need to be a simple caching resolver that uses the roots to query. 

## Hierarchy in DN42

 - . (dot)
   - arpa
     - in-addr
       - 172
           - 20
           - 22
           - 23
           - 31
   - dn42
     - <dn42 domain names>
   - hack
   - ffhh
   - <Future Top Level Domains?>
   - <ano, bit or other organisation TLDs?>
   - <ICANN TLDs>

Note: With this system it could be possible to merge the IANA root file.. but it would be beyond the scope of this project. 

## Servers

For all of these servers they have a specific IP assigned, only respond to their authoritative zones, and do not allow recursion. 

**<name>.root-servers.dn42** - This server is authoritative for "." (root zone). Is authorative for ICANN root as well. "172.in-addr.arpa" is delegated to ICANN, except rfc1918 zones which are delegated to dn42. The rest of rfc1918 as well as rfc4193 address space is delegated to dn42. 

**<name>.zone-servers.dn42** - This server is authoritative for "dn42", "hack", .. This would be where the records for all forward dns nameservers would be. Similar to our current root setup.

**<name>.in-addr-servers.arpa** - This server is authoritative for "arpa", "in-addr", and each of the 172 zones for dn42 ip space. For non dn42 ip space NS records to the respective darknet would need to be registered. 

**<name>.dn42-servers.arpa** - This server is authoritative for RFC 2317 delegations. For any inetnum object smaller than /24 and whos parent has no nameserver records, a C class parent zone is created (all its subnetworks are delegated to appropriate nameservers with CNAME)

Real-time server monitor is available at <http://nixnodes.net/dn42/dnsview>

## Setup

Contact one of the root-servers.dn42 operators if you wish to set up a root/zone/dn42 server. 

You may want to set up a resolver, see link below or use 172.23.0.53 directly.

Techical information available [here](https://nixnodes.net/wiki/n/DN42_DNS)

After frequent issues with the [[Old Hierarchical DNS]] system in early 2018, work has started to build a new and more reliable DNS system. The main goals are:
* Reliability and Consistency to avoid debugging very obscure issues that are also hard to reproduce.
* Low maintenance burden on operators.
* Proper DNSSEC support for everything.

# End Users
It is **strongly recommended** to run your own resolver for security and privacy reasons. Setting it up and maintaining it should be easy, see [Running your own instances](#running-your-own-instances).

If running your own resolver is not possible or undesirable, you can choose one or more instances from [dns/recursive-servers.dn42 in the registry](https://git.dn42.us/dn42/registry/src/master/data/dns/recursive-servers.dn42). Please make sure you fully understand the consequences and fully trust these operators.

You can also use the globally anycasted a.recursive-servers.dn42 but you won't have any control over which instance you get. This is a **very bad idea** from a security standpoint.

# Instances
The new DNS system has two different components:
* *.recursive-servers.dn42 and local resolvers responsible for handling queries from clients, validating DNSSEC and directing the queries at clearnet/dn42/ICVPN.
* *.delegation-servers.dn42 and *.master.delegation-servers.dn42 are a normal master-slave setup for providing the few official infrastructural zones.

## *.recursive-servers.dn42
These are simple resolvers capable of resolving dn42 domains. Every operator gets a single letter name pointing to addresses assigned from his own address space and is strongly encouraged to use anycasting across multiple nodes to improve reliability. There is also the global anycast a.recursive-servers.dn42 which includes some/all other instances. Whether an *.recursive-servers.dn42 can resolve clearnet queries or not is decided by its operator but all a.recursive-servers.dn42 instances MUST resolve clearnet queries correctly.

## *.delegation-servers.dn42
These are simple authoritative servers for the dn42 zone, rDNS and a few DNS infrastruture zones. Every operator gets a single letter name pointing to addresses assigned from his own address space and is strongly encouraged to use anycasting across multiple nodes to improve reliability. There is no anycast instance because that would make debugging much harder and *.recursive-servers.dn42 instances should do loadbalancing/failover across all instances listed in the registry.

## *.master.delegation-servers.dn42
These instances do not serve any clients. They poll the registry regularly and rebuild and resign (DNSSEC) the zones as needed. If any zone changes, all *.delegation-servers.dn42 instances are notified ([RFC1996](https://tools.ietf.org/html/rfc1996)) which then load the new zone data over AXFR ([RFC5936](https://tools.ietf.org/html/rfc5936)). The pool of masters is intentionally kept very small because of its much higher coordination needs and also the lacking support of a multi-master mode in many authoritative server implementations. The masters are only reachable over dedicated IPv6 assignments which are set up in a way that any master operator can hijack the address of a problematic master without having to wait for its operator to fix something.

# Running your own instances
* If you want to run your own instances, make sure you are subscribed to the [[mailinglist|contact]]. It is also strongly recommended to join #dn42-dns@hackint. All changes are announced to the mailinglist but IRC makes debugging sessions much easier.
* Choose the implementation(s) you want to use. It should support at least AXFR+NOTIFY (*.delegation-servers.dn42) or DNSSEC (*.recursive-servers.dn42).
* Check if [[TODO|TODO]] already has configuration snippets for your implementation.
  * If yes, download it from there and include it in the main configuration.
  * If not, then join us in #dn42-dns@hackint so we can add it together.
* Verify that everything works:
  * For *.delegation-servers.dn42: Do an AXFR against all zones and compare with the result of an existing instance. The result should be identical.
  * For *.recursive-servers.dn42: Query clearnet, dn42 and ICVPN domains including rDNS. Make sure that both signed and unsigned domains work properly.
* (Optional) Choose your single letter name and ask in #dn42-dns@hackint to get it added to the registry. Once added to the list, you must implement changes announced to the mailinglist within a week (faster is obviously better) or you might get removed again. We try to keep maintenance work as low as possible but we can't do it without the cooperation of all operators!

# [Monitoring](https://grafana.burble.com/d/E4iCaHoWk/dn42-dns-status?orgId=1&refresh=1m)
burble is providing monitoring for the new DNS system. It does simple checks on all instances every minute and also logs all changes into #dn42-dns@hackint.

# DNSSEC
There are currently two KSKs managed by JRB0001-MNT and YAMAKAJA-MNT. They are used once per quarter to sign the DNSKEY RRset. Each master operator has one ZSK which is used to sign the zones (except for the DNSKEY RRset). This setup leads to bigger responses but allows each KSK holder to solve emergencies independently. The signatures of the DNSKEY RRset are valid until the end of the first month of the next quarter to give enough time for coordinating the next siging. All other signatures are valid for 3 days and replaced at least once per day.

The set of valid KSKs can be found in the registry.
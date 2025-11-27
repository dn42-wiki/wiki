# Issue: PIM: Multicast vs. Unicast Routes

## Symptom

With pim6sd you might have stumbled over the following message in your logs: `For src [...], iif is [...], next hop router is [...]: NOT A PIM ROUTER`. And your multicast routing somehow does not work even though according to your pim6stat output it looks like the correct multicast router was detected via PIM. Or it might have worked in the past and suddenly stopped working.

This means that somehow the neighboring router which pim6sd has chosen is another router not capable of multicast routing via PIM. But how can that be, why did pim6sd chose that neighbor in the first place then?

## Background

One simple cause could be firewall rules filtering some PIM messages. But there can be another issue, which this page will dive into.

PIM stands for **Protocol Independent** Multicast. Which confusingly does not mean that PIM would be a fully independent protocol... It is actually very dependent on the existence of preestablished unicast routes, so typically quite dependent on another unicast routing protocol. It is only independent of how those unicast routes were established exactly, which metric was used etc. Without unicast routes PIM does not work though. PIM then tries to establish its multicast routing trees via/over those unicast routes (and for PIM-SM in any-source multicast, ASM, mode might even use that to tunnel multicast packets to before a multicast routing tree could be established, to reduce delays or to avoid packet loss).

**This means that for a PIM-SM daemon any unicast route configured in the kernel towards any PIM-SM router in the network must point to a next hop capable of PIM-SM.** Otherwise the incapable next hop router will potentially be a black hole for multicast routing.

## Examples

### Example 1

Nodes R1, R2, R3 are all peering via BGP on dedicated interface to each other. And have established unicast routes to each other through BGP. For PIM-SM all three routers are running a PIM daemon, but somehow PIM was not configured properly on the dedicated interfaces between R1 and R3 (acc. interfaces not added to the PIM daemon, incorrect filter rules, missing kernel multicast flag on these interfaces, ...).

PIM-SM between R1 and R3 will be broken, even though in theory between R1 and R3 via R2 could work.

### Example 2

Same as example 1 but here multicast routing between R1 and R4 will be broken, as the selected unicast route between R1 and R4 is via R5, but R5 is not running any daemon capable of PIM-SM at all.

Here it is apparent that not only misconfiguration can lead to this unicast vs. multicast routes issue. But also potentially any router in the network which is only capable of a unicast routing protocol while not yet being capable of PIM.

## Example 3

In this example R1 and R2 are routers for the same AS. Furthermore they serve several other client devices downstream. Therefore both R1 and R2 announce the same IPv6 /64 subnet for these client devices via BGP. However R1 and R2 have disabled PIM on their downstream interface, they only use MLD downstream while PIM is only used to their upstream routers R3 and R4.

R3 wants to establish a multicast tree to R1 via PIM. The shortest/"best" path for R3 to R1 according to BGP is R3->R2(->R1). 

But because R2 does not talk PIM on the downstream interface it will not forward PIM messages to R1, therefore creating a black hole for multicast.

## Mitigations

### BGP multicast channels

BGP allows using the dedicated BGP channels "IPv4 multicast" and "IPv6 multicast". This does *not* mean that with these channels BGP would distribute multicast routes. But it allows establishing an alternative unicast topology that is supposed to be dedicated for multicast protocols like PIM. And more notably distinct to the BGP topology used for unicast payload packets.

This should especially make it easier to avoid the second example. However it does not help in the situation where for instance bird is used for BGP and pim6sd for PIM and the latter crashed at some point in the future. Then even though bird is using the "ipv{4,6} multicast" channel it will not be aware that PIM ceased to work.

### (Only?) unicast host routes for multicast routers (IPv4: /32, IPv6: /128)

On PIM capable routers announce /32 and /128 unicast host routes (in the dedicated BGP IPv4/IPv6 multicast channels?), the unicast address(es) of this specific multicast router. This would especially avoid the issue in example 3, as a /128 for R1 received on R3 via R4 would be more specific and prefered over a /64 received on R3 from R2. Also only announcing and accepting host routes on the BGP IPv4/v6 multicast channels might be sufficient and safer?

ToDo: Verify that for PIM-SM to work properly that unicast routes only to multicast routers are sufficient. That unicast routes to any non-PIM downstream clients are not needed. For PIM-SM in ASM mode this should work as we only need unicast connectivity to PIM-SM capable Rendezvous-Points? Would this also work for SSM?

Typically there shouldn't be that many multicast routers for a subnet? So announcing only host routers shouldn't increase overhead that much on a BGP IPv4/v6 multicast channel? But might be a bit "unconventional" to announce such small subnets / host routes via BGP?

### Watchdog?

Write some watchdog script/daemon which enables/disables/filters BGP exports(/imports?) if it detects that a/no PIM session was established to that neighbor?

Question: Would there be a chicken & egg situation, or at least (a) small timeframe(s) for potential packetloss? We need unicast routes for PIM a priori, but we want to retract unicast routes if PIM is not established to avoid black holing? Maybe a watchdog would actually need to:

1) Always run PIM on all interfaces to (potential) routers
2) Initially only accept unicast routes from direct neighbors via BGP (on the IPv4/v6 multicast channel), no unicast routes from multi-hop routers yet
3) If both BGP and PIM are established to a neighbor: Then start accepting unicast routes via BGP via multi-hop
4) If PIM to a neighbor times out, retract/filter multi-hop unicast routes via BGP from this neighbor


### ROA?

Use a dedicated ROA filter database for multicast, (manually? automaticaly?) delete entries of misbehaving nodes?
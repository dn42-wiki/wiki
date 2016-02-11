_Work in progress_

## Introduction

DN42 is a somewhat unique undertaking, and a great way to learn about networking and routing techs.
If you feel like spicing the challenge up a bit, why not get familiar with IPv6 at the same time ?

There's nothing too different from IPv4, except one major point: it's a separate network space, and therefore you might not be able to reach the IPv4-part of DN42. That is, without NAT64, proxies, etc.. (read more below!) or if you're running dual-stack, both IPv4 and IPv6 at the same time.

## Getting Started

If you're already running IPv4 on DN42, here's how to get started:
 * Register an inet6num block on the registry. A /56-size prefix is good, but with the space available in IPv6 space, you can probably go for /48
 * Setup your interfaces to use them
 * Negociate with your v6-capable peers to setup peering sessions to allow your v6 prefixes
 * ???
 * Profit!

If not, you can follow the instructions on the [Getting Started](GettingStarted) page, as they'll mostly apply to IPv6 aswell.

## What can i do on DN42-v6 ?

A fair share of the services are available through IPv6. However some of the well known addresses might not work, so you'll have to find alternative services. For example, as of current, there's no IPv6-anycast for the wiki.dn42 address, so you want to use an IPv6-capable instance directly, such as [Mic92's Instance of the Wiki](as4242420092-de.wiki.dn42).

Quick list of some native-IPv6-capable services:
 * Some instances of this Wiki, but no anycast yet
 * Anycast [whois.dn42](whois.dn42)
 * DNS, including anycast
 * Media boards, including DN42-Chan
 * torrents.dn42

What doesn't work (yet?): 
 * Search engines, none seems to support IPv6 currently
 * Network Map (it doesn't show v6 AS-PATHs)
 * Pretty much everything from Freifunk and ChaosVPN
 * Any services hosted by Nixnodes or e-utp.dn42 

## Accessing legacy services from an IPv6-only stack
In order to access legacy IPv4 services from the IPv6 side of DN42, you're going to need some kind of service to jump from one to the other.
This can typically be done in two ways:
 * A dual-stack Proxy with Remote DNS
 * A dual-stack Router with NAT64, plus DNS64 services

It's important to note that since these services require IPv4 connectivity, you can't set them up yourself if you are running IPv6-only. You'll need to ask another DN42 participant to be your provider for these services, effectively using his node as a gateway/proxy.

### With a SOCKS5 proxy and Remote DNS
Just set it up like you would usually. Any IPv4 connection going through the proxy will be made to the proxy by IPv6, then from the proxy to target using IPv4.

### NAT64+DNS64
In order to maintain backward compatibility, a number of methods have been used to be able to reach IPv4 space from IPv6. NAT-PT was deprecated, so this mostly leaves us with NAT64.
NAT64 simply consists in embedding IPv4 addresses after an IPv6 prefix, and mapping the whole prefix to the whole of IPv4 space. Thanksfully, that's easily doable because we have 128-bit wide addresses in IPv6. As the name implies, we however have to perform Dynamic-NAT on the IPv4 side to squeeze all the incoming v6 space in v4 addresses (though it should be possible to run 1-1 mappings, but in that case, why not get a v4 address and NAT to it to begin with?)

Currently, NAT64 support in DN42 is non-existant, though there are ongoing experimentations with it. Technically, it is possible to announce a global anycast prefix for NAT64, allowing seamless IPv4 connectivity from any properly configured IPv6 host, or any using the DNS64 (which can also be setup on the anycast servers).

DNS64 itself simply allow to synthetizes AAAA records from the received usual A records. Because DNS runs at the transport level and does not care for Layer 3 triffles, this is a service that you can run on your Nameserver even without being Dual-Stack capable. (TODO: DNS64 Howto with BIND9)
As such, any address that can only resolve to IPv4 will now also resolve to an address corresponding to it through the NAT64 prefix.

## Routing to Internet and DN42
So now that you've got IPv6 setup for DN42, you'd like to start using it on the Internet aswell. Or maybe you already do. But how to use your services on both public Internet and DN42 ?

### With NPT
A first approach is to use NPT: Network Prefix Translation. Yes, this sounds a lot like NAT, but fear not: it does not have most of its problems as it is fully stateless. Initially, the purpose of NPT was to allow multi-homing without an ASN: how can you be reachable through several prefixes allocated by different ISPs ? The IPv6-way of doing it would be to assign multiple addresses from the multiple prefixes to all your nodes, but isn't that just too complicated ?

Enter NPT. Address your services using a reserved private block, and map that block to a public block upon routing to internet. 
For example, if you've been assigned the <PUBLIC-PREFIX>::/48 prefix, and want to be reachable on DN42 aswell, you can use only ULA addresses from DN42 internally (or your own!), then map them to outside prefixes. Note that they'll need to all use the same prefix size to maintain the one-to-one mapping, so you may have to subnet the public prefix.

In Linux's netfilter, this can be implemented through the use of the NETMAP target, for the example above:
`ip6tables -t nat -A POSTROUTING -d 2000::/3 -s <DN42-PREFIX>:<SUBNET>::/56 -j NETMAP --to <PUBLIC-PREFIX>:<SUBNET>::/56; # Map ULA to the public prefix for outgoing packets
ip6tables -t nat -A PREROUTING -s 2000::/3 -d <PUBLIC-PREFIX>:<SUBNET>::/56 -j NETMAP --to <DN42-PREFIX>:<SUBNET>::/56; # Map public prefix to ULA for incoming packets`


### With Multiple Prefixes

## More Info
This page is a work in progress. Please contact Fira if you feel like more information should be added here! Also see ASN 4242423218 for an example of IPv6-only AS on DN42.
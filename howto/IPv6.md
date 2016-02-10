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

## Accessing legacy services: NAT64 and DNS64

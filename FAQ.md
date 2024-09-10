## How do I connect to DN42?

We have a [page](/howto/Getting-Started) for that!


### What BGP daemon should I use?

This is really up to you: that's the magic of open protocols.

Many users run Bird or FRRouting in a VPS, but there is a variety of BGP implementations in use across the network and some networks even run hardware routers or experimental software. DN42 is a great place to try out and get familiar with your implementation of choice in a live network. 


### Can I connect to DN42 from home through my ISP?

Sure. As long as you have a 24/7 connection, can configure VPN tunnels and run BGP then you can connect to DN42. Do let your peers know if you have a dynamic IP address and you may also want to consider preventing transit traffic through your network if you have a relatively slow connection.

Many users do use a virtual private server (VPS) for connecting to DN42. A VPS is a relatively cheap option that  is always on and which (typically) provides a faster, more stable connection than your home. VPS have another advantage that if you mess something up, you won't break your home access to the internet. 


### Does the registry use monotone or git?

The DN42 registry used to be maintained in monotone, but was moved to git following resource and performance
issues. There may still be references back to monotone in some of the documentation, but the registry location is now:

[https://git.dn42.dev/dn42/registry](https://git.dn42.dev/dn42/registry)

### Can I use Windows to clone and update the registry ?

No. The registry includes IPv6 resources but NTFS does not support having a `:` in filenames.

A simple workaround is to use a non-Windows VM to do your changes, or use the [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/install-win10) when working with the registry. 

### Can I reuse my public AS number/IPv4/IPv6?

AS number: Yes, see AS206633/AS205532 for example

IP: Probably no. Globally routable IPs are not supported in DN42; whilst in theory they could be announced, there are filters in place used by almost all networks which prevent that, and some networks will filter traffic to the private DN42 range only to enhance security. It's literally impossible to convince the whole of dn42 to lift those filters.

If you'd like to explore dn42 without too many changes to your network, you could announce both IPv6 GUA and ULA addresses to your network. IPv6 has a specified source address selection process which means that a host will always use an ULA address to talk to ULA destinations and a GUA address to talk to GUA destination.


### What about IPv6 in dn42?

DN42 supports IPv6 and some users do run IPv6 only networks. DN42 uses the "Unique Local Address" range (ULA, *fd00::/8*) defined by [RFC 4193](https://tools.ietf.org/html/rfc4193).


### Why are you using ASN in the 76100-76199 range?

DN42 now uses ASNs in the 4242420000-4242429999 range, but the 76100-76199 range was used historically, and there are still a very small number of networks using legacy ASNs that have not migrated. 

Previously, DN42 has used a two other ASN ranges:

We used to assign ASN in the 64600-64855 range, where you would get ASN 64600+X if you had 172.22.X.0/24. Once the address range was extended to include additional blocks, this process was obsoleted. Another issue with the private ASN range 64512-65534 was that other projects are also using it (for instance, Freifunk, Anonet, etc), which can lead to conflicts. 

Prior to using ASNs in the new private ASN range 4200000000-4294967294 ([RFC6996](http://tools.ietf.org/html/rfc6996)) Some ASNs were allocated from the [ASN reserved block](http://www.iana.org/assignments/as-numbers/as-numbers.xhtml) in the 76100-76199 range. 

### Why can't my Docker containers connect to other DN42 hosts?

By default, Docker overlaps with the entire DN42 range and then some. (172.16.0.0/12 == 172.16.0.0 - 172.31.255.255)

In order to prevent this, you need to supply a different subnet range to the Docker daemon. This can be done by creating or updating `/etc/docker/daemon.json` with something along the following (this will use 192.168.128.0/18 == 192.168.128.0 - 192.168.128.0 - 192.168.191.255)
```json
{
  "default-address-pools" : [
    {
      "base" : "192.168.128.0/18",
      "size" : 24
    }
  ]
}
```
Note, I (@bri / AS4242422825) have only tested this with Docker version 23.0.0, build e92dd87. But it should work with any current version. I don't know how Swarm etc. networking works, this might need additional tweaking for other versions. (Referenced from <https://straz.to/2021-09-08-docker-address-pools/> and <https://docs.docker.com/network/bridge/> — I used this to get my `thelounge` container to connect to hackint.dn42.)

### Can I update the wiki?

Yes, the wiki can be edited when browsing to [wiki.dn42](https://wiki.dn42).

After you have registered and set up SSH key authentication,
you can also clone and push to the
[git repository](https://git.dn42.dev/wiki/wiki) directly.

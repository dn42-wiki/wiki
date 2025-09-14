# Introduction
Here be dragons. This section should cover the basics:
* IKEv1
* Three stages: Key distribution, IPSec setup, GRE setup
* In theory, BGPd can set up IPSec flows itself, but we're not using that here because that prevents you from using another BGP daemon such as bird
* Debugging

# Key generation and distribution
## Using pre-generated keys
OpenBSD generates keys suitable for IPSec usage during installation. The public key can be found in `/etc/isakmpd/local.pub`.

## Generating your own keys
If you don't want to use a pre-generated key, refer to [isakmpd(8)](http://man.openbsd.org/isakmpd.8#X.509_AUTHENTICATION).

## Distributing keys
Send your public key to your peer, preferably digitally signed. A signature can be created with `gpg -sb -a local.pub` and checked with `gpg --verify local.pub.asc`. Since the key is not private, it can be transmitted in the open and on a public forum, such as a Pastebin service.

Once your peer sent you their public key, put it under `/etc/isakmpd/pubkeys/fqdn`, `/etc/isakmpd/pubkeys/ipv4` or `/etc/isakmpd/pubkeys/ipv6`, depending on the ID type the peer is using. The key file should be named after the peer ID. For example, if your peer is `1.3.3.7`, you place their public key under `/etc/isakmpd/pubkeys/ipv4/1.3.3.7`.

If your peer's public key is not in PEM format, you can use [pubkey-converter](https://dn42.us/git/user/ryan/pubkey-converter.git/plain/pubkey-converter.pl) to convert between key formats.

# IPSec Setup
Change the value of the `isakmpd_flags` variable in [`/etc/rc.conf.local`](http://man.openbsd.org/rc.conf.local.8) to `"-K"`, or add the `"-K"` flag if you already have flags in there.

Next, add the right flow parameters to [`/etc/ipsec.conf`](http://man.openbsd.org/ipsec.conf.5). We are using the following parameters in this example:

* Encryption: AES-128
* Authentication hash: HMAC-SHA1
* DH Group: `modp1536`
* Phase 1 lifetime: 28800 seconds
* Phase 2 lifetime: 3600 seconds
* Peer address: 1.3.3.7
* Local address: 3.4.5.6

The configuration file should look like this:

```conf
mymachine = "3.4.5.6"
mypeer    = "1.3.3.7"
ike esp transport proto gre from $mymachine to $mypeer \
  main auth hmac-sha1 enc aes-128 group modp1536 lifetime 28800 \
  quick auth hmac-sha1 enc aes-128 group modp1536 lifetime 3600
```

Load the configuration file into isakmpd: `ipsecctl -f /etc/ipsec.conf`. Once the connection is established, the IPSec flows can be listed with `ipsecctl -sa`:

```
# ipsecctl -sa
FLOWS:
flow esp in proto gre from 1.3.3.7 to 3.4.5.6 peer 1.3.3.7 srcid 3.4.5.6/32 dstid 1.3.3.7/32 type use
flow esp out proto gre from 3.4.5.6 to 1.3.3.7 peer 1.3.3.7 srcid 3.4.5.6/32 dstid 1.3.3.7/32 type require

SAD:
esp transport from 1.3.3.7 to 3.4.5.6 spi 0xdeadbeef auth hmac-sha1 enc aes
esp transport from 3.4.5.6 to 1.3.3.7 spi 0xf00df00d auth hmac-sha1 enc aes
```

# GRE Setup
Next, we will set up the GRE device. The [gre(4)](http://man.openbsd.org/gre.4) device encapsulates IPv4 and IPv6 traffic, which allows you to speak both address families over one tunnel if you only have native connectivity for one address family. The addresses configured onto the GRE device should come from a private address range that is not used anywhere in DN42, or a registered transfer net. For IPv6, you should use either ULAs or Link-Local addresses. In this example, we assume you are using 10.20.30.0/31 as the IPv4 transfer "net" (it has only two addresses, so calling it a network is a bit of an overstatement) and Link-Local addresses for IPv6.

```sh
# ifconfig gre0 tunnel 3.4.5.6 1.3.3.7
# ifconfig gre0 inet 10.20.30.0 10.20.30.1 # reverse these on your peer's side
# ifconfig gre0 inet6 eui64
```

These settings should also be added to [`/etc/hostname.gre0`](http://man.openbsd.org/hostname.if.5), .i.e.

```conf
tunnel 3.4.5.6 1.3.3.7
inet 10.20.30.0 10.20.30.1
inet6 eui64
```

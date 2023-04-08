# IPv6 Multicast

The following guide illustrates how to set up an IPv6 multicast router using [PIM-SM](https://en.wikipedia.org/wiki/Protocol_Independent_Multicast#Sparse_mode) (Protocol Independent Multicast in Sparse Mode) with your own personal multicast prefix.

rfc8815 deprecated pim-sm, please take a look at the new multicast page about pim-ssm: </howto/multicast>

## Quickstart

* Install pim6sd from here: <https://github.com/troglobit/pim6sd/>
    ```sh
    cd /usr/src
    git clone https://github.com/troglobit/pim6sd.git
    cd pim6sd
    ./autogen.sh
    ./configure
    make
    ```
* Find a peer who is already connected to the dn42 multicast backbone
* Calculate your personal, embedded-RP multicast prefix matching your network prefix via [RFC3956](https://tools.ietf.org/html/rfc3956)
  * Example:
    * Pattern: `ff7e:<RIID><plen>:<prefix>::/96`
        * Prefix: `fd00:2001:db8::/48`
        * Prefix length: `48 == 0x30`
        * RIID: An arbitrary number between `0x1` and `0xf`, for instance `0x2`
    * Result:
      * Multicast prefix: `ff7e:230:fd00:2001:db8::/96`
      * RP address: ``fd00:2001:db8::<RIID>`` -> ``fd00:2001:db8::2``

* Create a dummy interface to hold your calculated unicast Rendezvous Point address. This one needs to be reachable from within dn42. Also set "multicast on" on this dummy interface. Example:

    ```conf
    # /etc/network/interfaces.d/pim6sd
    auto pim-router-id
    iface pim-router-id inet manual
            pre-up ip link add name $IFACE type dummy
            post-up ip link set multicast on dev $IFACE
            post-up ip -6 a a fd00:2001:db8::2/128 dev $IFACE
            post-down ip link del $IFACE
    ```

* Create the configuration file:

    ```sh
    # /etc/pim6sd.conf
    # disable all interfaces by default
    default_phyint_status disable;

    # enable the pim-router-id interface first to acquire the correct primary address
    phyint pim-router-id enable;

    # add multicast-capable peer interfaces below
    phyint dn42-peer1 enable;

    # configure rendezvous point for the personal multicast prefix
    cand_rp pim-router-id;
    group_prefix ff7e:230:fd00:2001:db8::/96;
    ```

    The `phyint` statement enables [PIM](https://tools.ietf.org/html/rfc7761) and [MLD](https://tools.ietf.org/html/rfc2710) on the target interface - by default all interfaces are in the disable state. Enable an interface if it is directed towards a multicast-capable peer or other multicast-capable routers in your autonomous system. Also enable it for downstream network segments with multicast listeners and senders, like for example your home (W)LAN segments.

    With `cand_rp` and `group_prefix` statements you can configure this router as a Rendezvous Point (RP) for your personal multicast group prefix. The address on the interface given as `cand_rp` will be used as the primary address for your RP, it therefore *must* be routable.

---

## Testing & Applications

### Creating a test network namespace

On your router:

```sh
allow-hotplug pim-ns0
iface pim-ns0 inet manual
    pre-up ip link add pim-ns0 type veth peer name pim-ns1
    post-up ip netns add pim-ns0
    post-up ip link set addr 02:11:22:00:00:02 netns pim-ns0 name pim-ns0 up dev pim-ns1
    post-up ip link set addr 02:11:22:00:00:01 up dev pim-ns0
    post-up ip -6 a a fdd5:69d5:c530:1::1/64 dev pim-ns0
    post-up ip netns exec pim-ns0 ip -6 a a fdd5:69d5:c530:1::2/64 dev pim-ns0
    post-up ip netns exec pim-ns0 ip -6 r a default via fdd5:69d5:c530:1::1
    post-down ip link del pim-ns0
    post-down ip netns del pim-ns0
```

You can now switch into this test network namespace via "ip netns exec /bin/bash". Inside this network namespace you can try:

### Creating a test multicast listener

```sh
$ socat -u UDP6-RECV:1234,reuseaddr,ipv6-join-group="[ff7e:230:fdd5:69d5:c530::123]:eth0" -
```

### Creating a test multicast sender

First select which interface should be the default one for your multicast traffic. Then send multicast packets via ICMPv6:

```sh
$ ip -6 route add ff7e:230:fdd5:69d5:c530::/96 dev eth0 table local
$ ping6 -t 16 ff7e:230:fdd5:69d5:c530::123
```

The "-t 16", a hop-limit of 16, is important here as **by default all multicast traffic is usually send with a hop-limit of just 1**.

---

## Advanced Configurations



### Nomenclature

#### Bootstrap Router (BSR)

Router that collects multicast group information from all RP in the network and advertises it across the network.

#### Rendezvous Point (RP)

Router where senders and receivers will meet for a certain multicast address. Senders must send their data to it, after which it will be forwarded to receivers. As soon as a receivers DR learns of the sender it will ask their router to forward data along a direct path between sender and receiver.

#### Designated Router (DR)

First-hop router that stand in for sender and receiver on their downstream networks. The senders DR sends their data towards the RP encapsulated in PIM Register packets. The receivers DR will send join and prune messages to the RP, managing the group subscription.

### RFC3306: "Unicast-Prefix-based IPv6 Multicast Addresses"

Before RFC3956 (embedded RP addresses) personal, network prefix based multicast prefixes were calculated via RFC3306. Example:

* Pattern: `ff3e:<plen>:<prefix>::/96`
  * Prefix: `fd00:2001:db8::/48`
    * Prefix length: `48 == 0x30`
  * Result: `ff3e:30:fd00:2001:db8::/96`

* Pros:
  * More flexible RP address selection
  * Allows filtering on the BSR

* Cons:
  * Needs a central BSR for coordination (or static RP configuration)
  * Allows filtering on the BSR

However you can usually just announce and use both RFC3306 and RFC3956 based multicast prefixes, if you want to. pim6sd allows adding multiple ``group_prefix`` entries.

### Address Management

#### Bootstrap Router

If you want to be participate as a bootstrap router candidate, please read up on how PIM works first. If you join with a bootstrap router candidate add it here below with contact information and join #dn42-multicast on HackInt:
* \<BSR-ADDR1> - foo@example.com, foo@HackInt
* \<BSR-ADDR2> - ...

#### Shared multicast addresses

Next to personal multicast prefixes generated by network prefix (RFC3306 or RFC3956) there can also be multicast addresses not owned by a specific AS. In general any one can just set up a multicast sender or listener for those. However to work, they need a reliable RP for coordination.

If you want to offer an RP candidate for a shared multicast address, please read up on how PIM works first. If you join with an RP candidate for a shared multicast address add it here below with contact information and join #dn42-multicast on HackInt:
* \<multicast-address1>/128:
  - \<RP-address1> - foo@example.com, foo@HackInt
  - \<RP-address2> - bar@example.com, bar@HackInt
* \<multicast-address2>/128:
  - ...

## Questions?

* Join: ``#dn42-multicast`` on ``HackInt``

---

ToDo:
* We have a solution for personal multicast prefixes tied to the network prefix of an AS owner. But what to do with multicast addresses that not only have listeners but also senders globally? We could have everyone add an additional "group_prefix ff00::/8" and then multicast router with the lowest address would win and become the central RP for all these addresses... not really scalable, robust or decentral though :-/. Should we use PIM-DM for some of these addresses instead (e.g. ones which generally have a low throughput, for instance Bittorrent Local Peer Discovery)? Or maybe those global addresses should be managed and configured as /128 and people who are interested in managing a specific, global multicast address will coordinate with each other?
* bootstrap router coordination; according to RFCs a bootstrap router can alter/filter the multicast prefixes it received from candidate RPs. Should a bootstrap router check and filter any multicast prefix that was generated from a network prefix which does not match the network prefix used by the PR?

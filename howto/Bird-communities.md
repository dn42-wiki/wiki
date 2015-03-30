Bird is a commonly used BGP daemon.  This page provides configuration and help for using BGP communities with Bird for dn42.

Communities can be used to prioritize traffic based on different flags, in DN42 we are using communities to prioritize based on latency, bandwidth and encryption. Please note that everyone should be using community 64511.

The community is applied to the route when it is imported and exported, therefore you need to change your bird configuration, in /etc/bird/peers4 if you followed the [Bird](/howto/Bird) guide. 

The calculations for finding the best route can be stored in a separate file, for example /etc/bird/community_filters.conf.

Below, you will see an example config for peers4 as well as the and the suggested improvement by tombii (prefers low latency) to original filter implementation by welterde (prefers high BW over low latency).

To properly assign the right community to your peer, please reference the table below. If you are running your own network and peering internally, please also apply the communities inside your network.

## BGP community criteria
```
(64511, 1) :: latency \in [0, 2.7ms]
(64511, 2) :: latency \in [2.7ms, 7.3ms]
(64511, 3) :: latency \in [7.3ms, 20ms]
(64511, x) :: latency \in [exp(x-1), exp(x)] ms (for x < 10)
 
(64511, 21) :: bw >= 0.1mbit
(64511, 22) :: bw >= 1mbit
(64511, 23) :: bw >= 10mbit
(64511, 24) :: bw >= 100mbit
(64511, 25) :: bw >= 1000mbit
(64511, 2x) :: bw >= 10^(x-2) mbit
bw = min(up,down) for asymmetric connections
 
(64511, 31) :: not encrypted
(64511, 32) :: encrypted with unsafe vpn solution
(64511, 33) :: encrypted with safe vpn solution (but no PFS - the usual OpenVPN p2p configuration falls in this category)
(64511, 34) :: encrypted with safe vpn solution with PFS 

Propagation:
- - for latency pick max(received_route.latency, link_latency)
- - for encryption and bandwidth pick min between received BGP community and peer link
```
For example, if your peer is 12ms away and the link speed between you is 250Mbit/s and you are peering using OpenVPN P2P, then the community string would be (3, 24, 33).

## Example configurations 
```
# /etc/bird/peers4/tombii.conf
# As you are redefining the import and export filters, the check for is_valid_network() is no longer imported from dnpeers
# This comes from my own experience so let me know if I'm wrong :) -tombii
protocol bgp tombii from dnpeers {
  neighbor 172.23.102.x as 4242420321;
  import filter {
    if is_valid_network() && !is_self_net() then {
      update_flags(3,24,33);
      accept;
    }
    reject;
  };
  export filter {
    if is_valid_network() then {
      update_flags(3,24,33);
      accept;
    }
    reject;
  };
};
```
```
#/etc/bird/community_filters.conf
function update_latency(int link_latency) {
        bgp_community.add((64511, link_latency));
             if (64511, 9) ~ bgp_community then { bgp_community.delete([(64511, 1..8)]); return 9; }
        else if (64511, 8) ~ bgp_community then { bgp_community.delete([(64511, 1..7)]); return 8; }
        else if (64511, 7) ~ bgp_community then { bgp_community.delete([(64511, 1..6)]); return 7; }
        else if (64511, 6) ~ bgp_community then { bgp_community.delete([(64511, 1..5)]); return 6; }
        else if (64511, 5) ~ bgp_community then { bgp_community.delete([(64511, 1..4)]); return 5; }
        else if (64511, 4) ~ bgp_community then { bgp_community.delete([(64511, 1..3)]); return 4; }
        else if (64511, 3) ~ bgp_community then { bgp_community.delete([(64511, 1..2)]); return 3; }
        else if (64511, 2) ~ bgp_community then { bgp_community.delete([(64511, 1..1)]); return 2; }
        else return 1;
}

function update_bandwidth(int link_bandwidth) {
        bgp_community.add((64511, link_bandwidth));
             if (64511, 21) ~ bgp_community then { bgp_community.delete([(64511, 22..29)]); return 21; }
        else if (64511, 22) ~ bgp_community then { bgp_community.delete([(64511, 23..29)]); return 22; }
        else if (64511, 23) ~ bgp_community then { bgp_community.delete([(64511, 24..29)]); return 23; }
        else if (64511, 24) ~ bgp_community then { bgp_community.delete([(64511, 25..29)]); return 24; }
        else if (64511, 25) ~ bgp_community then { bgp_community.delete([(64511, 26..29)]); return 25; }
        else if (64511, 26) ~ bgp_community then { bgp_community.delete([(64511, 27..29)]); return 26; }
        else if (64511, 27) ~ bgp_community then { bgp_community.delete([(64511, 28..29)]); return 27; }
        else if (64511, 28) ~ bgp_community then { bgp_community.delete([(64511, 29..29)]); return 28; }
        else return 29;
}

function update_crypto(int link_crypto) {
        bgp_community.add((64511, link_crypto));
             if (64511, 31) ~ bgp_community then { bgp_community.delete([(64511, 32..34)]); return 31; }
        else if (64511, 32) ~ bgp_community then { bgp_community.delete([(64511, 33..34)]); return 32; }
        else if (64511, 33) ~ bgp_community then { bgp_community.delete([(64511, 34..34)]); return 33; }
        else return 34;
}
	
function update_flags(int link_latency; int link_bandwidth; int link_crypto)
int latency;
int bandwidth;
int crypto;
{
latency = update_latency(link_latency);
bandwidth = update_bandwidth(link_bandwidth) - 20;
crypto = update_crypto(link_crypto) - 30;
if bandwidth > 4 then bandwidth = 4;
<<<<<<< HEAD
bgp_local_pref = 100*bandwidth + 100*(10-latency)-100*bgp_path.len+50*crypto;
=======
>>>>>>> 8228e6e222fe084fe7b8fff23fddf55e68668d3d
return true;
} 
```
Please remember to include /etc/bird/community_filters.conf in your bird.conf/birdc6.conf
```
# filter helpers
#################

include "/etc/bird/filter4.conf";
<<<<<<< HEAD
**include "/etc/bird/community_filters.conf";**
=======
include "/etc/bird/community_filters.conf"; 
>>>>>>> 8228e6e222fe084fe7b8fff23fddf55e68668d3d
```


***

<<<<<<< HEAD
=======
### Bird bgp_local_pref calculation
If you are running a bigger network and also want to prioritize your traffic based on the communities, then you can look at the following below:
```
bgp_local_pref = 10000+100*bandwidth + 50*(10-latency)-200*bgp_path.len+100*crypto; (as suggested by tombii)
bgp_local_pref = 1000*bandwidth - 10*latency; if crypto < 2 then bgp_local_pref = 0; (as suggested by Jplitza)
```
This calculation goes into the /etc/bird/community_filters.conf  just above the return true; line. However for starters I recommend to skip the bgp_local_pref calculation part until you fully unterstand BGP routing and how this will affect not only you but the whole network. Assigning community flags to your peerings will hoever have an impact on dn42 in total. Remember, probably none of these alternatives are a good fit for your network, you will need to apply one and see how it affects your traffic and then going back and tweaking the formula and checking again.

>>>>>>> 8228e6e222fe084fe7b8fff23fddf55e68668d3d
Original implementation by Jplitza: https://gist.github.com/welterde/524cc9b37a618e29093d

All props to him for the bird code based on the suggestion from welterde. 

Original email from welterde: http://lists.spaceboyz.net/pipermail/dn42/2015-February/000982.html

My modification is only for the calculation of bgp_local_pref to adjust for prefering lower latency. Feel free to play around with the formula to find something that suits your needs.
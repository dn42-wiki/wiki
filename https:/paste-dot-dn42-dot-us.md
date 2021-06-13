➜  ~ birdc show route for 172.21.100.16 all
BIRD 2.0.7 ready.
Table master4:
172.21.100.16/29     unicast [dn42_3088_v6 07:32:53.332] ! (100) [AS38173i]
	via fe80::3088:198 on wg3088
	Type: BGP univ
	BGP.origin: IGP
	BGP.as_path: 4242423088 38173
	BGP.next_hop: :: fe80::3088:198
	BGP.local_pref: 100
	BGP.large_community: (4242423088, 1, 198)
                     unicast [dn42_0604 07:35:38.812] (100) [AS38173i]
	via 172.23.89.1 on wg0604
	Type: BGP univ
	BGP.origin: IGP
	BGP.as_path: 4242420604 4242421331 4242423088 38173
	BGP.next_hop: 172.23.89.1
	BGP.local_pref: 100
	BGP.community: (64511,1) (64511,25) (64511,34)
	BGP.large_community: (207268, 1, 52) (4242420604, 2, 23) (4242420604, 501, 4242421331) (4242420604, 502, 51) (4242420604, 504, 1) (4242423088, 1, 192)
                     unicast [dn42_2980_v6 07:33:03.142 from fe80::2980] (100) [AS38173i]
	via 172.23.105.2 on wg2980
	Type: BGP univ
	BGP.origin: IGP
	BGP.as_path: 4242422980 4242423088 38173
	BGP.next_hop: 172.23.105.2
	BGP.local_pref: 100
	BGP.community: (64511,24) (64511,34)
	BGP.large_community: (4242423088, 1, 191)
                     unicast [dn42_0925_v6 04:52:00.296 from fe80::925] (100) [AS38173i]
	via 172.21.68.32 on wg0925
	Type: BGP univ
	BGP.origin: IGP
	BGP.as_path: 4242420925 4242423088 38173
	BGP.next_hop: 172.21.68.32
	BGP.local_pref: 100
	BGP.community: (64511,6) (64511,24) (64511,34)
	BGP.large_community: (4242420925, 10, 2) (4242420925, 20, 4209255236) (4242420925, 30, 4209255132) (4242420925, 51, 1) (4242420925, 52, 1) (4242423088, 1, 192)
                     unicast [dn42_1826 04:51:55.440] (100) [AS38173i]
	via 172.20.143.50 on wg1826
	Type: BGP univ
	BGP.origin: IGP
	BGP.as_path: 4242421826 4242423088 38173
	BGP.next_hop: 172.20.143.50
	BGP.local_pref: 100
	BGP.community: (64511,1) (64511,24) (64511,33)
	BGP.large_community: (4242423088, 1, 193)
                     unicast [dn42_3914 04:51:53.076] (100) [AS38173i]
	via 172.20.53.105 on wg3914
	Type: BGP univ
	BGP.origin: IGP
	BGP.as_path: 4242423914 4242423088 38173
	BGP.next_hop: 172.20.53.105
	BGP.local_pref: 100
	BGP.large_community: (4242423088, 1, 192)
                     unicast [dn42_2464_v6 04:51:12.079 from fe80::2464] (100) [AS38173i]
	via 172.20.191.204 on wg2464
	Type: BGP univ
	BGP.origin: IGP
	BGP.as_path: 4242422464 4242423088 38173
	BGP.next_hop: 172.20.191.204
	BGP.local_pref: 100
	BGP.large_community: (4242422464, 2, 4242423088) (4242422464, 64511, 52) (4242423088, 1, 192)
                     unicast [dn42_1876_v6 04:51:12.276 from fe80::1876] (100) [AS38173i]
	via 172.22.66.57 on wg1876
	Type: BGP univ
	BGP.origin: IGP
	BGP.as_path: 4242421876 4242422601 4242423088 38173
	BGP.next_hop: 172.22.66.57
	BGP.local_pref: 100
	BGP.community: (64511,1) (64511,24) (64511,34)
	BGP.large_community: (4242422601, 120, 23) (4242423088, 1, 198)
                     unicast [dn42_2330_v6 04:51:12.252] (100) [AS38173i]
	via fe80::2330:5 on wg2330
	Type: BGP univ
	BGP.origin: IGP
	BGP.as_path: 4242422330 4242423088 38173
	BGP.next_hop: :: fe80::2330:5
	BGP.local_pref: 100
	BGP.large_community: (4242423088, 1, 192)
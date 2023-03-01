we're planning to have a mcast-ix.dn42 somewhere in the cloud at #dn42 for years now...

now we have a pull req with cosmetical issues only: https://git.dn42.dev/dn42/registry/pulls/2575

the main goal is to have a shared lan where ases can peer to each other with the following conditions:
* pure ethernet
* low latency between the vms
* native support for jumbo frames
* possibility to private vlans between participants with the same conditions

how to participate:

all you have to do is prepare a qcow2 or vmdk image and upload it to somewhere and ping nop-mnt (mc36 @ irc) with the url... i'll wget it once then boot up your vm connected to the switchport... you'll have raw dn42 reachability there and pat-ed clearnet to continue your installation or upgrades or to connect to the rest of your infra...

only one thing to look for twice, the "console=ttyS0,115200n8" be present as a kernel parameter... if you need vnc access instead, just ask for it....

if you need a private peering here between you and an other participant, just ask for a private ethernet connection...

consider enabling lldp on your interfaces because it helps speed up things on the switch moreover if you'll have more interfaces there then will help you too...

also consider enabling pim sparse mode on the ix and then you can have the rtp://172.23.199.110@232.2.3.2:1234/ stream

last but not least, always save your configs! there will be a daily recurring power cut sheduled to 21:00pm cest +-1minutes to have the infrastructure auto-upgraded...



the whole idea is to consider the following hypervisor configuration:

```
dn42ix#show startup-config vdc
vdc definition vm-bri
 connect ethernet2825 vm-switch
 cpu host
 image /rtr/ix/bri.img
 memory 1024
 nic virtio-net-pci
 mac cafe.beef.b00b
 exit
vdc definition vm-clearnet
 local ethernet66602
 local ethernet66603
 connect ethernet66601 vm-switch
 exit
vdc definition vm-jlu5
 connect ethernet1080 vm-switch
 cpu host
 image /rtr/ix/jlu5.img
 cdrom /rtr/ix/jlu5.iso
 memory 1024
 exit
vdc definition vm-lare
 connect ethernet3035 vm-switch
 cpu host
 image /rtr/ix/lare.img
 memory 1024
 exit
vdc definition vm-nop
 connect ethernet1955 vm-switch
 exit
vdc definition vm-routeserver
 exit
vdc definition vm-switch
 connect ethernet1080 vm-jlu5
 connect ethernet1955 vm-nop
 connect ethernet2825 vm-bri
 connect ethernet3035 vm-lare
 connect ethernet66601 vm-clearnet
 exit

dn42ix#
```

now you can have drop-in replacement vm-s to experiment with like whats it looks a like if the ix is provisioned on a juniper vsrx3 shitload or a cisco nxosv or plain freerouter in software mode or in p4dpdk mode.... 

then publishing a small report on r/networking on behalf of #dn42 measurements

and probably doing even more crazyer projects/experiment if we settle to have a proper dn42 ix finally with low latency shared vlan between the vms...

like a real ix...

static addressing plan, there is a randomized dhcp and slaac on the subnet but consider picking up a static ip and pere with that:



| nick/mnter     | asn* | your-ipv4-fixed-ip | your-ipv6-fixed-ip                    | your-ipv6-linklocal      | public lg                                                 |
|:---------------|:-----|:-------------------|:--------------------------------------|:-------------------------|:----------------------------------------------------------|
| sw1-mcastix    | 1951 | N/A                | N/A                                   | N/A                      | TBD: SOON                                                 |
| rs1-mcastix    | 1951 | 172.23.124.126/27  | fde0:93fa:7a0:c1ca::179/64            | fe80::200:bff:fead:beef  | TBD: SOON                                                 |
| rtr1-badcorp   | 1952 | 172.23.124.97/27   | fde0:93fa:7a0:c1ca::666/64            | fe80::260:54ff:fe33:2178 | TBD: SOON                                                 |
| rtr1-nop       | 1955 | 172.23.124.122/27  | fde0:93fa:7a0:c1ca::1955/64           | fe80::200:ccff:fe1e:c0de | telnet sandbox.freertr.org                                |
| rtr1-catgirls  | 1411 | 172.23.124.101/27  | TBD                                   | fe80::1411:5             | TBD: SOON                                                 |
| rtr1-catgirls2 | 1411 | TBD                | TBD                                   | TBD                      | TBD: SOON                                                 |
| rtr1-lare      | 3035 | 172.23.124.114/27  | fde0:93fa:7a0:c1ca:0:42:4242:3035/64  | fe80::21f:45ff:fe11:7356 | clearnet: https://lg.lare.cc/ dn42: https://lg.lare.dn42/           |
| rtr1-bri       | 2825 | TBD                | TBD                                   | TBD                      | TBD                                                       |
| rtr1-jlu5      | 1080 | TBD                | TBD                                   | TBD                      | TBD                                                       |
| rtr1-fl        | 1975 | TBD                | TBD                                   | TBD                      | TBD                                                       |




TBD: add yourself please here while keeping some ordering

*: so your as number xxxx shortened here, the rigthmost part after the 424242xxxx.... this also will be your ethernetXXXX and so on so just remember this by heart XD


| function        |  console                   |
|:----------------|:---------------------------|
| switch          | telnet ix.nop.dn42 20001   |
| route server 1  | telnet ix.nop.dn42 20002   |
| bad-corp-rtr1   | telnet ix.nop.dn42 20003   |


public mrt dumps and config archive of the infra at http://ix.nop.dn42/ here



it'll be shitload in the beginning but hopefully it could improve the common knowledge....


please consider joining #dn42-ix#2 to speed up sorting out potential issues, etc

we're planning to have a mcast-ix.dn42 somewhere in the cloud at #dn42 for years now...

now we have a pull req with cosmetical issues only: https://git.dn42.dev/dn42/registry/pulls/2572

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

it'll be shitload in the beginning but hopefully it could improve the common knowledge....


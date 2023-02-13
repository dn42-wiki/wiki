## multicast

rfc8815 deprecated pim-sm so pim-ssm is the way to go!

for it to work, you'll need to do the following:

ask your peering to enable ipv4/ipv6 multicast afi on your peering

set up ipv4/ipv6 pim for the (s,g) joins to pass through

you're done, you should have your streams

current participants:

nop-mnt

grawity-mnt

mirsal-mnt

C4TG1RL5-MNT 
[frr configuration](https://git.lemonsh.moe/C4TG1RL5/dn42/src/branch/master/lab.rtr.famfo.catgirls.dn42/frr) 
_Please make sure you understand how to configure and use frr before you use anything from this configuration!_

current streams:

vlc rtp://172.23.199.110@232.2.3.2:1234/

feel free to ask for a peering and set it up!

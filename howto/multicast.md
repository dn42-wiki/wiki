## multicast

rfc8815 deprecated pim-sm so pim-ssm is the way to go!

for it to work, you'll need to do the following:

ask your peering to enable ipv4/ipv6 multicast afi on your peering

set up ipv4/ipv6 pim for the (s,g) joins to pass through

you're done, you should listen to some metal via " vlc rtp://172.23.199.110@232.2.3.2:1234/ "

current participants:

nop-mnt

grawity-mnt

mirsal-mnt

current streams:

vlc rtp://172.23.199.110@232.2.3.2:1234/

feel free to ask for a peering and set it up!

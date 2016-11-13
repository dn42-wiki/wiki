## EdgeRouterPro-8 DN42 config example with v1.9.0 

After a lot of searching and trying I [Phil/ALS7] finnaly got a working config

##Features

* IPv4/IPv6 Tunnel via OpenVPN
* dn42 DNS

##How-To

--> still work in Progress

1) you need to create all required fields in the registry --> look at [[Getting started]] page.

The data i used are the following:

Own ASN: AS4242422684
Own IPv4: 172.20.4.64/27
Own IPv6: fd33:ac1d:d1ce::/48

2) get a peer --> ask nice @ [[IRC]]

3) You need following data

--tunnel options, secret key
--ASN from the peer (in this example i use remote-as XXXXX)
--ip's

...


start a ssh session to your router

copy vpn key to /config/auth/giveITaName -- Create folder if needed

configure
set interface openssh vtun0  
set interfaces openvpn vtun0 mode site-to-site  
set interfaces openvpn vtun0 local-port 1194  //you get the port from your peer  
set interfaces openvpn vtun0 remote-port 1194 //you get the port from your peer  
set interfaces openvpn vtun0 local-address 172.20.4.64 //your sife dn42 ip address  
set interfaces openvpn vtun0 remote-address X.X.X.X //dn42 link address from your peer  
set interfaces openvpn vtun0 remote-host X.X.X.Y //clearnet ip address from your peer  
set interfaces openvpn vtun0 shared-secret-key-file /config/auth/giveITaName  // your keyfile  
set interfaces openvpn vtun0 openvpn-option "--comp-lzo"  //if your peer support compression  
commit   
save  

Now the ipv4 tunnel should be up&running







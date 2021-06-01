# Example Configuration for direct peer to peer

* Replace `<PEER_NAME>` with a self chosen name to identify this peer
* Replace `<PROTO>` with either `udp` or `udp6`, depending if you reach your remote peer with ipv4 o ipv6
* Replace `<REMOTE_HOST>` with the public ip address of your peer
* Replace `<REMOTE_PORT>` with the port number, where your peer's openvpn daemon listen for traffic
* Replace `<LOCAL_HOST>` with your public ip
* Replace `<INTERFACE_NAME>` with a self chosen name, this will be the name of your network interface (tun device) for this peering
* Replace `<LOCAL_GATEWAY_IP>` with your own dn42 ip address
* Replace `<REMOTE_GATEWAY_IP>` with dn42 ip address of your peer
* `<LOCAL_GATEWAY_IPV6> <REMOTE_GATEWAY_IPV6>` same as ipv4, but both ip addresses needs to be in the same subnet. For simplicity you can always use an address from link-local ipv6 range (fe80::/64)

```
#/etc/openvpn/<PEER_NAME>
proto       <PROTO>
mode        p2p
remote      <REMOTE_HOST>
rport       <REMOTE_PORT>
local       <LOCAL_HOST>
lport       <LOCAL_PORT>
dev-type    tun
tun-ipv6
resolv-retry infinite
dev         <INTERFACE_NAME>
comp-lzo
persist-key
persist-tun
cipher aes-256-cbc
ifconfig-ipv6 <LOCAL_GATEWAY_IPV6> <REMOTE_GATEWAY_IPV6>
ifconfig <LOCAL_GATEWAY_IP>  <REMOTE_GATEWAY_IP>
secret /etc/openvpn/<PEER_NAME>.key

# The secret can also be included inline with the config by 
#   wrapping it in <secret></secret> tags.
# <secret>
# ... Key File contents go here ...
# </secret>
```

then create a new key and share it with your peer

```
$ openvpn --genkey --secret /etc/openvpn/<PEER_NAME>.key
```

# Example Configuration if one peer has a floating ip

## peer with fixed ip

```
proto       <PROTO>
mode        p2p
dev-type    tun
comp-lzo
dev  <INTERFACE_NAME>
persist-key
persist-tun
tun-ipv6
cipher aes-256-cbc
resolv-retry infinite
float
port <LOCAL_PORT>
ifconfig-ipv6 <LOCAL_GATEWAY_IPV6> <REMOTE_GATEWAY_IPV6>
ifconfig  <LOCAL_GATEWAY_IP>  <REMOTE_GATEWAY_IP>
secret /etc/openvpn/<PEER_NAME>.key
```

## peer with floating ip

* Notice the local gateway ip of your peer is your remote gateway ip and 
  his remote gateway is your local gateway
* `<REMOTE_HOST>` is the ip address of your peer
* `<REMOTE_PORT>` is openvpn port, where your peer listen for traffic

```
proto       <PROTO>
mode        p2p
remote      <REMOTE_HOST>
rport       <REMOTE_PORT>
lport       float
dev-type    tun
tun-ipv6
dev         <INTERFACE_NAME>
comp-lzo
persist-key
persist-tun
cipher aes-256-cbc
resolv-retry infinite
ifconfig  <LOCAL_GATEWAY_IP>  <REMOTE_GATEWAY_IP>
ifconfig-ipv6 <LOCAL_GATEWAY_IPV6> <LOCAL_GATEWAY_IPV6>
secret /etc/openvpn/<PEER_NAME>.key
```

# Example configuration for connecting roaming clients to dn42

Clients connect using certificates, and simply get attributed dn42 IPs in the order they connect.  This is useful for roaming clients, where you don't really care which IP you have.  Note that once a client has connected for the first time, it will keep the same IP on subsequent connections (option `ifconfig-pool-persist`).

## Server configuration

Replace `<PORT>` with the UDP port you want OpenVPN to listen to, and change the IP ranges (`ifconfig` and `route-gateway` options).

```
mode server
tls-server

dh dh2048.pem
cipher aes-256-cbc

ca keys/ca.crt
cert keys/roaming-dn42.crt
key keys/roaming-dn42.key

client-config-dir /etc/openvpn/roaming

dev tun-roaming
persist-tun
tun-ipv6
tun-mtu 1500
fragment 1300
mssfix
log /var/log/openvpn-dn42-roaming.log
status /var/log/openvpn-dn42-roaming-status.log 60

# Should work for both IPv4 and IPv6
proto udp6
port <PORT>

# IPv6
###tun-ipv6
###push tun-ipv6
###ifconfig-ipv6 2001:db8:42:42::1 2001:db8:42:42::2
###ifconfig-ipv6-pool 2001:db8:42:42::3/64

topology subnet
push "topology subnet"

keepalive 10 60

# That's 172.22.X.144/28 (172.22.X.144 to 172.22.X.159)
ifconfig 172.22.X.145 255.255.255.240
ifconfig-pool 172.22.X.146 172.22.X.158 255.255.255.240

ifconfig-pool-persist pool-persist.txt

push "route-gateway 172.22.X.145"
push "route 172.22.0.0 255.254.0.0" 
###push "route 172.31.0.0 255.255.0.0" 
###push "route 10.0.0.0 255.0.0.0" 
```

## Client configuration

Change `<SERVER>` and `<PORT>`.

```
client

ca   ca.crt
cert myclient.crt
key  myclient.key

dev tun
proto udp6
cipher aes-256-cbc

remote <SERVER> <PORT>

tun-mtu 1500
fragment 1300
mssfix

route-delay 2
nobind
persist-key
persist-tun
resolv-retry infinite

verb 3
```

## Certificate management

Use easy-rsa, it's easy to use.  Below is a very short description, find a real tutorial if you don't know how it works.

Build the CA: `. vars`, `./build-ca`, then generate the server key: `./build-key-server roaming-dn42`.

Then, for each client, generate a private key and a certificate: ```./build-key myclient```.  The Common Name is the only important information (it will be used to identify the client, for instance in the logs).

# See also
* [Network settings](https://internal.dn42/howto/networksettings)

# External Links
* multicast: 
 * **OpenVPN**
 * [Optimizations for multicast over TAP w/ OpenVPN](https://community.openvpn.net/openvpn/ticket/79)
 * [Sending multicast over a openvpn tunnel](http://forums.openvpn.net/topic8036.html)

 * **RFC**
 * [IPv6 - RFC3306](https://tools.ietf.org/html/rfc3306)
 * [IPv4 - multicast](https://en.wikipedia.org/wiki/Multicast_address#GLOP_addressing)
 * [IPv4 - GLOB calculator](http://labs.spritelink.net/glop)
 * [RFC3108 GLOP Addressing in 233/8](http://tools.ietf.org/html/rfc3180)
 * [RFC3138 Extended Assignments in 233/8](https://tools.ietf.org/html/rfc3138)

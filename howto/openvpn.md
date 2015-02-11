# Example Configuration for direct peer to peer
* Replace `<PEER_NAME>` with a self chosen name to identify this peer
* Replace `<PROTO>` with either `udp` or `udp6`, depending if you reach your remote peer with ipv4 o ipv6
* Replace `<REMOTE_HOST>` with the public ip address of your peer
* Replace `<REMOTE_PORT>` with the port number, where your peer's openvpn daemon listen for traffic
* Replace `<LOCAL_HOST>` with your public ip
* Replace `<INTERFACE_NAME>` with a self chosen name, this will be the name of your network interface (tun device) for this peering
* Replace `<LOCAL_GATEWAY_IP>` with your own dn42 ip address
* Replace `<REMOTE_GATEWAY_IP>` with dn42 ip address of your peer

```
#/etc/openvpn/<PEER_NAME>
daemon
proto       <PROTO>
mode        p2p
remote      <REMOTE_HOST>
rport       <REMOTE_PORT>
local       <LOCAL_HOST>
lport       <LOCAL_PORT>
dev-type    tun
dev         <INTERFACE_NAME>
comp-lzo
persist-key
persist-tun
ifconfig    <LOCAL_GATEWAY_IP>  <REMOTE_GATEWAY_IP>
secret /etc/openvpn/<PEER_NAME>.key
```

then create a new key and share it with your peer

```
$ openvpn --genkey --secret /etc/openvpn/<PEER_NAME>.key
```

# Example Configuration if one peer has floating ip

## peer with fixed ip

```
daemon
proto       <PROTO>
mode        p2p
dev-type    tun
comp-lzo
dev         <INTERFACE_NAME>
persist-key
persist-tun
float
port        <LOCAL_PORT>
ifconfig    <LOCAL_GATEWAY_IP>  <REMOTE_GATEWAY_IP>
secret /etc/openvpn/<PEER_NAME>.key
```

## peer with floating ip

* Notice the local gateway ip of your peer is your remote gateway ip and 
  his remote gateway is your local gateway
* `<REMOTE_HOST>` is the ip address of your peer
* `<REMOTE_PORT>` is openvpn port, where your peer listen for traffic

```
daemon
proto       <PROTO>
mode        p2p
remote      <REMOTE_HOST>
rport       <REMOTE_PORT>
lport       float
dev-type    tun
dev         <INTERFACE_NAME>
comp-lzo
persist-key
persist-tun
ifconfig    <LOCAL_GATEWAY_IP>  <REMOTE_GATEWAY_IP>
secret /etc/openvpn/<PEER_NAME>.key
```
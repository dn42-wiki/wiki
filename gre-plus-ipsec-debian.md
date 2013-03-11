# GRE + IPsec on Debian based distros

* Install racoon from ipsec-tools.
* Define an IPsec security policy in /etc/ipsec-tools.conf
* Load the IPsec security policy into the IPsec security policy database.
* Configure the racoon daemon.
* Configure a GRE tunnel.

## Used resources in this example:
* tunnel endpoints: 1.2.3.4 and 5.6.7.8
* internal IPv4 addresses: 10.0.0.1 and 10.0.0.2

## Define an IPsec security policy
Example policy on 1.2.3.4:
```
#!/usr/sbin/setkey -f
spdadd 1.2.3.4 5.6.7.8 gre -P out ipsec esp/transport//require;
spdadd 5.6.7.8 1.2.3.4 gre -P in  ipsec esp/transport//require;
```
Change the direction on 5.6.7.8.

## Load the IPsec security policy into the IPsec security policy database
Load the policy with the setkey command.
```
setkey -f /etc/ipsec-tools.conf
```
Afterward check the policy database with:
```
setkey -DP
```

## Configure the racoon daemon
An example /etc/racoon/racoon.conf.
```
path pre_shared_key "/etc/racoon/psk.txt";
path certificate    "/etc/racoon/certs";
log info;

listen {
  # replace with local tunnel endpoint
  isakmp      1.2.3.4 [500];
  isakmp_natt 1.2.3.4 [4500];
}

# replace with remote tunnel endpoint
remote 5.6.7.8 [500] {
  exchange_mode    main;
  proposal_check   strict;
  my_identifier    asn1dn;
  peers_identifier asn1dn;
  lifetime         time 1 hour;
  certificate_type x509 "local.crt" "local.key";
  peers_certfile   x509 "remote.crt";
  ca_type          x509 "ca.crt";
  verify_cert      on;
  send_cert        off;
  send_cr          off;
  
  proposal {
    encryption_algorithm  aes 256;
    hash_algorithm        sha256;
    authentication_method rsasig;
    dh_group              modp4096;
  }
}

# local tunnel endpoint, remote tunnel endpoint, GRE ip protocol number
sainfo (address 1.2.3.4 address 5.6.7.8 47) {
  pfs_group                modp4096;
  lifetime                 time 1 hour;
  encryption_algorithm     aes 256;
  authentication_algorithm hmac_sha1;
  compression_algorithm    deflate;
}
```

## Configure a GRE tunnel
Add this to /etc/network/interfaces:
```
auto tun0
iface tun0 inet static
  address 10.0.0.1
  netmask 255.255.255.255
  up ifconfig tun0 multicast
  pre-up iptunnel add tun0 mode gre local 1.2.3.4 remote 5.6.7.8 ttl 255
  pointtopoint 10.0.0.2
  post-down iptunnel del tun0
```

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

## Load the IPsec security policy into the IPsec security policy database
Load the policy with the setkey command.
```
setkey -f /etc/ipsec-tools.conf
```
Afterward check the policy database with:
```
setkey -DP
```

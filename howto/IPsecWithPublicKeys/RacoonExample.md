Quote from #dn42: `hexa: nobody runs racoon on their own free will :)`.

See also [strongswan](howto/IPsecWithPublicKeys/strongSwan5Example)

The keys are generated with plainrsa-gen.

```
Usage: plainrsa-gen [options]

  -b bits       Generate <bits> long RSA key (default=1024)
  -e pubexp     Public exponent to use (default=0x3)
  -f filename   Filename to store the key to (default=stdout)
  -i filename   Input source for format conversion
  -h            Help
```
I'd probably go with 4096 bits.


in your racoon.conf:
```
path certificate "/etc/racoon/keys";

listen {
  isakmp 192.168.255.1[500];
}

remote 192.168.255.2 {
  exchange_mode main;
  certificate_type plain_rsa "local.priv.key";
  peers_certfile plain_rsa "remote.pub.key";
  proposal {
    authentication_method rsasig;
    lifetime time 8 hour;
    encryption_algorithm aes256;
    hash_algorithm sha256;
    dh_group modp1024;
  }
}
```

## Se also

[debian specific configuration](IPsecWithPublicKeys/GRE plus IPsec Debian)
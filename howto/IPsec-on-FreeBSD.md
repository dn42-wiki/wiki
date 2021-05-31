# IPsec on FreeBSD

These instructions are for IPsec in transport mode not IPsec in tunnel mode. IPsec in tunnel mode requires a too tight coupling with the routing table for dynamic routing because the policies can only be specified based on source/destination address and protocol not based on interfaces.

## Requirements
* Root access to both endpoints.
* Static IPv4 addresses for both endpoints unless you want to write a small shell script as hook for racoon.
* At least one static IPv4 on at least one endpoint unless you hate yourself.

## Kernel configuration
The FreeBSD GENERIC kernel lacks support for in-kernel IPsec processing. Add this two lines to your kernel config and (re-)build your own kernel.
If you're new to FreeBSD check Chapters [15.9.1](http://www.freebsd.org/doc/handbook/ipsec.html) and [9](http://www.freebsd.org/doc/handbook/kernelconfig.html) of the FreeBSD handbook.
```
  options   IPSEC        #IP security
  device    crypto
```
Reboot into your new kernel.

## Userland configuration

Install the racoon daemon. It's included in the [security/ipsec-tools](http://www.freshports.org/security/ipsec-tools/) port.
Racoon is pain in the ass to configure the first time because it's error messages aren't helping and the complexity of IPsec. Don't let this stop you.
```
path    pre_shared_key  "/usr/local/etc/racoon/psk";
path    certificate     "/usr/local/etc/racoon/certs";
log     info;

listen {
        isakmp          a.b.c.d [500];
        isakmp_natt     a.b.c.d [4500];
}

padding {
        strict_check    on;
}

timer {
        natt_keepalive   5 sec;
        interval         3 sec;
        phase1          45 sec; # give embedded CPUs time to finish RSA operations
        phase2          45 sec;
}

remote b.c.d.e [500] {
        exchange_mode           main;
        proposal_check          strict;
        my_identifier           asn1dn;
        peers_identifier        asn1dn;
        lifetime                time 1 hour;
        certificate_type        x509 "self.crt" "self.key";
        peers_certfile          x509 "peer.crt";
        ca_type                 x509 "ca.crt";
        verify_cert             on;
        send_cert               off; # neither send
        send_cr                 off; # nor request a crt to be send

        proposal {
                encryption_algorithm    aes 256;
                hash_algorithm          sha256;
                authentication_method   rsasig;
                dh_group                modp4096;
        }
}

sainfo (address a.b.c.d gre address b.c.d.e gre) {
        pfs_group                       modp4096;
        lifetime                        time 1 hour;
        encryption_algorithm            aes 256;
        authentication_algorithm        hmac_sha1;
}

```

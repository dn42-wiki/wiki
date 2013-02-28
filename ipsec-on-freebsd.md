# IPsec on FreeBSD

These instructions are for IPsec in transport mode not IPsec in tunnel mode. IPsec in tunnel mode requires a too tight coupling with the routing table for dynamic routing because the policies can only be specified based on source/destination address and protocol not based on interfaces.

## Requirements
* Root access to both endpoints.
* Static IPv4 addresses for both endpoints unless you want to write a small shell script as hook for raccon.
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
 x
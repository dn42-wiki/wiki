# IPsec with public key authentication
## Stop using pre-shared keys!
### Pre-shared keys suck, because _reasons_

*  __The key must be kept secret__, which means it must be shared only over a secure channel e.g. PGP, face-to-face
*  Most implementations will accept insecure (too short, too simple) keys
*  The [insecure][1] [IKE][2] [aggressive mode][3] must be used to support distinct PSKs for multiple dynamic peers, or
*  All dynamic peers must use the same PSK in order to use the more secure IKE main mode

[1]: http://www.sersc.org/journals/IJAST/vol8/2.pdf "Vulnerabilities of VPN using IPSec and Defensive Measures"
[2]: http://carnal0wnage.attackresearch.com/2011/12/aggressive-mode-vpn-ike-scan-psk-crack.html "Aggressive Mode VPN -- IKE-Scan, PSK-Crack, and Cain"
[3]: http://rayas-security.blogspot.com/2013/06/ipsec-vpn-main-mode-vs-aggressive-mode.html "IPsec VPN, Main mode Vs Aggressive mode"

### Public keys are _better_
*  They can be transmitted over insecure channels without compromising security
*  No need to generate a new key for each connection; just send the same public key to each new peer
*  Most implementations generate keys using high quality random numbers by default; one must _try_ to generate an insecure key
*  Dynamic peers can all have distinct public keys and still use IKE main mode

### So why isn't everyone using public keys already?
*  The various IKE implementations don't all use the same format to represent public keys _(but we can **easily** [convert](#Conversion-tool) between them)_
*  Documentation on [how to](#How-To-examples), and why you should, configure public keys instead of PSKs is lacking _(which is why this page exists)_
*  Some implementations don't expose the ability to use public keys directly, only allowing a choice between PSKs and X.509 certificates

### Public keys means certificates, right? Certificates are hard :(
Many IKE implementations support manually configuring trusted public keys, without having to create a CA, generate CSRs, sign certificates, or remember/look up the commands to do those things.

Keep in mind that certificates are just public keys wrapped with some extra metadata so that your router can automatically verify that it belongs to someone you trust. Certificates are useful for instances where there are so many peers that it's infeasible to manually configure each one's public key, such as a "road warrior" configuration or DMVPN. In those scenarios it makes sense to set up a Certificate Authority to handle it.

## Ok fine, how do I public key?
### Conversion tool
Different implementations use different formats to represent public keys, and it's necessary to be able to convert between them. Here is a script for that purpose:

https://github.com/ryanriske/pubkey-converter

### How-To examples
| Implementation           | Key format      |
| :----------------------- | --------------: |
| [Cisco IOS][a]           | Hexadecimal DER |
| [Mikrotik RouterOS][b]   | PEM             |
| [OpenBSD][c]             | PEM             |
| [Racoon][d]              | Base64 RFC 3110 |
| [strongSwan < 5.0.0][e]  | Base64 RFC 3110 |
| [strongSwan >= 5.0.0][f] | PEM             |
| [VyOS/EdgeOS][g]         | Base64 RFC 3110 |

[a]: /howto/IPsecWithPublicKeys/CiscoIOSExample
[b]: /howto/IPsecWithPublicKeys/RouterOSExample
[c]: /howto/IPsecWithPublicKeys/OpenBSDExample
[d]: /howto/IPsecWithPublicKeys/RacoonExample
[e]: /howto/IPsecWithPublicKeys/strongSwan4Example
[f]: /howto/IPsecWithPublicKeys/strongSwan5Example
[g]: /howto/IPsecWithPublicKeys/VyOSExample

### Notes
1.  Best practice is to generate the private key on the router itself, and not transfer it to another machine. This part should be kept secret!
2.  Generate a key of at least 2048 bits, preferably 4096 if both ends support it.
3.  Some implementations support more than one key format. The examples here only show how to use one of them (usually PEM) for brevity.
4.  RFC 3110 format is the same as that described in RFC 2537. The former obsoletes the latter.
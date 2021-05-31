DN42 ACME CA
==================

Certificates can be automatically generated with the [ACME-CA](http://acme.dn42). More information can be found on [acme.dn42](http://acme.dn42/)

DN42 Self-Serve CA
==================

This client is used for automating the process of requesting TLS certificates. (Available via: [dn42](https://ca.dn42/ca-client), [iana](https://ca.dn42.us/ca-client), [git](anon@git.dn42:dn42/ca-client)) 


VALIDATION PROCESS
==================

The process validates ownership by verifying control of both a users MNT object in the registry and the authoritative DNS server. 
The following steps take place in creating a signed certificate.

*User Flow*

1. User generates a 2048+ bit rsa key and CSR for their MNT object.
2. User generates a sha256 hash of the rsa public key (commonly known as a [public keypin][keypin]) and adds it as a remark in their MNT
3. User submits the csr to the CA to validate and sign. 
4. CA checks for the keypin in their MNT object (if it does not find it in the local copy of the [monotone repo][ca-mtn] it will check against io.nixnodes.dn42)
5. (optional) CA revokes prior certificate as superseded.
6. CA signs and returns the user certificate.

*Server Flow*

1. User generates a 2048+ bit rsa key and CSR for the dns CN. Also including any SAN domains.
2. User generates a sha256 hash of the rsa public key (commonly known as a [public keypin][keypin]) and adds it as a txt record in their DNS.
3. User uses the user certificate to authenticate and submits the csr to the CA to validate and sign. 
4. CA checks for the user keypin in their MNT object (if it does not find it in the local copy of the [monotone repo][ca-mtn] it will check against io.nixnodes.dn42)
5. CA checks the dns records for the CN and each SAN for the tls keypin.
6. (optional) CA revokes prior certificate as superseded.
7. CA signs and returns the tls certificate.

*User Renewals*

User certificates are signed for 180 days. To renew follow the steps above starting from number 3.

*Server renewals*

Server certificates are signed for 45 days. To renew follow the steps above starting from number 3. 

[keypin]: https://developer.mozilla.org/en-US/docs/Web/Security/Public_Key_Pinning
[ca-mtn]: https://ca.dn42/reg/mntner/

*Certificate Revocations*

1. User uses the user certificate to authenticate and submits the serial and revoke reason to CA.
2. CA checks user keypin in their MNT object (if it does not find it in the local copy of the [monotone repo][ca-mtn] it will check against io.nixnodes.dn42)
3. CA checks that owner in certificate matches.
4. CA revokes certificate and updates revocation list.

INSTALL
=======

get the script here: 

curl https://ca.dn42/ca.dn42 > ca.dn42; chmod +x ca.dn42

available via git: anon@git.dn42:dn42/ca-client


KNOWN ISSUES
============

## openssl prior to 1.0.2 returns "SSL certificate problem: permitted subtree violation"

The way openssl validated name constraints prevented it from accepting dns names that started with a dot.
Because the name constraint is "DNS:.dn42" it fails to validate.

[Read more on this mailing list thread][libssl-1]


[libssl-1]: https://groups.google.com/forum/#!topic/mailing.openssl.dev/drG3U-S4iaE


## X.509 nameConstraints on certificates not supported on OS X

Browsers and clients that rely on Apple's [Secure Transport][osx-1] library does not support X.509's nameConstraints.

Read more on this [stack exchange post][osx-2]


[osx-1]: https://developer.apple.com/library/mac/documentation/Security/Reference/secureTransportRef/
[osx-2]: http://security.stackexchange.com/a/97133


How to Run
==========

````
Usage:  # OWNER is your MNT handle.
   ./ca.dn42 user-gen OWNER EMAIL          # Output to OWNER.csr and OWNER.key
   ./ca.dn42 user-sig OWNER                # Output to OWNER.crt and OWNER.p12
   ./ca.dn42 tls-gen DNS OWNER EMAIL [SAN] # Output to OWNER_DNS.csr and OWNER.key
   ./ca.dn42 tls-sig DNS OWNER             # Output to OWNER_DNS.crt and OWNER_DNS.p12
   ./ca.dn42 revoke OWNER CERTFILE [REASON]


Revoke Reasons: unspecified, keyCompromise, affiliationChanged,
   superseded, cessationOfOperation, certificateHold, removeFromCRL

Environtment Options:
   DN42CA_PKCS12 = 1                # Generate pkcs12 file for certificate.
````

Example
=======

Generate the user key

````
$ ./ca.dn42 user-gen XUU-MNT xuu@sour.is
Generating a 2048 bit RSA private key
...............................+++
.........................+++
writing new private key to 'XUU-MNT.key'
-----
=
= You need to have this pin added to your mnt object before proceeding to the next step.
=
|MNT Key Pin| remarks: pin-sha256:HdqCid0sedWXX3Q0uG98rYjJyTNOzaT13WfWpr1GvIw=
````

## Sign the user key

`````
$ ./ca.dn42 user-sign XUU-MNT xuu@sour.is
== USER CERT ==
   C:XD
   O:dn42
   OU:dn42 Certificate Authority
   CN:XUU-MNT
   emailAddress:xuu@sour.is
   owner:XUU-MNT
   pin-sha256:HdqCid0sedWXX3Q0uG98rYjJyTNOzaT13WfWpr1GvIw=
OK https://ca.dn42/crt/XUU-MNT.crt
Enter Export Password:
Verifying - Enter Export Password:
````

## Generate the server key

````
$ ./ca.dn42 tls-gen ca.dn42 XUU-MNT xuu@sour.is DNS:ca.dn42

Generating a 2048 bit RSA private key
...........................................+++
.......................+++
writing new private key to 'XUU-MNT_ca.dn42.key'
-----
writing RSA key
=
= |DNS Key Pin| You need to have this pin added to your dns records  before proceeding to the next step.
=
_dn42_tlsverify.ca.dn42. IN TXT XUU-MNT:pin-sha256:Qu/X5GNqOo05TdL7oexkamE34OUuDE60T+f0xc60UPQ=
````

After you set this TXT-Record for your domain, you can verify it with the following command (by replacing the domain with your own):

````
$ dig +short TXT _dn42_tlsverify.ca.dn42.
"XUU-MNT:pin-sha256:Qu/X5GNqOo05TdL7oexkamE34OUuDE60T+f0xc60UPQ="
````

## Sign the server key

````
$ ./ca.dn42 tls-sign ca.dn42 XUU-MNT
== USER CERT ==
   C:XD 
   O:dn42 
   OU:dn42 Certificate Authority 
   CN:XUU-MNT 
   emailAddress:xuu@sour.is 
   owner:XUU-MNT 
   pin-sha256:HdqCid0sedWXX3Q0uG98rYjJyTNOzaT13WfWpr1GvIw= 
== DNS CSR ==
   C:XD 
   O:dn42 
   OU:dn42 Certificate Authority 
   CN:ca.dn42 
   emailAddress:xuu@sour.is 
   owner:XUU-MNT 
   pin-sha256:Qu/X5GNqOo05TdL7oexkamE34OUuDE60T+f0xc60UPQ= 
== DNS Tests ==
   CN Record: ca.dn42 PASSED
   SAN Record: ca.dn42 PASSED
OK https://ca.dn42/crt/XUU-MNT_ca.dn42.crt
Enter Export Password: ****
Verifying - Enter Export Password: ****
````

The generated certificate will be valid for 3 months, to renew it simply run ````./ca.dn42 tls-sign ca.dn42 XUU-MNT```` again. This could be also automated in cron:

````
0 0 1 * * /etc/ssl/dn42/ca.dn42 tls-sign wiki.dn42 MIC92-MNT
````

or with a systemd timer:

````
# update-dn42-ca.timer
[Timer]
OnBootSec=1h
OnUnitActiveSec=1w
Persistent=yes

[Install]
WantedBy=timers.target
````

````
[Service]
Type=oneshot
WorkingDirectory=/etc/ssl/dn42
ExecStart=/etc/ssl/dn42/ca.dn42 tls-sign wiki.dn42 MIC92-MNT
# accept multiple ExecStart lines for other certificates
#ExecStart=/etc/ssl/dn42/ca.dn42 tls-sign foobar.dn42 MIC92-MNT
ExecStart=/usr/bin/nginx -s reload
````

## Revoke a certificate.

````
$ ./ca.dn42 revoke XUU-MNT XUU-MNT.crt 
== USER CERT ==
   C:XD 
   O:dn42 
   OU:dn42 Certificate Authority 
   CN:XUU-MNT 
   emailAddress:xuu@sour.is 
   owner:XUU-MNT 
   pin-sha256:HdqCid0sedWXX3Q0uG98rYjJyTNOzaT13WfWpr1GvIw= 
== REVOKE CERT ==
OK
````

## Certificate transparency
All issued certificates will be logged to [xuu's mattermost instance](https://teams.dn42/dn42/channels/tls-certificates).
# SSL Certificate Authority

internal.dn42 is signed by an internally maintained CA that is only allowed to sign *.dn42 domains.
If you would like to have a certificate signed by this CA send a CSR to xuu@sour.is

The CA certificate:

```
-----BEGIN CERTIFICATE-----
MIIDhzCCAm+gAwIBAgIJALhBYKXcLej6MA0GCSqGSIb3DQEBCwUAMCgxJjAkBgNV
BAMTHURONDIgSW50ZXJuYWwgQ0EgKFVOVkVSSUZJRUQpMB4XDTE0MTIyMDE4NDAw
NVoXDTI0MTIxNzE4NDAwNVowKDEmMCQGA1UEAxMdRE40MiBJbnRlcm5hbCBDQSAo
VU5WRVJJRklFRCkwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDViXIb
VcWw+tnZCbZuy3ME4vQJsiX5ik5WkqkBaj5vk7zt+Ca8XvaM8cqppb8kEOCkC+MV
/qp5R2BAukKAAcmACQ9FHx6XYGxMQztU9tTMUuAqWH8JihWjBSoEfBQ9UpJHbgvo
7AAY382rcaLQJs3QgxtNiUjeblPlAy6AE3TUBEiNwa7MTZ7f2YHbVF/9DpvUZee6
KytOalzgbKcuFsquf4vIBtcKav1Qwmdr8eehQHdo8Nxv32uZqd272Q+EInFmzDPu
KpJdhwc/7S/+ohL/fs6RQphnJvLR572cXTzwEIkFAGqym3Fx30Q7Keoq6Cx46yez
lwL2k7C82bE4c+//AgMBAAGjgbMwgbAwHQYDVR0OBBYEFNeJoQrHPqh2SMplqb1V
ac9OWmkiMFgGA1UdIwRRME+AFNeJoQrHPqh2SMplqb1Vac9OWmkioSykKjAoMSYw
JAYDVQQDEx1ETjQyIEludGVybmFsIENBIChVTlZFUklGSUVEKYIJALhBYKXcLej6
MBIGA1UdEwEB/wQIMAYBAf8CAQAwCwYDVR0PBAQDAgEGMBQGA1UdHgQNMAugCTAH
ggUuZG40MjANBgkqhkiG9w0BAQsFAAOCAQEAMqVN55ruWA70znyWMB9+A4BcsFgI
uFVZIOnJEy72Nsz0VvfEEW/3rxKs0UnLcnfBHlx2WHdD2zUJLiTAf6ziRhXpFPXY
Ys3RJFE/8ZDVH3+dGOBekJusDX0YQcwXA/NVO2ogM6WIRIz7QabvOIJBaYXu71ZB
ci29iKFLJ4dsUG69hoeDghwkij2mCR2G/tP+xbrb7xGM73tDjuzmESYlUAVgKtlH
gfcWBU6anZMFJV9Y2lkNhxw5G7JMDSYsfONskzPet9HeHrmu67EnXMapELCjZL3O
X0KmpxYGil6Ly5xImaVqwxnm7wlDiNT6vd0cPgtKd/YynPFNw9Eh+MSamw==
-----END CERTIFICATE-----
```

Certificate fingerprint
```
$ openssl x509 -sha256 -fingerprint -noout -in dn42.crt
SHA256 Fingerprint=8C:8E:C1:12:DB:85:3E:59:CB:1A:DF:90:74:A4:0C:83:B5:ED:57:1E:BC:06:E0:0D:80:B3:47:68:11:77:E1:C9
```

## Testing constraints

The name constraints can be verified for example by using openssl:
```
    openssl x509 -in dn42.crt -text -noout
```
which will show among other things:
```
    X509v3 Name Constraints: 
      Permitted:
        DNS:.dn42
```

The following sites have been set up to demonstrate the CA failing to sign arbitrary domains:

* [badkey.sour.is](https://badkey.sour.is) - Host is in HSTS preload with key pinning. The browser should fail because the keypin does not match. 
* [badkey.xuu.me](https://badkey.xuu.me) - Hostname is outside of domain allowed list.
* [badkey.internal.dn42](https://badkey.internal.dn42) - Valid hostname and keypinned. But certificate contains bad subject alternate names.

They all use the same certificate, that should be regarded invalid by whatever software you use because of
```
        Subject: CN=badkey.internal.dn42
[...]
            X509v3 Subject Alternative Name: 
                DNS:badkey.sour.is, DNS:badkey.xuu.me, DNS:badkey.xuu.dn42, DNS: google.com, DNS:*.com, DNS:*

```

## Importing the certificate

- In archlinux you can install the package [ca-certificates-dn42](https://aur.archlinux.org/packages/ca-certificates-dn42) from AUR
- cacert have a comprehensive FAQ on how to import your own root certificates in [browsers](http://wiki.cacert.org/FAQ/BrowserClients) and [other software](http://wiki.cacert.org/FAQ/ImportRootCert)

## PKI Store

All issued keys are posted in a git repository at: https://dn42.us/git/dn42/pki/tree/
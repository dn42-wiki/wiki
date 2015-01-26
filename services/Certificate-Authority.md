# SSL Certificate Authority

internal.dn42 is signed by an internally maintained CA that is only allowed to sign *.dn42 domains or 172.22.0.0/15 ip addresses. 

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

If you would like to trust the certificate import the following:

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

If you would like to have a certificate signed by this CA send a CSR to dn42@xuu.cc
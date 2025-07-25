# SSL Certificate Authority

internal.dn42 is signed by an internally maintained CA that is only allowed to sign *.dn42 domains.
If you would like to have a certificate signed by this CA there is [an automated process to do so](/services/Automatic-CA). The CA is maintained by xuu@dn42.us.

If you are required to specify a license to clarify redistribution, then it [can be considered](https://groups.io/g/dn42/message/844) as [CC0](https://creativecommons.org/public-domain/cc0/).

The CA certificate ([dn42](https://ca.dn42/crt/root-ca.crt), [iana](https://ca.dn42.us/crt/root-ca.crt)):

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 137808117760 (0x2016010000)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=XD, O=dn42, OU=dn42 Certificate Authority, CN=dn42 Root Authority CA
        Validity
            Not Before: Jan 16 00:12:04 2016 GMT
            Not After : Dec 31 23:59:59 2030 GMT
        Subject: C=XD, O=dn42, OU=dn42 Certificate Authority, CN=dn42 Root Authority CA
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:c1:19:10:de:01:86:11:f1:82:0c:b0:d4:e5:ff:
                    9a:c8:e3:aa:f4:00:08:82:c0:cf:7f:05:7a:21:97:
                    c1:b5:8b:a3:d1:54:ee:fa:04:0f:77:d5:5c:98:4b:
                    d9:88:18:c1:17:10:92:e5:24:fa:ef:61:eb:5d:7b:
                    11:e5:be:ba:89:f2:60:c9:3b:82:05:3a:74:54:60:
                    23:66:1a:d8:cd:28:7b:f1:ea:55:25:9a:8c:04:a0:
                    ff:9d:48:54:4c:9d:bc:2d:a0:df:71:ae:64:47:0d:
                    e7:75:05:f4:c5:02:2a:d2:0c:be:a3:63:54:62:2b:
                    ad:29:eb:6a:08:a4:5e:a8:eb:f1:52:14:4e:d1:5d:
                    41:2f:d3:19:ba:e4:82:36:7a:d1:a3:f2:84:f6:07:
                    b2:f6:0c:30:db:db:76:ee:e9:14:05:c7:8f:75:b7:
                    3f:d5:d5:35:56:d0:92:44:df:26:1e:00:fa:ae:cb:
                    7a:c9:50:67:5d:69:f8:f9:fd:25:a7:1d:db:40:b1:
                    42:bc:45:57:e1:c9:1c:42:ba:69:80:1e:ea:25:99:
                    12:9f:6f:23:a3:d2:2e:4a:cd:15:e4:7c:49:f9:d1:
                    c0:f0:19:0c:15:50:ce:a6:51:bb:aa:16:b2:82:ec:
                    f4:61:44:8c:1c:dd:65:60:04:77:b0:4d:99:67:17:
                    fb:09
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Certificate Sign, CRL Sign
            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Subject Key Identifier: 
                54:76:88:B2:C0:B5:30:D0:FC:4F:C9:6D:3B:F9:8C:55:11:AC:15:15
            X509v3 Authority Key Identifier: 
                keyid:54:76:88:B2:C0:B5:30:D0:FC:4F:C9:6D:3B:F9:8C:55:11:AC:15:15

            X509v3 Name Constraints: 
                Permitted:
                  DNS:.dn42
                  IP:172.20.0.0/255.252.0.0
                  IP:FD42:0:0:0:0:0:0:0/FFFF:0:0:0:0:0:0:0

    Signature Algorithm: sha256WithRSAEncryption
         5c:a4:3b:41:a0:81:69:e2:71:99:4d:75:4b:5a:20:0d:2a:d9:
         ec:ea:bc:8d:4f:b0:6c:f3:2e:41:1a:a0:75:f3:de:7e:3a:e0:
         a7:b9:db:cd:f5:16:e4:6a:cb:e7:cc:2a:8f:ee:7f:14:0a:a5:
         b5:f9:66:48:81:e5:68:1e:0c:a6:a3:3c:a7:2b:e3:95:cf:e3:
         63:15:0d:16:09:63:d9:66:31:3b:42:2e:7c:1a:e5:28:8e:5e:
         3d:9e:28:99:48:e9:47:86:11:e2:04:29:60:2b:96:95:99:ae:
         3f:ab:ff:3f:45:ab:7e:07:45:4e:4d:0b:18:40:3d:3b:02:9c:
         4e:a9:0f:a5:c2:3f:4a:30:77:ae:66:5c:b3:8d:b2:41:6b:e2:
         98:01:7d:e0:6b:52:70:4d:3d:b8:a9:48:f5:02:d2:d9:40:66:
         b6:5e:44:25:11:55:ac:31:02:d7:67:72:6a:6a:bc:74:34:5f:
         75:dc:9a:4f:83:28:40:e0:2a:dc:3f:41:43:5a:47:07:2b:b7:
         a7:3f:d0:15:a2:42:d7:30:22:f2:f6:e4:b4:f6:3b:38:ca:6b:
         4c:e7:3c:a4:70:cb:de:af:0a:14:ff:23:25:ca:04:cd:9e:49:
         c3:4b:e4:0a:b5:0b:84:b5:ef:b4:5b:63:07:47:63:cd:5c:50:
         0b:42:0a:a9
-----BEGIN CERTIFICATE-----
MIID8DCCAtigAwIBAgIFIBYBAAAwDQYJKoZIhvcNAQELBQAwYjELMAkGA1UEBhMC
WEQxDTALBgNVBAoMBGRuNDIxIzAhBgNVBAsMGmRuNDIgQ2VydGlmaWNhdGUgQXV0
aG9yaXR5MR8wHQYDVQQDDBZkbjQyIFJvb3QgQXV0aG9yaXR5IENBMCAXDTE2MDEx
NjAwMTIwNFoYDzIwMzAxMjMxMjM1OTU5WjBiMQswCQYDVQQGEwJYRDENMAsGA1UE
CgwEZG40MjEjMCEGA1UECwwaZG40MiBDZXJ0aWZpY2F0ZSBBdXRob3JpdHkxHzAd
BgNVBAMMFmRuNDIgUm9vdCBBdXRob3JpdHkgQ0EwggEiMA0GCSqGSIb3DQEBAQUA
A4IBDwAwggEKAoIBAQDBGRDeAYYR8YIMsNTl/5rI46r0AAiCwM9/BXohl8G1i6PR
VO76BA931VyYS9mIGMEXEJLlJPrvYetdexHlvrqJ8mDJO4IFOnRUYCNmGtjNKHvx
6lUlmowEoP+dSFRMnbwtoN9xrmRHDed1BfTFAirSDL6jY1RiK60p62oIpF6o6/FS
FE7RXUEv0xm65II2etGj8oT2B7L2DDDb23bu6RQFx491tz/V1TVW0JJE3yYeAPqu
y3rJUGddafj5/SWnHdtAsUK8RVfhyRxCummAHuolmRKfbyOj0i5KzRXkfEn50cDw
GQwVUM6mUbuqFrKC7PRhRIwc3WVgBHewTZlnF/sJAgMBAAGjgaowgacwDgYDVR0P
AQH/BAQDAgEGMA8GA1UdEwEB/wQFMAMBAf8wHQYDVR0OBBYEFFR2iLLAtTDQ/E/J
bTv5jFURrBUVMB8GA1UdIwQYMBaAFFR2iLLAtTDQ/E/JbTv5jFURrBUVMEQGA1Ud
HgQ9MDugOTAHggUuZG40MjAKhwisFAAA//wAADAihyD9QgAAAAAAAAAAAAAAAAAA
//8AAAAAAAAAAAAAAAAAADANBgkqhkiG9w0BAQsFAAOCAQEAXKQ7QaCBaeJxmU11
S1ogDSrZ7Oq8jU+wbPMuQRqgdfPefjrgp7nbzfUW5GrL58wqj+5/FAqltflmSIHl
aB4MpqM8pyvjlc/jYxUNFglj2WYxO0IufBrlKI5ePZ4omUjpR4YR4gQpYCuWlZmu
P6v/P0WrfgdFTk0LGEA9OwKcTqkPpcI/SjB3rmZcs42yQWvimAF94GtScE09uKlI
9QLS2UBmtl5EJRFVrDEC12dyamq8dDRfddyaT4MoQOAq3D9BQ1pHByu3pz/QFaJC
1zAi8vbktPY7OMprTOc8pHDL3q8KFP8jJcoEzZ5Jw0vkCrULhLXvtFtjB0djzVxQ
C0IKqQ==
-----END CERTIFICATE-----
```


## Testing constraints

The name constraints can be verified for example by using openssl:
```sh
openssl x509 -in dn42.crt -text -noout
```
which will show among other things:
```
    X509v3 Name Constraints: 
      Permitted:
        DNS:.dn42
```

## Importing the certificate

- cacert have a comprehensive FAQ on how to import your own root certificates in [browsers](http://wiki.cacert.org/FAQ/BrowserClients) and [other software](http://wiki.cacert.org/FAQ/ImportRootCert)

### Archlinux

Install `ca-certificates-dn42` from [AUR](https://aur.archlinux.org/packages/ca-certificates-dn42/)

### Debian/Ubuntu

#### Unofficial Debian Package

```bash
wget https://ca.dn42.us/ca-dn42_20161122.0_all.deb
# If you're on a dn42-only network:
# wget --no-check-certificate https://ca.dn42/ca-dn42_20161122.0_all.deb
sudo dpkg -i ca-dn42_20161122.0_all.deb
sudo dpkg-reconfigure ca-certificates
```

You will be asked which certificates you would like to enabled. By default, the dn42 root certifcate (dn42/root-ca.crt) is not enabled, be sure to enable it. This package is waiting for inclusion in Debian (Debian bug [#845351](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=845351)).

#### Manual Installation

```bash
$ mkdir /usr/share/ca-certificates/extra
$ cat > /usr/share/ca-certificates/extra/dn42.crt <<EOF
-----BEGIN CERTIFICATE-----
MIID8DCCAtigAwIBAgIFIBYBAAAwDQYJKoZIhvcNAQELBQAwYjELMAkGA1UEBhMC
WEQxDTALBgNVBAoMBGRuNDIxIzAhBgNVBAsMGmRuNDIgQ2VydGlmaWNhdGUgQXV0
aG9yaXR5MR8wHQYDVQQDDBZkbjQyIFJvb3QgQXV0aG9yaXR5IENBMCAXDTE2MDEx
NjAwMTIwNFoYDzIwMzAxMjMxMjM1OTU5WjBiMQswCQYDVQQGEwJYRDENMAsGA1UE
CgwEZG40MjEjMCEGA1UECwwaZG40MiBDZXJ0aWZpY2F0ZSBBdXRob3JpdHkxHzAd
BgNVBAMMFmRuNDIgUm9vdCBBdXRob3JpdHkgQ0EwggEiMA0GCSqGSIb3DQEBAQUA
A4IBDwAwggEKAoIBAQDBGRDeAYYR8YIMsNTl/5rI46r0AAiCwM9/BXohl8G1i6PR
VO76BA931VyYS9mIGMEXEJLlJPrvYetdexHlvrqJ8mDJO4IFOnRUYCNmGtjNKHvx
6lUlmowEoP+dSFRMnbwtoN9xrmRHDed1BfTFAirSDL6jY1RiK60p62oIpF6o6/FS
FE7RXUEv0xm65II2etGj8oT2B7L2DDDb23bu6RQFx491tz/V1TVW0JJE3yYeAPqu
y3rJUGddafj5/SWnHdtAsUK8RVfhyRxCummAHuolmRKfbyOj0i5KzRXkfEn50cDw
GQwVUM6mUbuqFrKC7PRhRIwc3WVgBHewTZlnF/sJAgMBAAGjgaowgacwDgYDVR0P
AQH/BAQDAgEGMA8GA1UdEwEB/wQFMAMBAf8wHQYDVR0OBBYEFFR2iLLAtTDQ/E/J
bTv5jFURrBUVMB8GA1UdIwQYMBaAFFR2iLLAtTDQ/E/JbTv5jFURrBUVMEQGA1Ud
HgQ9MDugOTAHggUuZG40MjAKhwisFAAA//wAADAihyD9QgAAAAAAAAAAAAAAAAAA
//8AAAAAAAAAAAAAAAAAADANBgkqhkiG9w0BAQsFAAOCAQEAXKQ7QaCBaeJxmU11
S1ogDSrZ7Oq8jU+wbPMuQRqgdfPefjrgp7nbzfUW5GrL58wqj+5/FAqltflmSIHl
aB4MpqM8pyvjlc/jYxUNFglj2WYxO0IufBrlKI5ePZ4omUjpR4YR4gQpYCuWlZmu
P6v/P0WrfgdFTk0LGEA9OwKcTqkPpcI/SjB3rmZcs42yQWvimAF94GtScE09uKlI
9QLS2UBmtl5EJRFVrDEC12dyamq8dDRfddyaT4MoQOAq3D9BQ1pHByu3pz/QFaJC
1zAi8vbktPY7OMprTOc8pHDL3q8KFP8jJcoEzZ5Jw0vkCrULhLXvtFtjB0djzVxQ
C0IKqQ==
-----END CERTIFICATE-----
EOF
$ echo "extra/dn42.crt" >> /etc/ca-certificates.conf
$ update-ca-certificates
```

## PKI Store

All issued keys and crl information are posted at: <https://ca.dn42/>

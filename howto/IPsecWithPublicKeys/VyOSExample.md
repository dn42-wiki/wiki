# IPsec with public key authentication on VyOS/EdgeOS
## Setup
### Generate an RSA keypair

    ubnt@ubnt:~$ generate vpn rsa-key bits 4096 random /dev/urandom
    Generating rsa-key to /config/ipsec.d/rsa-keys/localhost.key

    Your new local RSA key has been generated
    The public portion of the key is:

    0sAQPNdF370ZEbN+kZUJQ10qnBlZujrg39ujfk20ILTjELksOIdJw/4jiU1MfpqFDKuB/XxERwJQp2POsFyV/n76jAgxIYBfFYfuaBcIH1rdNQtDhCnkmWzlueRXGEsz0Af79n8TKyQ9otzNhJ2cPE1CWCJbKqbIUN3piviLgGlItWNeya+Tl3Oj3ZfEVwr1QOvUAw32+m4L8T9jf1vqSlOTHpRpxxPWBrLEzstk0FOcZISji2JBpDOCU8Kpyyf74JM+LxsOIHwmS15b6iFZR3U9KZLqbbd0dSy/cM8P4XjrwM5UMyRDjrLqvuA/K/33BgtnxdQR3e9DJoYH3Qr8eRgSkR+jHyq06LvgHkHbMvrEjUnc3n8bg+YfR4oyJpIWsKjfIXmN1Q51KzxAPIAww+YSYUYtamSsQsspVAtMIQqR4e0r1In1qyoSn8VCPlksNMWpqYHbSjDo5HJYoSwxf2epzMtCvhenn0OuiH0xlgzziA+wBi6txksTMvJYcPJYnBVR2NIBjkWftOfmkY+rKMozViGjyd6kB7C8lqd8W7Ha5Ds2WxIY22DM3HcYH/zTp9z2xbuMOsbIgib/Y12Kh0wHyCz0lzFvs+d6CZwinyIXNKB/Vo4iiwT5luL5mGqf3pZx4zB+30GYSs/6MaELRF9BxD7tfqYCkOLXUtxyZ4Pdl2sw==

### Exchange public keys with your peer
1. Display the public key. Send the key data portion to your peer.

        ubnt@ubnt:~$ show vpn ike rsa-keys

        Local public key (/config/ipsec.d/rsa-keys/localhost.key):

        0sAQPNdF370ZEbN+kZUJQ10qnBlZujrg39ujfk20ILTjELksOIdJw/4jiU1MfpqFDKuB/XxERwJQp2POsFyV/n76jAgxIYBfFYfuaBcIH1rdNQtDhCnkmWzlueRXGEsz0Af79n8TKyQ9otzNhJ2cPE1CWCJbKqbIUN3piviLgGlItWNeya+Tl3Oj3ZfEVwr1QOvUAw32+m4L8T9jf1vqSlOTHpRpxxPWBrLEzstk0FOcZISji2JBpDOCU8Kpyyf74JM+LxsOIHwmS15b6iFZR3U9KZLqbbd0dSy/cM8P4XjrwM5UMyRDjrLqvuA/K/33BgtnxdQR3e9DJoYH3Qr8eRgSkR+jHyq06LvgHkHbMvrEjUnc3n8bg+YfR4oyJpIWsKjfIXmN1Q51KzxAPIAww+YSYUYtamSsQsspVAtMIQqR4e0r1In1qyoSn8VCPlksNMWpqYHbSjDo5HJYoSwxf2epzMtCvhenn0OuiH0xlgzziA+wBi6txksTMvJYcPJYnBVR2NIBjkWftOfmkY+rKMozViGjyd6kB7C8lqd8W7Ha5Ds2WxIY22DM3HcYH/zTp9z2xbuMOsbIgib/Y12Kh0wHyCz0lzFvs+d6CZwinyIXNKB/Vo4iiwT5luL5mGqf3pZx4zB+30GYSs/6MaELRF9BxD7tfqYCkOLXUtxyZ4Pdl2sw==

2. Convert your peer's public key to the Base64 RFC 3110 format using the [pubkey-converter][pubkey-converter] script, if necessary.

[pubkey-converter]: <https://dn42.us/git/user/ryan/pubkey-converter.git/plain/pubkey-converter.pl> "Public key conversion script"

## Configuration
### Configure the phase 1 IKE parameters
In this example, we'll use the following settings:

| Key           | Value         |
| :------------ | :------------ |
| Encryption    | AES-128       |
| Hash          | HMAC-SHA1     |
| DH Group      | 5 (modp1536)  |
| Lifetime      | 28800 seconds |
| Peer address  | 192.0.2.2     |
| Local address | 192.0.2.1     |

1. Add your peer's public key

        vyos@vyos:~$ configure
        [edit]
        vyos@vyos# set vpn rsa-keys rsa-key-name my-peer rsa-key 0sAwEAAb4ETtKRLxcFNty56regsR61pq7hQl3NnjwABL16wZXGynKxZlj11VbdqcNwaTaqHZLV4Xfy867nImSs0DD9Cko5LzWwyM1Ih4SB+rIjfmBt7nRUrilnYvfWAONG1CLTI2tXnM/miNqiY+PxlCiMPr1KrTJWBWOknqqhhL2dOBfp3Ryx1yRxDACFG4wgpwmndJOnmefnV6qZXWiOdoIsBsBqQKiDY0g2uI+S3KxK27JL3KZWcA2ehhvtxmq4vwcMXplYeedei3EEmWxtddAZCApXor9bkVoVp2io+a0D1ALevYMD5SIygu55Q888n5puYNry/cUjX20/F/YK+J9u2UExWewN4AIt/jMNm7nJNWpuFHfLX1V/igHrdGzoEM0E/i+nGz9CWTVTLoFUmkTjpt31FPmomSVEI7MbNXG7cpa+X55PWd1apheR52XJZPZfCnMf1DjilYbLMRG05RK8zI3QlX3UXHira0dq4OBZ+Aow+dGp+jLmwjgdBDnkQdVu0iP6bp+5/oz6mWvDQ65EVECAIXKR5zIsiKn9ZU18H+lp4xWMjiSw3Y+87Y5KeQPmX73Ygolow6VvtCBvX8CS4Plszn3i0Qp8184eLEWIY314Z8Z+HwBAjUv3MkqI93leokAjMbt23ttaJbWlWgG47BAJOEcWlMFkDNcZtOngUrzF

2. Configure an ISAKMP policy

        [edit]
        vyos@vyos# edit vpn ipsec ike-group FOO
        [edit vpn ipsec ike-group FOO]
        vyos@vyos# set lifetime 28800
        [edit vpn ipsec ike-group FOO]
        vyos@vyos# set proposal 1 encryption aes128
        [edit vpn ipsec ike-group FOO]
        vyos@vyos# set proposal 1 hash sha1
        [edit vpn ipsec ike-group FOO]
        vyos@vyos# set proposal 1 dh-group 5
    	[edit vpn ipsec ike-group FOO]
    	vyos@vyos# commit

3. Set your peer definition to use the public key

        [edit vpn ipsec ike-group FOO]
        vyos@vyos# up
        [edit vpn ipsec]
        vyos@vyos# edit site-to-site peer 192.0.2.2
        [edit vpn ipsec site-to-site peer 192.0.2.2]
        vyos@vyos# set authentication mode rsa
        [edit vpn ipsec site-to-site peer 192.0.2.2]
        vyos@vyos# set authentication rsa-key-name my-peer

4. All done! Configure the phase 2 parameters as you otherwise would.

## Full GRE/IPsec example
    interfaces {
        ethernet eth0 {
            address 192.0.2.1/30
            description WAN
            duplex auto
            speed auto
        }
        tunnel tun0 {
            address 10.1.2.0/31
            encapsulation gre
            local-ip 192.0.2.1
            mtu 1400
            multicast disable
            remote-ip 192.0.2.2
            ttl 255
        }
    }
    vpn {
        ipsec {
            esp-group BAR {
                compression disable
                lifetime 3600
                mode transport
                pfs dh-group5
                proposal 1 {
                    encryption aes128
                    hash sha1
                }
            }
            ike-group FOO {
                lifetime 28800
                proposal 1 {
                    dh-group 5
                    encryption aes128
                    hash sha1
                }
            }
            ipsec-interfaces {
                interface eth0
            }
            site-to-site {
                peer 192.0.2.2 {
                    authentication {
                        mode rsa
                        rsa-key-name my-peer
                    }
                    connection-type initiate
                    default-esp-group BAR
                    ike-group FOO
                    local-ip 192.0.2.1
                    tunnel 0 {
                        protocol gre
                    }
                }
            }
        }
        rsa-keys {
            rsa-key-name my-peer {
                rsa-key 0sAwEAAbsbRoUcgdm4A4Nm+PLxWcW+zFis7pkaJ0MkGVzM7VC8nmngkM+W2zqZyQ4NUTBKKfGOUc4Ogi6gyhlzUnHdag9tDERIX+BwlDO6G4arod9z9KqmJuX4AOYVjH5QlAPz7NDMAezVekGoVLPGdOAMPD6NN54ihLRH6V3if8AGoJRpiajhcgQipjeQnhH4QhsYK4XSjayGT1onQwA8nhy5kt4ofyqSale4Fl4166S9tCn4RKwtlJDjR6VIrg6op6Ip8+ke2vjEHPJHj6qVsxfRgOk2d8pY8oPVt8ayc5F1z+lqJ7R0fADfN+AQSaBqOMmg5dHDFYWwgYkU5egdVKS7Oko6uNuUWsZ0VEnRoPZ4syJEUbiF5wGfaVBaaVLZYUlRLQCffB4JKzp+JesVToCX6JYRfb4JYQWFCDeQfrqRZHM4r13h8MOWPn9cqXcP47RKJjzNp6595biUotmCbMHyy/uveMWxK6vDzPQRkywqMMJE2qOyACmbMnSce9KlYhvma82Vd+z/9/U9NEy0s5MaYNDn+q+KYT5My3NSv52F6sLVGrKxTk79tzUejZcoukJv+gf51Epam4kVHzPIal/khsfjZn6YCU2j5+qcdRmzF+SG5c2WicvEU2Gc4ratfYNEPxU5oArzHIhIz6x2nAF+szcx/x8GEyXPNHnxEboJB7ox
            }
        }
    }

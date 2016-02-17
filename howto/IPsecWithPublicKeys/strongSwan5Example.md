# IPsec with public key authentication on strongSwan >= 5.0.0
## Setup
### Generate an RSA keypair

    root@debian:~# mkdir /etc/ipsec.d/public
    root@debian:~# ipsec pki --gen --type rsa --outform pem --size 4096 > /etc/ipsec.d/private/mykey.pem
    root@debian:~# ipsec pki --pub --in /etc/ipsec.d/private/mykey.pem --outform pem > /etc/ipsec.d/public/mykey.pem
    root@debian:~# echo ": RSA mykey.pem" >> /etc/ipsec.secrets

### Exchange public keys with your peer
1. Display the public key. Send the key data to your peer.

        root@debian:~# more /etc/ipsec.d/public/mykey.pub
        -----BEGIN PUBLIC KEY-----
        MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEA2/FWIJuVUtfsLovavNp+
        nPSsT2mQoNK3ZUUwuEKfBjT7mhijdXRHh1SAtIaU2aen5+d5q6e27vMCCYOQLagn
        9CkKatBq54zGNvDSzQEpz0mIsaBx9xjvhsgqAmKCTpLtKuMz6cZbH8y8o9/ZZ8Kv
        +Jht67T8BDKXczgOg5IIaX84UpCrlSgmnSvKYKu3PXnt91bZ66HaDZJjPf9aiMNc
        fvuUqVfFWnsV2zI6HFvG/uwkqLalsnPaAwVeIWl2Ovy2Jzdj0GRLSYx87eneSBo+
        7tjlURQTudAj1+53SFOkBcCPSnzPYpIC3hBfZ8Zw8r/25moW3xf8TlLLJqgAh50Y
        tVyvyVSv1MKYBdjZcFsEXUceC5LI9JZryB/Serq0R+4//ZiR3LEtetVKNvco9bcI
        JHXr88HM2XeYRfRPAB6wembIEMKYdwIhwYAPPAtL+lDHtZBiBAIAp0y0FhaozSzl
        MSry8tbJR2fD/i8/yXr5isVfjJZdw8WK0LAd8a8zvmNIFKgiKWjoDgIycM5HrRD+
        rY0Br9xONkNdgB7Lz/wPEyUsiIiZpawM/S4taX7ExK4Wi3pdxkOHLn2ZyaWKsdhX
        PpCkdMfSOJ0SqCUcVze+xD8GlInUQsPgbDGvxT73jT6Ie+wSA94Cgs3mq7FS6cNo
        ZAASv7cT9DG+xfQmjrJC9SUCAwEAAQ==
        -----END PUBLIC KEY-----

2. Convert your peer's public key to the PEM format using the [pubkey-converter][pubkey-converter] script, if necessary.

[pubkey-converter]: https://dn42.us/git/user/ryan/pubkey-converter.git/plain/pubkey-converter.pl "Public key conversion script"

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

        root@debian:~# cat << EOF > /etc/ipsec.d/public/peerkey.pub
        -----BEGIN PUBLIC KEY-----
        MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAuQ1hX3+AEiLis4p5jvmY
        IfEgaq9488GU2nkuR1gZK4/CphrccmztgADU/TkiE5IOOo7zKPparcl8dZwJfX+j
        9oIEfvIHqWbF8rIkmJDAn5ZQANDsVKSVvu8nqNZhAslBkFUw1Il6KJrdZEwV4ILL
        jZmPjN/TIpzPStufeKLfUr7OodFnJXdvWGW1OwRVR8YPZP+13S2nbaqohJZYulTz
        EcEaWFhXRKzm1dxXqt2H0S93mv5CT0JzVUAv35lB39NMqcRwrg5Fsw0z448nlenS
        pIDnP8AIPm5shNf9kZJCksOE9GfPMz5KZDOoR3rWPvvapY/Zs+SBz8drPwHrZahG
        KjxV2Txl7XYkFEWvre+rpiWSidIsnzizEBQ1ea6Inwd6Y29uzmQLhus1OPfhnCIk
        AVycK+stCZObPl6abomk/avaT+FwnjRgSZzkotlbA2IHgEjF26C4a3ZUGBwDZ+7r
        U2A7DKbHRN84f62BcMyiZpavXPvk4AUQD5+lRItbFDDeZYtoIdHb9gmMcNHWsGL8
        YVxF2A8IAvr7Q1TQfwm2WPAWvjai2rbl6w8ZYiQBPUwkFtUATXU7XyGcLmjCGJGg
        HZoAqPHddX1z7HosFId+SSc7ugCnUNtwSYHq+DczBVdTa65aPv758zxeKzUqJBKy
        mP4HkvHlEmXHP2oAQ4G6PTkCAwEAAQ==
        -----END PUBLIC KEY-----
        EOF

2. Configure a connection policy in ipsec.conf for your peer

        root@debian:~# cat << EOF >> /etc/ipsec.conf
        conn MYPEER
            # peer IPs
            left=192.0.2.1
            right=192.0.2.2
            # phase 1 parameters
            ike=aes128-sha1-modp1536!
            ikelifetime=28800s
            # authentication
            authby=pubkey
            leftrsasigkey=/etc/ipsec.d/public/mykey.pub
            rightrsasigkey=/etc/ipsec.d/public/peerkey.pub
        EOF

3. All done! Configure the phase 2 parameters as you otherwise would.

## Full GRE/IPsec example
    root@debian:~# ip addr show dev gre1
    11: gre1@NONE: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1400 qdisc noqueue state UNKNOWN
        link/gre 192.0.2.1 peer 192.0.2.2
        inet 10.1.2.0/31 scope global gre1
           valid_lft forever preferred_lft forever
        inet6 fe80::200:5efe:6825:1c22/64 scope link
           valid_lft forever preferred_lft forever
    root@debian:~# more /etc/ipsec.conf
    # ipsec.conf - strongSwan IPsec configuration file
    
    config setup
    
    conn %default
        keyexchange=ikev1
        dpdaction=restart
    
    conn MYPEER
        # peer IPs
        left=192.0.2.1
        right=192.0.2.2
        # phase 1 parameters
        ike=aes128-sha1-modp1536!
        ikelifetime=28800s
        # authentication
        authby=pubkey
        leftrsasigkey=/etc/ipsec.d/public/mykey.pub
        rightrsasigkey=/etc/ipsec.d/public/peerkey.pub
        # phase 2 parameters
        esp=aes128-sha1-modp1536!
        lifetime=3600s
        type=transport
        leftprotoport=gre
        rightprotoport=gre
        # startup
        auto=route
        keyingtries=%forever

If your peer is using a Cisco router and is behind NAT, then you might need to add the following option:

        rightid=NATIP
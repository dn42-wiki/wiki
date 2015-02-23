# IPsec with public key authentication on strongSwan < 5.0.0
## Setup
### Generate an RSA keypair

    root@debian:~# mkdir /etc/ipsec.d/public
    root@debian:~# ipsec pki --gen --type rsa --outform pem --size 4096 > /etc/ipsec.d/private/mykey.pem
    root@debian:~# ipsec pki --pub --in /etc/ipsec.d/private/mykey.pem --outform pem > /etc/ipsec.d/public/mykey.pub
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

2. Convert your peer's public key to the Base64 RFC 3110 format using the [pubkey-converter][pubkey-converter] script, if necessary.

[pubkey-converter]: https://git.sour.is/git/user/ryan/pubkey-converter.git/plain/pubkey-converter.pl "Public key conversion script"

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

_Note: strongSwan < 5.0.0 will read PEM-formatted **private** keys, but requires public keys to be added to the configuration in Base64 RFC 3110 format. Convert your host's public key from PEM to Base64 RFC 3110 for inclusion in ipsec.conf as described below._

1. Configure a connection policy in ipsec.conf for your peer. The `leftrsasigkey` attribute is your host's public key in Base64 RFC 3110 format enclosed in double quotes, and `rightrsasigkey` is your peer's key.

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
            leftrsasigkey="0sAwEAAdvxViCblVLX7C6L2rzafpz0rE9pkKDSt2VFMLhCnwY0+5oYo3V0R4dUgLSGlNmnp+fneauntu7zAgmDkC2oJ/QpCmrQaueMxjbw0s0BKc9JiLGgcfcY74bIKgJigk6S7SrjM+nGWx/MvKPf2WfCr/iYbeu0/AQyl3M4DoOSCGl/OFKQq5UoJp0rymCrtz157fdW2euh2g2SYz3/WojDXH77lKlXxVp7FdsyOhxbxv7sJKi2pbJz2gMFXiFpdjr8tic3Y9BkS0mMfO3p3kgaPu7Y5VEUE7nQI9fud0hTpAXAj0p8z2KSAt4QX2fGcPK/9uZqFt8X/E5SyyaoAIedGLVcr8lUr9TCmAXY2XBbBF1HHguSyPSWa8gf0nq6tEfuP/2YkdyxLXrVSjb3KPW3CCR16/PBzNl3mEX0TwAesHpmyBDCmHcCIcGADzwLS/pQx7WQYgQCAKdMtBYWqM0s5TEq8vLWyUdnw/4vP8l6+YrFX4yWXcPFitCwHfGvM75jSBSoIilo6A4CMnDOR60Q/q2NAa/cTjZDXYAey8/8DxMlLIiImaWsDP0uLWl+xMSuFot6XcZDhy59mcmlirHYVz6QpHTH0jidEqglHFc3vsQ/BpSJ1ELD4Gwxr8U+940+iHvsEgPeAoLN5quxUunDaGQAEr+3E/QxvsX0Jo6yQvUl"
            rightrsasigkey="0sAwEAAbkNYV9/gBIi4rOKeY75mCHxIGqvePPBlNp5LkdYGSuPwqYa3HJs7YAA1P05IhOSDjqO8yj6Wq3JfHWcCX1/o/aCBH7yB6lmxfKyJJiQwJ+WUADQ7FSklb7vJ6jWYQLJQZBVMNSJeiia3WRMFeCCy42Zj4zf0yKcz0rbn3ii31K+zqHRZyV3b1hltTsEVUfGD2T/td0tp22qqISWWLpU8xHBGlhYV0Ss5tXcV6rdh9Evd5r+Qk9Cc1VAL9+ZQd/TTKnEcK4ORbMNM+OPJ5Xp0qSA5z/ACD5ubITX/ZGSQpLDhPRnzzM+SmQzqEd61j772qWP2bPkgc/Haz8B62WoRio8Vdk8Ze12JBRFr63vq6YlkonSLJ84sxAUNXmuiJ8HemNvbs5kC4brNTj34ZwiJAFcnCvrLQmTmz5emm6JpP2r2k/hcJ40YEmc5KLZWwNiB4BIxduguGt2VBgcA2fu61NgOwymx0TfOH+tgXDMomaWr1z75OAFEA+fpUSLWxQw3mWLaCHR2/YJjHDR1rBi/GFcRdgPCAL6+0NU0H8JtljwFr42otq25esPGWIkAT1MJBbVAE11O18hnC5owhiRoB2aAKjx3XV9c+x6LBSHfkknO7oAp1DbcEmB6vg3MwVXU2uuWj7++fM8Xis1KiQSspj+B5Lx5RJlxz9qAEOBuj05"
        EOF

2. All done! Configure the phase 2 parameters as you otherwise would.

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
        leftrsasigkey="0sAwEAAdvxViCblVLX7C6L2rzafpz0rE9pkKDSt2VFMLhCnwY0+5oYo3V0R4dUgLSGlNmnp+fneauntu7zAgmDkC2oJ/QpCmrQaueMxjbw0s0BKc9JiLGgcfcY74bIKgJigk6S7SrjM+nGWx/MvKPf2WfCr/iYbeu0/AQyl3M4DoOSCGl/OFKQq5UoJp0rymCrtz157fdW2euh2g2SYz3/WojDXH77lKlXxVp7FdsyOhxbxv7sJKi2pbJz2gMFXiFpdjr8tic3Y9BkS0mMfO3p3kgaPu7Y5VEUE7nQI9fud0hTpAXAj0p8z2KSAt4QX2fGcPK/9uZqFt8X/E5SyyaoAIedGLVcr8lUr9TCmAXY2XBbBF1HHguSyPSWa8gf0nq6tEfuP/2YkdyxLXrVSjb3KPW3CCR16/PBzNl3mEX0TwAesHpmyBDCmHcCIcGADzwLS/pQx7WQYgQCAKdMtBYWqM0s5TEq8vLWyUdnw/4vP8l6+YrFX4yWXcPFitCwHfGvM75jSBSoIilo6A4CMnDOR60Q/q2NAa/cTjZDXYAey8/8DxMlLIiImaWsDP0uLWl+xMSuFot6XcZDhy59mcmlirHYVz6QpHTH0jidEqglHFc3vsQ/BpSJ1ELD4Gwxr8U+940+iHvsEgPeAoLN5quxUunDaGQAEr+3E/QxvsX0Jo6yQvUl"
        rightrsasigkey="0sAwEAAbkNYV9/gBIi4rOKeY75mCHxIGqvePPBlNp5LkdYGSuPwqYa3HJs7YAA1P05IhOSDjqO8yj6Wq3JfHWcCX1/o/aCBH7yB6lmxfKyJJiQwJ+WUADQ7FSklb7vJ6jWYQLJQZBVMNSJeiia3WRMFeCCy42Zj4zf0yKcz0rbn3ii31K+zqHRZyV3b1hltTsEVUfGD2T/td0tp22qqISWWLpU8xHBGlhYV0Ss5tXcV6rdh9Evd5r+Qk9Cc1VAL9+ZQd/TTKnEcK4ORbMNM+OPJ5Xp0qSA5z/ACD5ubITX/ZGSQpLDhPRnzzM+SmQzqEd61j772qWP2bPkgc/Haz8B62WoRio8Vdk8Ze12JBRFr63vq6YlkonSLJ84sxAUNXmuiJ8HemNvbs5kC4brNTj34ZwiJAFcnCvrLQmTmz5emm6JpP2r2k/hcJ40YEmc5KLZWwNiB4BIxduguGt2VBgcA2fu61NgOwymx0TfOH+tgXDMomaWr1z75OAFEA+fpUSLWxQw3mWLaCHR2/YJjHDR1rBi/GFcRdgPCAL6+0NU0H8JtljwFr42otq25esPGWIkAT1MJBbVAE11O18hnC5owhiRoB2aAKjx3XV9c+x6LBSHfkknO7oAp1DbcEmB6vg3MwVXU2uuWj7++fM8Xis1KiQSspj+B5Lx5RJlxz9qAEOBuj05"
        # phase 2 parameters
        esp=aes128-sha1!
        pfs=yes
        pfsgroup=modp1536
        lifetime=3600s
        type=transport
        leftprotoport=gre
        rightprotoport=gre
        # startup
        auto=route
        keyingtries=%forever
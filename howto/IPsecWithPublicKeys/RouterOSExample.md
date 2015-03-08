# IPsec with public key authentication on Mikrotik RouterOS
## Setup
### Generate an RSA keypair

    [admin@mtk1] > /ip ipsec key
    [admin@mtk1] /ip ipsec key> generate-key mykey key-size=4096
    For key bigger than 1024bit this may take a while..
    [admin@mtk1] /ip ipsec key> print
    Flags: P - private-key, R - rsa
     #    NAME                                                             KEY-SIZE
     0 PR mykey                                                            4096-bit

### Exchange public keys with your peer
1. Export the public key to a file.

        [admin@mtk1] /ip ipsec key> export-pub-key mykey file-name=mykey.pub
        
        [admin@mtk1] /ip ipsec key> /file print where name=mykey.pub
         # NAME                   TYPE                        SIZE CREATION-TIME
         2 mykey.pub              ssh key                      451 jul/20/2014 12:35:33

2. Copy the file to your workstation and send it to your peer. The contents of the file should look like this:

        -----BEGIN PUBLIC KEY-----
        MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAv4RHohMZP4F5qTJKqoSL
        TqefoZZRt1RVI5dOocjV1pJZnqcXMtHfQ/5+O+igUCAX+yBv0hie+U32FWcy5cQO
        +xaohZW1zFzvlRWVqOpTwdk/993Zmy070T1FzK4kFShsNtxYrtYNheCnakgfXgMg
        23w/35zcof64/ewzF6RuqkTzmccIFCWDuv2IobXTOYAk7G3PGN4xWscvFIroIy5s
        4E8oOmKWVoFErQA6XetJzI+X+knzI3J/6/Pff4Tz7TLxu1m2I0InFaBv1G0+BXnh
        QOvIM7fvs5s0YWaUdT+vz8F0SHtb6Q/IdWc4JJPH/Q2t4HKTkk7FUnvvub2GxVbs
        8QIDAQAB
        -----END PUBLIC KEY-----

3. Convert your peer's public key to the PEM format using the [pubkey-converter][pubkey-converter] script, if necessary.

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

1. Copy your peer's PEM-encoded public key to the router and import it. (Hit enter when it asks for a passphrase)

        [admin@mtk1] /ip ipsec key> import peer-key.pub name=peer-key
        passphrase:
        
        [admin@mtk1] /ip ipsec key> print
        Flags: P - private-key, R - rsa
         #    NAME                                                             KEY-SIZE
         0 PR mykey                                                            4096-bit
         1  R peer-key                                                         4096-bit

2. Configure your peer definition to use the public key

        [admin@mtk1] /ip ipsec peer> add address=192.0.2.2 local-address=192.0.2.1 enc-algorithm=aes-128 hash-algorithm=sha1 dh-group=modp1536 lifetime=28800 key=mykey remote-key=peer-key auth-method=rsa-key
        [admin@mtk1] /ip ipsec peer> print
        Flags: X - disabled
         0   address=192.0.2.2/32 local-address=192.0.2.1 passive=no port=500
             auth-method=rsa-key key=mykey remote-key=peer-key generate-policy=no
             exchange-mode=main send-initial-contact=yes nat-traversal=no
             proposal-check=obey hash-algorithm=sha1 enc-algorithm=aes-128
             dh-group=modp1536 lifetime=8h lifebytes=0 dpd-interval=2m
             dpd-maximum-failures=5

3. All done! Configure the phase 2 parameters as you otherwise would.

## Full GRE/IPsec example
    # jul/20/2014 13:00:04 by RouterOS 6.15
    # software id = HBCA-0B2J
    #
    /interface gre
    add dscp=inherit local-address=192.0.2.1 mtu=1400 name=gre-tunnel1 \
        remote-address=192.0.2.2
    /ip address
    add address=10.1.2.0/31 interface=gre-tunnel1 network=10.1.2.0
    /ip ipsec proposal
    set [ find default=yes ] lifetime=1h pfs-group=modp1536
    /ip ipsec peer
    add address=192.0.2.2/32 auth-method=rsa-key dh-group=modp1536 key=mykey \
        lifetime=8h local-address=192.0.2.1 remote-key=peer-key
    /ip ipsec policy
    add dst-address=192.0.2.2/32 protocol=gre sa-dst-address=192.0.2.2 \
        sa-src-address=192.0.2.1 src-address=192.0.2.1/32
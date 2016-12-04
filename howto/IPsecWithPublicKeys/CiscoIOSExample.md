# IPsec with public key authentication on Cisco IOS
## Setup
### Generate an RSA keypair
_Note: You may already have completed this step, since it's required to enable SSH._

1. Configure a hostname and domain name.

        Router#conf t
        Router(config)#hostname foo
        foo(config)#ip domain-name bar

2. Generate an RSA key. The maximum length was increased from 2048 to 4096 as of release 15.1(1)T

        foo(config)#crypto key generate rsa general-keys modulus 2048
        % The key modulus size is 2048 bits
        % Generating 2048 bit RSA keys, keys will be non-exportable...
        foo(config)#exit

### Exchange public keys with your peer
1. Display the public key. Send the key data portion to your peer.

         foo#show crypto key mypubkey rsa foo.bar
         % Key pair was generated at: 19:24:02 UTC Jul 19 2014
         Key name: foo.bar
         Storage Device: not specified
         Usage: General Purpose Key
         Key is not exportable.
         Key Data:
          30820122 300D0609 2A864886 F70D0101 01050003 82010F00 3082010A 02820101
          00ABF25E 090CBDFC 47B3763B 01E38993 584F1D47 49DEE0FC 6A766D95 F416C5A8
          83E16EF2 19C00BC9 64B3E351 D6F43E57 461AC689 912C22FE C4BE10EE 05750F27
          FEBB9C8C 2DFC7DD7 C0D1E8B2 7F022F54 04101205 60E47D99 2307E625 404F1130
          CBD1759B BBDBBF89 0C0F6B09 52E50A81 BFCC6AA6 96AFF612 B700AEA5 0EDFCDDB
          D3C7E014 2A59CD82 29A403CA 01EE580A CC4A3A2C C36369FE D2FA0FEF 2DC32D50
          1C55A296 3CBD6AAC 6AA66C73 FAB30A12 CFD1341D C261E013 8A7DA310 8D0E6C99
          C248D554 D0D68508 3EA53F0F 971DA7A6 203CA186 A79F9D93 0D2E54EF F7E311B2
          F7A8B486 D980661D DEB6C0B3 80A82583 4936F131 57C6D204 0AA5ED7F 7749F044
          8F020301 0001

2. Convert your peer's public key to the hexadecimal DER format using the [pubkey-converter][pubkey-converter] script, if necessary.

[pubkey-converter]: https://git.dn42.us/ryan/pubkey-converter/raw/master/pubkey-converter.pl "Public key conversion script"

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

        foo#conf t
        Enter configuration commands, one per line.  End with CNTL/Z.
        foo(config)#crypto key pubkey-chain rsa
        foo(config-pubkey-chain)#addressed-key 192.0.2.2
        foo(config-pubkey-key)#key-string
        Enter a public key as a hexidecimal number ....
        
        foo(config-pubkey)#30820122 300D0609 2A864886 F70D0101 01050003 82010F00 3082010A 02820101
        foo(config-pubkey)#00F3E0AA 8924E512 C08BA87C 73820A15 E5180DDC EF827221 2B3864BF B2D2A5E0
        foo(config-pubkey)#33D04C1D 43A0CAF8 617EEEBA 7DB5BD38 429660CC 3144618E 4F386201 52483DA7
        foo(config-pubkey)#FDDF7AAC DCA8C19D FF5B1956 14F63831 ACE70D0F 4C557CE7 220C7E8B 9A2C837F
        foo(config-pubkey)#065A2B41 23B68074 C57F6F78 2F222DA1 915A095C CED77FF3 F1DF849C 3FD086EA
        foo(config-pubkey)#0A74901D 99DA97D5 0CFB22E7 C6C827E6 E53BE215 8D4928D2 458B02E2 1600F1F1
        foo(config-pubkey)#F1128D63 3A55EC4B 54E6A8E1 0B197E1E 7DA31CC2 54C30A2D 03BE5B8A 16A5E324
        foo(config-pubkey)#F3B15B67 C3FB2831 7F31610A 3BD59E74 E749DC25 74424F3F 7EC305AD 0BFA5008
        foo(config-pubkey)#E36C6E00 854433B6 A0E8DBF1 7A4741DD A91A5320 1D5150FA 28F12273 56A3E9A2
        foo(config-pubkey)#D5020301 0001
        foo(config-pubkey)#quit
        foo(config-pubkey-key)#exit
        foo(config-pubkey-chain)#exit

2. Configure an ISAKMP policy

        foo(config)#crypto isakmp policy 10
        foo(config-isakmp)#encryption aes
        foo(config-isakmp)#hash sha
        foo(config-isakmp)#group 5
        foo(config-isakmp)#lifetime 28800
        foo(config-isakmp)#authentication rsa-sig
        foo(config-isakmp)#exit

3. All done! Configure the phase 2 parameters as you otherwise would.

## Full GRE/IPsec example
    crypto key pubkey-chain rsa
     addressed-key 192.0.2.2
      address 192.0.2.2
      key-string
       30820122 300D0609 2A864886 F70D0101 01050003 82010F00 3082010A 02820101
       00F3E0AA 8924E512 C08BA87C 73820A15 E5180DDC EF827221 2B3864BF B2D2A5E0
       33D04C1D 43A0CAF8 617EEEBA 7DB5BD38 429660CC 3144618E 4F386201 52483DA7
       FDDF7AAC DCA8C19D FF5B1956 14F63831 ACE70D0F 4C557CE7 220C7E8B 9A2C837F
       065A2B41 23B68074 C57F6F78 2F222DA1 915A095C CED77FF3 F1DF849C 3FD086EA
       0A74901D 99DA97D5 0CFB22E7 C6C827E6 E53BE215 8D4928D2 458B02E2 1600F1F1
       F1128D63 3A55EC4B 54E6A8E1 0B197E1E 7DA31CC2 54C30A2D 03BE5B8A 16A5E324
       F3B15B67 C3FB2831 7F31610A 3BD59E74 E749DC25 74424F3F 7EC305AD 0BFA5008
       E36C6E00 854433B6 A0E8DBF1 7A4741DD A91A5320 1D5150FA 28F12273 56A3E9A2
       D5020301 0001
      quit
    !
    crypto isakmp policy 10
     encr aes
     group 5
     lifetime 28800
    !
    crypto ipsec transform-set tset esp-aes esp-sha-hmac
     mode transport
    !
    crypto ipsec profile FOO
     set transform-set tset
     set pfs group5
    !
    interface Tunnel0
     ip address 10.1.2.0 255.255.255.254
     ip mtu 1400
     tunnel source 192.0.2.1
     tunnel destination 192.0.2.2
     tunnel protection ipsec profile FOO
    !
    interface FastEthernet0/0
     description WAN
     ip address 192.0.2.1 255.255.255.0
     duplex full
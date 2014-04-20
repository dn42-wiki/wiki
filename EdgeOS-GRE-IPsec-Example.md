# EdgeOS GRE/IPsec config example
This is an example configuration, created on EdgeOS version 1.5.0alpha1, for the Ubiquiti EdgeRouter Lite derived from the config used on a peering router in AS64746.

## Features
* Zone-based firewall
* BGP prefix filtering and route summarization
* GRE/IPsec tunnel in transport mode with "plainrsa" public key authentication
* TCP MSS clamping to avoid fragmentation

## Setup
This configuration assumes that both peers have static public IPs.

You'll need to generate a public/private keypair for your router if you intend to use "plainrsa" authentication for your IPsec connections. The local public key listed in the output is what you'll send to your peer.

    ryan@edge1:~$ generate vpn rsa-key bits 4096
    ryan@edge1:~$ show vpn ike rsa-keys

    Local public key (/config/ipsec.d/rsa-keys/localhost.key):

    0sAQPNdF370ZEbN+kZUJQ10qnBlZujrg39ujfk20ILTjELksOIdJw/4jiU1MfpqFDKuB/XxERwJQp2POsFyV/n76jAgxIYBfFYfuaBcIH1rdNQtDhCnkmWzlueRXGEsz0Af79n8TKyQ9otzNhJ2cPE1CWCJbKqbIUN3piviLgGlItWNeya+Tl3Oj3ZfEVwr1QOvUAw32+m4L8T9jf1vqSlOTHpRpxxPWBrLEzstk0FOcZISji2JBpDOCU8Kpyyf74JM+LxsOIHwmS15b6iFZR3U9KZLqbbd0dSy/cM8P4XjrwM5UMyRDjrLqvuA/K/33BgtnxdQR3e9DJoYH3Qr8eRgSkR+jHyq06LvgHkHbMvrEjUnc3n8bg+YfR4oyJpIWsKjfIXmN1Q51KzxAPIAww+YSYUYtamSsQsspVAtMIQqR4e0r1In1qyoSn8VCPlksNMWpqYHbSjDo5HJYoSwxf2epzMtCvhenn0OuiH0xlgzziA+wBi6txksTMvJYcPJYnBVR2NIBjkWftOfmkY+rKMozViGjyd6kB7C8lqd8W7Ha5Ds2WxIY22DM3HcYH/zTp9z2xbuMOsbIgib/Y12Kh0wHyCz0lzFvs+d6CZwinyIXNKB/Vo4iiwT5luL5mGqf3pZx4zB+30GYSs/6MaELRF9BxD7tfqYCkOLXUtxyZ4Pdl2sw==
If your peer sends you a key in PEM format (starts with `-----BEGIN PUBLIC KEY-----`), you'll need to convert it to the format used by EdgeOS (begins with `0s`) in order to insert it into the configuration. See [this forum post](http://community.ubnt.com/t5/EdgeMAX/ERL-lt-gt-Mikrotik-IPsec-Connections/m-p/534682#M13015) for a script to convert between the two key formats.

## Configuration

    firewall {
        all-ping enable
        broadcast-ping disable
        ipv6-receive-redirects disable
        ipv6-src-route disable
        ip-src-route disable
        log-martians enable
        name DN42-to-Local {
            default-action reject
            rule 10 {
                action accept
                description Established/Related
                state {
                    established enable
                    related enable
                }
            }
            rule 20 {
                action accept
                description ICMP
                protocol icmp
            }
            rule 30 {
                action accept
                description BGP
                destination {
                    port bgp
                }
                protocol tcp
                state {
                    new enable
                }
                tcp {
                    flags SYN,!ACK,!FIN,!RST
                }
            }
        }
        name DN42-to-LAN {
            default-action reject
            rule 10 {
                action accept
                description Established/Related
                state {
                    established enable
                    related enable
                }
            }
            rule 20 {
                action accept
                description ICMP
                protocol icmp
            }
        }
        name WAN-to-Local {
            default-action drop
            rule 10 {
                action accept
                description Established/Related
                state {
                    established enable
                    related enable
                }
            }
            rule 20 {
                action accept
                description ICMP
                protocol icmp
            }
            rule 30 {
                action accept
                description "SSH Management"
                destination {
                    port 22
                }
                protocol tcp
                state {
                    new enable
                }
                tcp {
                    flags SYN,!ACK,!FIN,!RST
                }
            }
            rule 40 {
                action accept
                description IKE
                destination {
                    port 500,4500
                }
                protocol udp
            }
            rule 50 {
                action accept
                description IPSEC/ESP
                protocol esp
            }
            rule 60 {
                action accept
                description "GRE over IPsec"
                ipsec {
                    match-ipsec
                }
                protocol gre
            }
        }
        name established-only {
            default-action drop
            rule 10 {
                action accept
                description Established/Related
                state {
                    established enable
                    related enable
                }
            }
        }
        name allow-all-v4 {
            default-action accept
        }
        options {
            mss-clamp {
                interface-type tun
                mss 1300
            }
        }
        receive-redirects disable
        send-redirects enable
        source-validation disable
        syn-cookies enable
    }
    interfaces {
        ethernet eth0 {
            address 192.0.2.2/30
            description WAN
            duplex auto
            speed auto
        }
        ethernet eth1 {
            address 172.23.248.33/27
            description LAN
            duplex auto
            speed auto
        }
        ethernet eth2 {
            disable
            duplex auto
            speed auto
        }
        loopback lo {
            address 172.23.248.2/32
        }
        tunnel tun0 {
            address 172.23.248.10/31
            description "CREST-DN42 AS64828"
            encapsulation gre
            local-ip 192.0.2.2
            mtu 1400
            multicast disable
            remote-ip 192.0.2.243
            ttl 255
        }
    }
    policy {
        prefix-list DN42-IPv4 {
            rule 1 {
                action permit
                description "DN42 native"
                ge 23
                le 28
                prefix 172.22.0.0/15
            }
            rule 2 {
                action permit
                description "DN42 anycast"
                ge 32
                prefix 172.22.0.0/24
            }
            rule 3 {
                action permit
                description Freifunk
                ge 16
                prefix 10.0.0.0/8
            }
            rule 4 {
                action permit
                description ChaosVPN
                ge 23
                prefix 172.31.0.0/16
            }
            rule 65535 {
                action deny
                prefix 0.0.0.0/0
            }
        }
        route-map DN42 {
            rule 1 {
                action permit
                match {
                    ip {
                        address {
                            prefix-list DN42-IPv4
                        }
                    }
                }
            }
            rule 65535 {
                action deny
            }
        }
    }
    protocols {
        bgp 64746 {
            aggregate-address 172.23.248.0/24 {
                summary-only
            }
            neighbor 172.23.248.11 {
                description CREST-DN42
                peer-group DN42
                remote-as 64828
                update-source 172.23.248.10
            }
            network 172.23.248.0/24 {
            }
            parameters {
                router-id 172.23.248.2
            }
            peer-group DN42 {
                route-map {
                    export DN42
                    import DN42
                }
                soft-reconfiguration {
                    inbound
                }
            }
        }
        static {
            route 0.0.0.0/0 {
                next-hop 192.0.2.1 {
                }
            }
            route 172.23.248.0/24 {
                blackhole {
                    distance 255
                }
            }
        }
    }
    service {
        nat {
            rule 6000 {
                outbound-interface eth0
                type masquerade
            }
        }
        ssh {
            disable-password-authentication
            port 22
            protocol-version v2
        }
        ubnt-discover {
            disable
        }
    }
    system {
        config-management {
            commit-revisions 10
        }
        domain-name ryan.dn42
        host-name edge1
        login {
            banner {
                pre-login ""
            }
            user ryan {
                authentication {
                    encrypted-password :)
                    public-keys ryan {
                        key AAAAB3NzaC1yc2EAAAADAQABAAACAQCymzCbuc777hZ8acvK+68tB7WlZl9V8rQjeQCHny2f9Fy2uSnDHXymUzQJSBY8dr4QM07owCFyYciYqhJRBeBRiaP1dj6avzZzlrOC2xuXSWw4aCYVkEaBPWkntCvBjmPhtvA+x5w8qm0X+B41DG1D44qzrQSmL5geheQCHWSf48Za6RUvPxPuQ+xfBMlIaWscRn95NST2102sYwfl3GDJEqV8FqZ5gQeuG3LDRBQmVEZOSMFIN0pOrp6+UYDe6LSw8eD3uBNrkfbbwwEqjHKFNuYaIw/XNdY0nqhHec0KjsuPLHTQMc44h8CPL5ytAtjF1WnPAE4e3aDQFnB05V/3GThJI010bNkLw5zbGkq0QUa7SmFfAsyOg50grByqZWY/J997HXjWdsgK+7d3K4VQXlI1Uak6G2i0Vb5KX0Xv6dmFmsqwuomeGozBJOl3YebvHI/39Y1VcZls2Zkjg4dBWJQGhsZv8wAX8bf7owtLPE+PcWvX5dRmk44r93mk1M1PTz7XAJGXfeii/OV+QRZZkbzhi3h7VItF5Yv5nptMQUx+irUrIX3gaTHOu8cMTxtP52kIOGOEN/LmYbmrdc++QJNGGadopuZBDpCiR2xQhwQL5yKaXH6Rdenn9d0mdNTzdqw5QOUfjY+SqTMDqLk+ETY+YZ6fvJYDIm4yfgi//Q==
                        type ssh-rsa
                    }
                }
                level admin
            }
        }
        name-server 4.2.2.2
        name-server 8.8.8.8
        ntp {
            server 0.ubnt.pool.ntp.org {
            }
            server 1.ubnt.pool.ntp.org {
            }
            server 2.ubnt.pool.ntp.org {
            }
            server 3.ubnt.pool.ntp.org {
            }
        }
        offload {
            ipsec enable
            ipv4 {
                forwarding enable
            }
            ipv6 {
                forwarding enable
            }
        }
        options {
            reboot-on-panic true
        }
        package {
            repository squeeze {
                components "main contrib non-free"
                distribution squeeze
                password ""
                url http://http.us.debian.org/debian
                username ""
            }
            repository squeeze-security {
                components main
                distribution squeeze/updates
                password ""
                url http://security.debian.org
                username ""
            }
            repository squeeze-updates {
                components "main contrib non-free"
                distribution squeeze-updates
                password ""
                url http://http.us.debian.org/debian
                username ""
            }
        }
        syslog {
            global {
                facility all {
                    level notice
                }
                facility protocols {
                    level debug
                }
            }
        }
    }
    vpn {
        ipsec {
            auto-firewall-nat-exclude disable
            esp-group ESP-AES128-SHA1-DH5-TRANSPORT {
                compression disable
                lifetime 3600
                mode transport
                pfs dh-group5
                proposal 1 {
                    encryption aes128
                    hash sha1
                }
            }
            ike-group IKE-AES128-SHA1-DH5 {
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
                peer 192.0.2.243 {
                    authentication {
                        mode rsa
                        rsa-key-name crest-dn42
                    }
                    connection-type initiate
                    default-esp-group ESP-AES128-SHA1-DH5-TRANSPORT
                    ike-group IKE-AES128-SHA1-DH5
                    local-ip 192.0.2.2
                    tunnel 0 {
                        allow-nat-networks disable
                        allow-public-networks disable
                        esp-group ESP-AES128-SHA1-DH5-TRANSPORT
                        protocol gre
                    }
                }
            }
        }
        rsa-keys {
            rsa-key-name crest-dn42 {
                rsa-key 0sAwEAAbsbRoUcgdm4A4Nm+PLxWcW+zFis7pkaJ0MkGVzM7VC8nmngkM+W2zqZyQ4NUTBKKfGOUc4Ogi6gyhlzUnHdag9tDERIX+BwlDO6G4arod9z9KqmJuX4AOYVjH5QlAPz7NDMAezVekGoVLPGdOAMPD6NN54ihLRH6V3if8AGoJRpiajhcgQipjeQnhH4QhsYK4XSjayGT1onQwA8nhy5kt4ofyqSale4Fl4166S9tCn4RKwtlJDjR6VIrg6op6Ip8+ke2vjEHPJHj6qVsxfRgOk2d8pY8oPVt8ayc5F1z+lqJ7R0fADfN+AQSaBqOMmg5dHDFYWwgYkU5egdVKS7Oko6uNuUWsZ0VEnRoPZ4syJEUbiF5wGfaVBaaVLZYUlRLQCffB4JKzp+JesVToCX6JYRfb4JYQWFCDeQfrqRZHM4r13h8MOWPn9cqXcP47RKJjzNp6595biUotmCbMHyy/uveMWxK6vDzPQRkywqMMJE2qOyACmbMnSce9KlYhvma82Vd+z/9/U9NEy0s5MaYNDn+q+KYT5My3NSv52F6sLVGrKxTk79tzUejZcoukJv+gf51Epam4kVHzPIal/khsfjZn6YCU2j5+qcdRmzF+SG5c2WicvEU2Gc4ratfYNEPxU5oArzHIhIz6x2nAF+szcx/x8GEyXPNHnxEboJB7ox
            }
        }
    }
    zone-policy {
        zone DN42 {
            default-action reject
            description DN42
            from Local {
                firewall {
                    name allow-all-v4
                }
            }
            from LAN {
                firewall {
                    name allow-all-v4
                }
            }
            interface tun0
        }
        zone LAN {
            default-action reject
            from DN42 {
                firewall {
                    name DN42-to-LAN
                }
            }
            from Local {
                firewall {
                    name allow-all-v4
                }
            }
            from WAN {
                firewall {
                    name established-only
                }
            }
            interface eth1
        }
        zone Local {
            default-action reject
            from DN42 {
                firewall {
                    name DN42-to-Local
                }
            }
            from LAN {
                firewall {
                    name allow-all-v4
                }
            }
            from WAN {
                firewall {
                    name WAN-to-Local
                }
            }
            local-zone
        }
        zone WAN {
            default-action reject
            from LAN {
                firewall {
                    name allow-all-v4
                }
            }
            from Local {
                firewall {
                    name allow-all-v4
                }
            }
            interface eth0
        }
    }
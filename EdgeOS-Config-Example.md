## EdgeRouter Lite DN42 config example
This is the config I'm running on an Ubiquiti EdgeRouter Lite (AS76197). It features:

* dn42 DNS
* "classic" OpenVPN P2P (including the common "comp-lzo" option)
* BGP
* Some traffic-shaping rules for my very slow 3mbit DSL uplink
* 2 internal: One DN42 network (172.22.117.128/25 for me and my servers as well as a NAT 192.168.42.10/24 for my parents, so that they're save from dn42 - that network is NOT announced to dn42).
* Firewall to protect my NAS server and monitoring

```
firewall {
    all-ping enable
    broadcast-ping disable
    conntrack-expect-table-size 4096
    conntrack-hash-size 4096
    conntrack-table-size 32768
    conntrack-tcp-loose enable
    ipv6-name ROUTER_V6 {
        default-action drop
        rule 1 {
            action drop
            destination {
                port 22
            }
            protocol tcp
        }
    }
    ipv6-name WAN_IN_V6 {
        default-action drop
        enable-default-log
        rule 3 {
            action drop
            destination {
                port 22
            }
            protocol tcp
        }
    }
    ipv6-receive-redirects disable
    ipv6-src-route disable
    ip-src-route disable
    log-martians enable
    name DN42 {
        default-action drop
        rule 100 {
            action drop
            destination {
                address 172.22.117.181
            }
            source {
                address !172.22.117.128/25
            }
        }
        rule 101 {
            action drop
            destination {
                address 172.22.117.182
            }
            source {
                address !172.22.117.128/25
            }
        }
        rule 102 {
            action drop
            destination {
                address 172.22.117.183
            }
            source {
                address !172.22.117.128/25
            }
        }
    }
    name ROUTER_V4 {
        default-action accept
        rule 2 {
            action accept
            protocol icmp
        }
        rule 10 {
            action drop
            destination {
                port 22
            }
            protocol tcp
        }
    }
    name WAN_IN_V4 {
        default-action drop
        enable-default-log
        rule 1 {
            action accept
            description "allow established connections"
            protocol all
            state {
                established enable
                related enable
            }
        }
        rule 2 {
            action drop
            state {
                invalid enable
            }
        }
        rule 3 {
            action drop
            destination {
                port 22
            }
            protocol tcp
        }
    }
    receive-redirects disable
    send-redirects enable
    source-validation disable
    syn-cookies enable
}
interfaces {
    ethernet eth0 {
        duplex auto
        firewall {
            in {
                name WAN_IN_V4
            }
        }
        pppoe 0 {
            default-route auto
            firewall {
                local {
                    ipv6-name ROUTER_V6
                    name ROUTER_V4
                }
            }
            mtu 1492
            name-server auto
            password 12345678
            traffic-policy {
            }
            user-id some-t-online-crap@t-online.de
        }
        speed auto
    }
    ethernet eth1 {
        address 172.22.117.254/25
        duplex auto
        speed auto
        traffic-policy {
        }
    }
    ethernet eth2 {
        address 192.168.42.1/24
        duplex auto
        speed auto
    }
    loopback lo {
    }
    openvpn vtun0 {
        local-address 172.22.117.254 {
            subnet-mask 255.255.255.128
        }
        local-port 33121
        mode site-to-site
        openvpn-option --comp-lzo
        protocol udp
        remote-address 172.22.117.1
        remote-host 5.9.33.163
        remote-port 33121
        shared-secret-key-file /config/auth/felihome.key
    }
}
policy {
    prefix-list vpn-in {
        rule 10 {
            action permit
            ge 22
            le 28
            prefix 172.22.0.0/15
        }
    }
}
protocols {
    bgp 76197 {
        neighbor 172.22.117.1 {
            description feli-server
            peer-group dn42
            remote-as 64717
        }
        network 172.22.117.128/25 {
        }
        peer-group dn42 {
            soft-reconfiguration {
                inbound
            }
        }
    }
}
service {
    dhcp-server {
        disabled false
        dynamic-dns-update {
            enable true
        }
        shared-network-name int {
            authoritative disable
            subnet 172.22.117.128/25 {
                default-router 172.22.117.254
                dns-server 172.22.117.254
                domain-name feli-home.felicitus.org
                lease 86400
                start 172.22.117.129 {
                    stop 172.22.117.150
                }
                static-mapping monitoring {
                    ip-address 172.22.117.183
                    mac-address 52:54:00:20:df:46
                }
                static-mapping nas {
                    ip-address 172.22.117.181
                    mac-address e8:39:35:ee:22:7b
                }
            }
        }
        shared-network-name nat {
            authoritative disable
            subnet 192.168.42.0/24 {
                default-router 192.168.42.1
                dns-server 8.8.8.8
                dns-server 8.8.4.4
                lease 86400
                start 192.168.42.10 {
                    stop 192.168.42.100
                }
            }
        }
    }
    dns {
        forwarding {
            cache-size 150
            listen-on eth1
            listen-on eth2
            name-server 8.8.8.8
            name-server 8.8.4.4
            options server=/dn42/172.22.0.53
            options server=/22.172.in-addr.arpa/172.22.0.53
            options server=/23.172.in-addr.arpa/172.22.0.53
            options rebind-domain-ok=/dn42/
        }
    }
    nat {
        rule 6000 {
            outbound-interface pppoe0
            type masquerade
        }
        rule 7000 {
            outbound-interface eth2
            type masquerade
        }
    }
    ssh {
        port 22
        protocol-version v2
    }
    upnp {
        listen-on eth1 {
            outbound-interface pppoe0
        }
        listen-on eth2 {
            outbound-interface pppoe0
        }
    }
}
system {
    host-name ubnt
    login {
        user felicitus {
            authentication {
                encrypted-password errnope
                plaintext-password ""
                public-keys felicitus@felicitus.org {
                    key AAAAB3NzaC1yc2EAAAADAQABAAABAQDPTSLjSY/Be1XJ/klAwLiM1pKSvmbdcOgtgDB6nPcHkgX6JZu7g/Kejfuk4qIKL8GYYUQt7DlGY6n2u5rChWE/6KZJzXcUwS3pXk4LZ5KydWp7ihfvyRtUOBgKkRa1zQv+6KCH9WyR++ArwVTP8KSkrmDe6k7NWAjZqOuIJHG/AbEyTBapTJYjObZ0AM7wlwcB+oRM1BfZCP0Y+PIP2eGJS7Pyb32pITNKk3JuFXgAvbj5OeRrwtpZ9S+/7wIpaUVODPzrVmbC7vOXu/2KJ9aY2BmxUsxRbrvWMmWNiuE0YPt/7lUroK4pH3md3lWRcGUS/uYvhug7yG1yB81nyI15
                    type ssh-rsa
                }
            }
            level admin
        }
    }
    name-server 172.22.117.254
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
    time-zone UTC
}
traffic-policy {
    shaper client-up-s {
        bandwidth 30kbit
        class 20 {
            bandwidth 100%
            burst 6k
            match TCPACK {
                ip {
                    protocol tcp
                }
                mark 225
            }
            priority 5
            queue-limit 65
            queue-type fair-queue
        }
        class 30 {
            bandwidth 5%
            burst 15k
            ceiling 20%
            match ssh {
                ip {
                    destination {
                        port 22
                    }
                    dscp lowdelay
                    protocol tcp
                }
            }
            match ssh-ipv6 {
                ipv6 {
                    destination {
                        port 22
                    }
                    protocol tcp
                }
            }
            priority 6
            queue-limit 10
            queue-type fair-queue
        }
        default {
            bandwidth 95%
            burst 15k
            ceiling 100%
            priority 2
            queue-limit 13
            queue-type fair-queue
        }
    }
}


/* Warning: Do not remove the following line. */
/* === vyatta-config-version: "config-management@1:dhcp-relay@1:dhcp-server@4:firewall@4:ipsec@3:nat@3:qos@1:quagga@2:system@4:ubnt-pptp@1:ubnt-util@1:vrrp@1:webgui@1:webproxy@1:zone-policy@1" === */
/* Release version: v1.3.0.4605130.131011.1754 */
```
# NixOS

NixOS is a declarative Linux distribution based on the Nix package Manager. In this post I'll explain how I setup dn42 in this environment. I currently only peer with wireguard and use bird2. NixOS uses configuration files to manage the system state and has a builtin container module.

## container disclaimer

I had a spare IPv4 Address so I decided to use a container without a NAT and keep my host "clean" from dn42 Wireguard Interfaces and IP routes. However it's pain full to debug since nixos-rebuild restarts the container on every minor change. So every time you change a firewall rule or debug a DNS setting nixos-rebuild restarts the container before the change takes effect and since BGP is BGP, it can be really frustrating. 

You may also want to have a look at this [Issue](https://github.com/NixOS/nixpkgs/issues/43652) and [Pull Request](https://github.com/NixOS/nixpkgs/pull/80169)

If you still want to give it a try, here you'll find some inspiration from my setup. You can also use some of these nix expression in a non-container environment. 

## Building the container 

Defining the container environment is the base part of the setup. Beginning with network setup, Private Network disables the passthrough of Host Interfaces into the container and adds a bridged Interface to the host default Interface (e.g. eth0). The localAddress is the container side address and the hostAddress is the one the Host gets. Inside the ```container.<name>.config```, you can basicly import the same nix expression as from the Host and don't need to add some special container parts. 

```nix 
containers.dn42 = {
    hostAddress = "192.168.254.1"; # Transfer Network
    hostAddress6 = "2001:db08::42"; # Transfer Network
    localAddress = "116.203.1.5";
    localAddress6 = "2a01:4f8:c0c:4f7a::2/128";
    privateNetwork = true;
    autoStart = true;

    config = { config, pkgs, ... }: {
        imports = [
            ./peers # Folder with a config for every Peer
            ./dns.nix # Bind with the litschi.dn42 zone deligated
            ./bird.nix # Bird config for BGP Routing
            ./networking.nix # Static Network configuration (with firewall)
            ./nginx.nix # nginx config for litschi.dn42
        ];
        environment.systemPackages = with pkgs; [ 
            # Network debug tools
            dnsutils
            mtr
            tcpdump
            wireguard-tools
        ];
    }
}
```

In theory the container should now be starting and you can get shell access  with ```sudo nixos-container root-login <name> ```.

I mounted some host paths into the container for dns zone files and static homepage since the container is the only one providing .dn42 webservers. 

```nix
containers.dn42 = {
    bindMounts = {
        "/var/www/dn42" = {
            hostPath = "/var/www/dn42";
            isReadOnly = true;
            mountPoint = "/var/www/dn42";
        };
        "/var/dns/dn42" = {
            hostPath = "/var/dns/dn42";
            isReadOnly = true;
            mountPoint = "/var/dns";
        };
    };
}
```

### Network Setup 

As mentioned above, I got a spare public IPv4 Address, but by adding it as ```localAddress```, the container Part is configured static enough. But to forward traffic between Intferfaces ```/proc/sys/net/``` should configured

```nix
boot.kernel.sysctl = {
    "net.ipv4.ip_forward" = 1;
    "net.ipv6.conf.all.forwarding" = 1;
};
```
This allows our firewall to configure forwarding between peers and other tunnels. What is allowed to be forwarded can be configured in the firewall. Ferm has only few NixOS Options, but is pretty basic. Its configured with the ```services.ferm.config``` options, that contains just a string. Within this string there's standard plain ferm config. Example config is attached below. 
If the dn42 address is not bound at any other Interface, you need to add it to the lo Interface to use it as source IP when routing via peers with dedicated transfer net. 
```nix
networking.interfaces.lo = {
    ipv4.addresses = [
        {
            address = "172.23.73.65";
            prefixLength = 32;
        }
    ];
    ipv6.addresses = [
        {
            address = "fd67:24bd:a1ea::1";
            prefixLength = 128;
        }
    ];
};
```

#### Ferm example
```nix
services.ferm = {
    enable = true;
    config = ''
        domain ip table filter chain INPUT proto icmp ACCEPT;
        domain ip6 table filter chain INPUT proto (ipv6-icmp icmp) ACCEPT;
        domain (ip ip6) table filter {
            chain INPUT {
                policy DROP;
                interface lo ACCEPT;
                interface intern-+ ACCEPT;
                # website
                proto tcp dport (http https) ACCEPT;
                # wireguard
                proto udp dport ( <Wireguard Ports> ) ACCEPT;
                # bgp
                proto tcp dport (179) ACCEPT;
                # dns
                proto (udp tcp) dport domain ACCEPT;
                mod state state (INVALID) DROP;
                mod state state (ESTABLISHED RELATED) ACCEPT;
            }
            chain OUTPUT {
                policy ACCEPT;
            }
            chain FORWARD {
                policy DROP;
                # allow intern routing and dn42 forwarding
                interface dn42-+ outerface dn42-+ ACCEPT;
                interface intern-+ outerface intern-+ ACCEPT;
                interface intern-+ outerface dn42-+ ACCEPT;
                # but dn42 -> intern only with execptions
                interface dn42-+ outerface intern-+ {
                    proto (ipv6-icmp icmp) ACCEPT; # Allow SSH Access from dn42 to devices behind intern-+ Interfaces
                    proto tcp dport (ssh) ACCEPT;
                    mod state state (ESTABLISHED) ACCEPT;
                }
            }
        }
    '';
};
```

### Peering with wireguard

Explained above, every peer gets a dedicated wireguard Interface and so a dedicated file. In the container config folder theres a peer subfolder and within a folder for dn42- (extern) Peers and intern- configs e.g. my Home Router or mobile devices. 

A sample wireguard config may look like this:
```nix
{config, pkgs, ...}:
{
    networking.wireguard.interfaces.dn42-peer = {
        privateKey = "";
        allowedIPsAsRoutes = false;
        listenPort = 42420;

        peers = [
            {
                publicKey = "";
                allowedIPs = [ "0.0.0.0/0" "::/0" ];
                endpoint = "42.42.42.42:42421";
            }
        ];
        postSetup = ''
            ${pkgs.iproute}/bin/ip addr add 169.254.0.1/32 peer 169.254.0.0/32 dev dn42-peer
            ${pkgs.iproute}/bin/ip -6 addr add fe80::1220/64 dev dn42-peer
        '';
    };
}
```

As seen, the IP configuration is applied via ip-commands in the postSetup. This kinda works but isn't a fancy solution. There's room for improvements e.g. configuring static addresses and routes with networkd. 

### BGP Routing with bird2

Like ferm, Bird2 is configured by ```services.bird2.config``` containing a string. In there the example bird2 config from [wiki.dn42](/howto/Bird2) can be imported. Roa tables can be generated or downloaded from host providing them. 


#### ROA Updating script

Sample example to update ROA's : 
```nix
{ pkgs, lib, ... }:
let script = pkgs.writeShellScriptBin "update-roa" ''
    mkdir -p /etc/bird/
    ${pkgs.curl}/bin/curl -sfSLR {-o,-z}/etc/bird/roa_dn42_v6.conf https://dn42.burble.com/roa/dn42_roa_bird2_6.conf
    ${pkgs.curl}/bin/curl -sfSLR {-o,-z}/etc/bird/roa_dn42.conf https://dn42.burble.com/roa/dn42_roa_bird2_4.conf
    ${pkgs.bird2}/bin/birdc c 
    ${pkgs.bird2}/bin/birdc reload in all
    '';
in
{
    systemd.timers.dn42-roa = {
        description = "Trigger a ROA table update";

        timerConfig = {
            OnBootSec = "5m";
            OnUnitInactiveSec = "1h";
            Unit = "dn42-roa.service";
        };

        wantedBy = [ "timers.target" ];
        before = [ "bird.service" ];
    };

    systemd.services = {
        dn42-roa = {
            after = [ "network.target" ];
            description = "DN42 ROA Updated";
            unitConfig = {
                Type = "one-shot";
            };
            serviceConfig = {
                ExecStart = "${script}/bin/update-roa";
            };
        };
    };
}
```

### Bird Looking Glass

There is now (thanks to [Tchekda](https://github.com/NixOS/nixpkgs/pull/153481)) a direct way to setup a looking glass for bird on Nixos. [Documentation](https://github.com/NixOS/nixpkgs/blob/3aab5ebd436023ca8343a84804d51cd227dd01dd/nixos/modules/services/networking/bird-lg.nix) and sample : 

```nix
bird-lg = {
    proxy = {
        enable = true;
        allowedIPs = [ "172.20.XX.XX" "172.20.XX.YY" ];
    };
    frontend = {
        enable = true;
        netSpecificMode = "dn42";
        servers = [ "node1" "node2" ];
        domain = "domain.dn42";
    };
};
```

### Services

I also run services like a nameserver for .litschi.dn42 zones and a nginx webserver within this container. Since Host path for ```/var/www/dn42``` and ```/var/dns/dn42``` are booth binded into the container, zone config and e.g. website and be edited directly from Host without need the rebuild the hole container. 

### Sample configuration

You can find a sample Wireguard + Bird configuration made by Tchekda ready for dn42 on [this](https://github.com/Tchekda/nixos-configuration/tree/master/llitt/dn42) repository


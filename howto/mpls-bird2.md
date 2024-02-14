Original Article: [https://blog.sherpherd.top/2024/02/11/RunYourMPLSNetworkWithBIRD_en.html](https://blog.sherpherd.top/2024/02/11/RunYourMPLSNetworkWithBIRD_en.html)

# Intro
Now, most tutorials about running MPLS on Linux are based on FRR. Because in a long time, FRR and its predecessor Quagga are the only choices who provide industry standard MPLS  related protocol (LDP, BGP-LU, BGP IPv4/IPv6 MPLS L3VPN, etc.) implementation, by the time, most of other routing software don't even have availiable MPLS support.

The most popular routing software among DN42 users, BIRD, has added availiable MPLS support in its newest version (2.14), now BIRD has MPLS-aware, labeled route producing routing protocol, too. (No LDP still though) [<sup>[1]</sup>](#c1)

The newest BIRD 2.0 User Guide has added MPLS function related chapter[<sup>[2]</sup>](#c2), this is an excerpt:

> In BIRD, the whole process generally works this way: A MPLS-aware routing protocol (say BGP) receives
routing information including remote label. It produces a route with attribute mpls policy (p. 30) specifying
desired MPLS label policy (p. 18). Such route then passes the import filter (which could modify the MPLS
label policy or perhaps assign a static label) and when it is accepted, a local MPLS label is selected (according
to the label policy) and attached to the route, producing labeled route. When a new MPLS label is allocated,
the MPLS-aware protocol automatically produces corresponding MPLS route. When all labeled routes that
use specific local MPLS label are retracted, the corresponding MPLS route is retracted too. <br>
There are three important concepts for MPLS in BIRD: MPLS domains, MPLS tables and MPLS channels.
MPLS domain represents an independent label space, all MPLS-aware protocols are associated with some MPLS domain. It is responsible for label management, handling label allocation requests from MPLS-aware protocols. MPLS table is just a routing table for MPLS routes. Routers usually have one MPLS domain and one MPLS table, with Kernel protocol to export MPLS routes into kernel FIB. <br> 
MPLS channels make protocols MPLS-aware, they are responsible for keeping track of active FECs (and corresponding allocated labels), selecting FECs / local labels for labeled routes, and maintaining correspondence between labeled routes and MPLS routes.

As mentioned above, the current BGP implementation of BIRD is MPLS-awared, can be used to assign and distribute MPLS labeled route.

In this article, I will make use of official BIRD document to show readers how to construct a simple MPLS VPN network running with BIRD.

# Prerequisites
* Unless specific configuration, your node must running completely independent kernel (dedicated server, KVM virtualization, etc.), so that you can enable MPLS kernel module.
* If you using Vultr VPS, due to unknown reason, Vultr integrated system image has some trouble with MPLS, please reinstall your system with official ISO after deployment.

# Table of Contents
- [Intro](#intro)
- [Prerequisites](#prerequisites)
- [Table of Contents](#table-of-contents)
- [1 Lab Topo](#1-lab-topo)
  - [1.1 Node Specs](#1-1-node-specs)
- [2 Preliminary Work](#2-preliminary-work)
  - [2.1 Enable MPLS Kernel Module](#2-1-enable-mpls-kernel-module)
  - [2.2 Kernel Parameter Adjustment](#2-2-kernel-parameter-adjustment)
    - [2.2.1 Enable MPLS Input on MPLS Port](#2-2-1-enable-mpls-input-on-mpls-port)
  - [2.3 Create VRF and assign VRF for port](#2-3-create-vrf-and-assign-vrf-for-port)
    - [2.3.1 Adjust Client-faced Port MTU to Avoid Fragmentation](#2-3-1-adjust-client-faced-port-mtu-to-avoid-fragmentation)
  - [2.4 IP Address and Static Route Configuration](#2-4-ip-address-and-static-route-configuration)
  - [2.5 Installing BIRD](#2-5-installing-bird)
    - [2.5.1 Install Compiling and Building Dependencies](#2-5-1-install-compiling-and-building-dependencies)
    - [2.5.2 Clone BIRD 2.14 Repo](#2-5-2-clone-bird-214-repo)
    - [2.5.3 Build Software Package of BIRD 2.14](#2-5-3-build-software-package-of-bird-214)
- [3 BIRD Configuration](#3-bird-configuration)
  - [3.1 Basic Setup](#3-1-basic-setup)
    - [3.1.1 Router ID](#3-1-1-router-id)
    - [3.1.2 Adding MPLS Domain and Tables](#3-1-2-adding-mpls-domain-and-tables)
    - [3.1.3 Adding MPLS and VRF Related Protocol and Configuration](#3-1-3-adding-mpls-and-vrf-related-protocol-and-configuration)
  - [3.2 BGP Configuration](#3-2-bgp-configuration)
  - [3.3 Setup Binding between MPLS L3VPN and VRF Instance](#3-3-setup-binding-between-mpls-l3vpn-and-vrf-instance)
  - [3.4 Complete BIRD Configuration File](#3-4-complete-bird-configuration-file)
    - [3.4.1 R1](#3-4-1-r1)
    - [3.4.2 R2](#3-4-2-r2)
    - [3.4.3 R3](#3-4-3-r3)
- [4 Verification](#4-verification)
  - [4.1 Check VPNv4 Table](#4-1-check-vpnv4-table)
  - [4.2 Check Default IPv4 Table](#4-2-check-default-ipv4-table)
  - [4.3 Check VRF IPv4 Table](#4-3-check-vrf-ipv4-table)
  - [4.4 Check connectivity between PC1 and PC2](#4-4-check-connectivity-between-pc1-and-pc2)
- [5 Reference](#5-reference)

# 1 Lab Topo
```

                 ----------------------------------------------------------
--------         |    --------            --------            --------    |         --------
|      |  eth0 ens20  |      | ens19 ens19|      | ens20 ens19|      |  ens20 eth0  |      |
| PC1  | O==========O |  R1  |O==========O|  R3  |O==========O|  R2  | O==========O | PC2  |
|      |         |    |      |            |      |            |      |    |         |      |
--------         |    --------            --------            --------    |         --------
                 |                                                        |
                 |         Confederation ASN: 100                         |
                 |                    R1 ASN: 64512                       |
                 |                    R2 ASN: 64513                       |
                 |                    R3 ASN: 64514                       |
                 |                        RT: 100:500                     |
                 |                     R1 RT: 203.0.113.1:500             |
                 |                     R2 RT: 203.0.113.2:500             |
                 |                                                        |
                 ----------------------------------------------------------
                 |                    Address Assignment                  |
                 ----------------------------------------------------------
                 |                                                        |
                 |                  eth0@PC1: 192.168.1.2/24              |
                 |                  ens20@R1: 192.168.1.1/24              |
                 |                                (vrf blue)              |
                 |                     lo@R1: 203.0.113.1/32              |
                 |                  ens19@R1: 203.0.113.1/32              |
                 |                     lo@R3: 203.0.113.3/32              |
                 |                  ens19@R3: 203.0.113.3/32              |
                 |                  ens20@R3: 203.0.113.3/32              |
                 |                     lo@R2: 203.0.113.2/32              |
                 |                  ens19@R2: 203.0.113.2/32              |
                 |                  ens20@R2: 192.168.2.1/24              |
                 |                                (vrf blue)              |
                 |                  eth0@PC2: 192.168.2.2/24              |
                 |                                                        |
                 ----------------------------------------------------------

```

## 1.1 Node Specs
PC1, R1, R2 and PC2 all running Debian 12. R1 and R2 both installed newest version BIRD by the time I finished this, the BIRD 2.14.

**Notice: Remember to add third port for R1, R2 and R3 to make them able to access Internet for downloading BIRD software package or compiling dependencies**

# 2 Preliminary Work
## 2.1 Enable MPLS Kernel Module
Run these command on R1 and R2 with root permission:
```
modprobe mpls_router
modprobe mpls_iptunnel
modprobe mpls_gso
```
## 2.2 Kernel Parameter Adjustment
Run these command on R1, R2 and R3 with root permission to adjust parameters related to IP routing and MPLS, make them able to work[<sup>[3]</sup>](#c3):
```
cat >/etc/sysctl.d/90-mpls-router.conf <<EOF
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
net.ipv4.conf.all.rp_filter=0
net.mpls.platform_labels=1048575
net.ipv4.tcp_l3mdev_accept=1
net.ipv4.udp_l3mdev_accept=1
net.mpls.conf.lo.input=1
EOF
sysctl -p /etc/sysctl.d/90-mpls-router.conf
```
### 2.2.1 Enable MPLS Input on MPLS Port
Every port transits MPLS traffic need to enable MPLS input, run these command on R1 and R2 with root permission to enable MPLS input for their port ens19:
```
sysctl -w net.mpls.conf.ens19.input=1
```
So do on R3:
```
sysctl -w net.mpls.conf.ens19.input=1
sysctl -w net.mpls.conf.ens20.input=1
```
**Notice: Every MPLS traffic transiting port need this configuration**
## 2.3 Create VRF and assign VRF for port
Run these command on R1 and R2 with root permission to create a VRF interface named "blue":
```
ip link add blue type vrf table 500
ip link set blue up
```
Run these command on R1 and R2 with root permission to assign ens20 to VRF blue then enable it:
```
ip link set ens20 master blue up
```
### 2.3.1 Adjust Client-faced Port MTU to Avoid Fragmentation
In practice, increasing MTU of core network link is always harder than decreasing client-faced port MTU, and using MPLS incur additional packet header overhead (4 bytes per label), this made large packet may get fragmented when entering MPLS network. To avoid this, we need to approviately decrease the MTU of client-faced port.

Run these command on PC1 and PC2 with root permission to adjust MTU of eth0 then enable it:
```
ip link set eth0 mtu 1492 up
```
Run these command on R1 and R2 with root permission to adjust MTU of ens20:
```
ip link set ens20 mtu 1492
```
## 2.4 IP Address and Static Route Configuration
Run command with root permission on nodes below to done this.
R1:
```
ip addr add 203.0.113.1/32 dev lo
ip addr add 203.0.113.1/32 dev ens19 peer 203.0.113.3/32
ip addr add 192.168.1.1/24 dev ens20
```
R2:
```
ip addr add 203.0.113.2/32 dev lo
ip addr add 203.0.113.2/32 dev ens19 peer 203.0.113.3/32
ip addr add 192.168.2.1/24 dev ens20
```
R3:
```
ip addr add 203.0.113.3/32 dev lo
ip addr add 203.0.113.3/32 dev ens19 peer 203.0.113.1/32
ip addr add 203.0.113.3/32 dev ens20 peer 203.0.113.2/32
```
PC1:
```
ip addr add 192.168.1.2/24 dev eth0
ip route add 192.168.2.0/24 via 192.168.1.1
```
PC1:
```
ip addr add 192.168.2.2/24 dev eth0
ip route add 192.168.1.0/24 via 192.168.2.1
```
## 2.5 Installing BIRD
The compile installed BIRD is incomplete, it lacks system service file, docs, etc.

If you want complete BIRD, you have to build software package then install from it.

If you don't want build yourself, you can download them here (deb package):

[https://drive.google.com/drive/folders/1DUaFJgZGsEXI-RlreNxCG9mnERiAkIVB?usp=drive_link](https://drive.google.com/drive/folders/1DUaFJgZGsEXI-RlreNxCG9mnERiAkIVB?usp=drive_link)

If hints dependency miss, follow the hint to install missed dependency.

If you want to build it yourself, please read the rest of this chapter.

Prepare a compile node running Debian 12, no need of high spec, mine got 4 cores and 4 gigs of RAM, then do as follow.

### 2.5.1 Install Compiling and Building Dependencies
```
apt install -y git linuxdoc-tools autoconf build-essential libssh-dev libreadline-dev libncurses-dev flex bison checkinstall debhelper docbook-xsl libssh-gcrypt-dev quilt xsltproc linuxdoc-tools-latex texlive-latex-extra
```
```
pipx install apkg
```
### 2.5.2 Clone BIRD 2.14 Repo
```
git clone --branch v2.14 https://gitlab.nic.cz/labs/bird.git
```
### 2.5.3 Build Software Package of BIRD 2.14
Enter "bird" then run command below:
```
apkg build
```
Once finished, apkg gives hint like this:
```
built 3 packages in: pkg/pkgs/debian-12/bird2_2.14.1707409394.0e1fbaa5-cznic.1
```
The built software package is located in the location the hint mentioned, make use of it.

# 3 BIRD Configuration
Currently BIRD don't have implementation of distributing MPLS labeled route through IGP topo, so we use BGP-LU to do that.
## 3.1 Basic Setup
### 3.1.1 Router ID
Having a static Router ID is always not a bad thing.

R1:
```
router id 203.0.113.1;
```
So do on R2 and R3.

### 3.1.2 Adding MPLS Domain and Tables
Use R1 as example, so do on other nodes:
```
mpls domain mpls_dom;

mpls table bgp_mpls_table;

vpn4 table bgp_vpn4;

ipv4 table vrf_blue4; # This one is no need on R3
```

### 3.1.3 Adding MPLS and VRF Related Protocol and Configuration
Use R1 as example:
```
protocol kernel krt_mpls {
	mpls {
		table bgp_mpls_table;
		export all;
	};
}

protocol kernel vrf_blue_4 { # No need for R3, since it doesn't run any VRF instance
	vrf "blue";
	ipv4 {
		table vrf_blue4;
		export all;
		import all;
	};
	kernel table 500;
}

protocol static {
	ipv4;
	route 203.0.113.1/32 reject; # Inject direct route through static, for advertising it in BGP
}

protocol static {                # Same, no need for R3
	ipv4 { table vrf_blue4; };
	route 192.168.1.0/24 reject;
}
```

## 3.2 BGP Configuration
Use R1 as example:
```
protocol bgp r3 {
	local 203.0.113.1 as 64512;
	neighbor 203.0.113.3 as 64514;
	confederation 100;
	confederation member;
	ipv4 mpls { # Enable IPv4 Labeled Unicast channel, to enable MPLS reachability between MPLS nodes
		import all;
		export all;
	};
	vpn4 mpls { # Enable VPNv4 channel, carring IPv4 VPN route
		table bgp_vpn4;
		import all;
		export all;
	};
	mpls {
		label policy aggregate;
	};
}
```

## 3.3 Setup Binding between MPLS L3VPN and VRF Instance
No need for R3, it doesn't run any VRF instance.

Use R1 as example:
```
protocol l3vpn vpn_blue4 {
	vrf "blue";
	ipv4 { table vrf_blue4; }; # # Binding VRF Table
	vpn4 { table bgp_vpn4; }; # Binding VPNv4 Table
	mpls { label policy vrf; };

	rd 203.0.113.1:500;
	import target [(rt,100,500)]; # Define RT the desired route import from binded VPNv4 to VRF have
	export target [(rt,100,500)]; # Define RT the route export from VRF to binded VPNv4 will be attached
}
```

## 3.4 Complete BIRD Configuration File
### 3.4.1 R1
```
log syslog all;

router id 203.0.113.1;

mpls domain mpls_dom;

mpls table bgp_mpls_table;

vpn4 table bgp_vpn4;

ipv4 table vrf_blue4;

protocol device {

}

protocol direct {
	disabled; # Disable by default
	ipv4;			# Connect to default IPv4 table
	ipv6;			# ... and to default IPv6 table
}

protocol kernel {
	ipv4 {			# Connect protocol to IPv4 table by channel
	      export all;	# Export to protocol. default is export none
	      import all;
	};
}

protocol kernel {
	ipv6 { export all; };
}

protocol kernel krt_mpls {
	mpls {
		table bgp_mpls_table;
		export all;
	};
}

protocol kernel vrf_blue_4 {
	vrf "blue";
	ipv4 {
		table vrf_blue4;
		export all;
		import all;
	};
	kernel table 500;
}

protocol static {
	ipv4;			# Again, IPv4 channel with default options
	route 203.0.113.1/32 reject;
}

protocol static {
	ipv4 { table vrf_blue4; };
	route 192.168.1.0/24 reject;
}

protocol bgp r3 {
	local 203.0.113.1 as 64512;
	neighbor 203.0.113.3 as 64514;
	confederation 100;
	confederation member;
	ipv4 mpls {
		import all;
		export all;
	};
	vpn4 mpls {
		table bgp_vpn4;
		import all;
		export all;
	};
	mpls {
		label policy aggregate;
	};
}

protocol l3vpn vpn_blue4 {
	vrf "blue";
	ipv4 { table vrf_blue4; };
	vpn4 { table bgp_vpn4; };
	mpls { label policy vrf; };

	rd 203.0.113.1:500;
	import target [(rt,100,500)];
	export target [(rt,100,500)];
}

```
### 3.4.2 R2
```
router id 203.0.113.2;

log syslog all;

mpls domain mpls_dom;

mpls table bgp_mpls_table;

vpn4 table bgp_vpn4;

ipv4 table vrf_blue4;

protocol device {
}

protocol direct {
	disabled; # Disable by default
	ipv4;			# Connect to default IPv4 table
	ipv6;			# ... and to default IPv6 table
}

protocol kernel krt_mpls {
        mpls {
                table bgp_mpls_table;
                export all;
        };
}

protocol kernel vrf_blue_4 {
        vrf "blue";
        ipv4 {
                table vrf_blue4;
                export all;
                import all;
        };
        kernel table 500;
}

protocol kernel {
	ipv4 {			# Connect protocol to IPv4 table by channel
	      export all;	# Export to protocol. default is export none
	};
}

protocol kernel {
	ipv6 { export all; };
}

protocol static {
	ipv4;			# Again, IPv4 channel with default options
	route 203.0.113.2/32 reject;
}

protocol static {
	ipv4 { table vrf_blue4; };
	route 192.168.2.0/24 reject;
}

protocol bgp r3 {
        local 203.0.113.2 as 64513;
        neighbor 203.0.113.3 as 64514;
        confederation 100;
        confederation member;
        ipv4 mpls {
                import all;
                export all;
        };
        vpn4 mpls {
                table bgp_vpn4;
                import all;
                export all;
        };
        mpls {
                label policy aggregate;
        };
}

protocol l3vpn vpn_blue4 {
        vrf "blue";
        ipv4 { table vrf_blue4; };
        vpn4 { table bgp_vpn4; };
        mpls { label policy vrf; };

        rd 203.0.113.2:500;
        import target [(rt,100,500)];
        export target [(rt,100,500)];
}

```
### 3.4.3 R3
```
log syslog all;

router id 203.0.113.3;

mpls domain mpls_dom;

mpls table bgp_mpls_table;

vpn4 table bgp_vpn4;

protocol device {
}

protocol direct {
	disabled;		# Disable by default
	ipv4;			# Connect to default IPv4 table
	ipv6;			# ... and to default IPv6 table
}

protocol kernel {
	ipv4 {			# Connect protocol to IPv4 table by channel
	      export all;	# Export to protocol. default is export none
	};
}

protocol kernel {
	ipv6 { export all; };
}

protocol kernel krt_mpls {
	mpls {
		table bgp_mpls_table;
		export all;
	};
};

protocol static {
	ipv4;			# Again, IPv4 channel with default options

}

protocol bgp r1 {
        local 203.0.113.3 as 64514;
        neighbor 203.0.113.1 as 64512;
        confederation 100;
        confederation member;
        ipv4 mpls {
		next hop self;
                import all;
                export all;
        };
        vpn4 mpls {
		next hop self;
                table bgp_vpn4;
                import all;
                export all;
        };
        mpls {
                label policy aggregate;
        };
}

protocol bgp r2 {
        local 203.0.113.3 as 64514;
        neighbor 203.0.113.2 as 64513;
        confederation 100;
        confederation member;
        ipv4 mpls {
		next hop self;
                import all;
                export all;
        };
        vpn4 mpls {
		next hop self;
                table bgp_vpn4;
                import all;
                export all;
        };
        mpls {
                label policy aggregate;
        };
}

```

# 4 Verification
## 4.1 Check VPNv4 Table
R1：
```
bird> show route table bgp_vpn4
Table bgp_vpn4:
203.0.113.2:500 192.168.2.0/24 mpls 1001 unicast [r3 23:20:55.236] * (100) [AS64513i]
        via 203.0.113.3 on ens19 mpls 1002
203.0.113.1:500 192.168.1.0/24 mpls 1002 unicast [vpn_blue4 22:58:48.918] * (120/0)
        dev blue
bird>
```
R2：
```
bird> show route table bgp_vpn4
Table bgp_vpn4:
203.0.113.2:500 192.168.2.0/24 mpls 1001 unicast [vpn_blue4 23:20:55.219] * (120/0)
        dev blue
203.0.113.1:500 192.168.1.0/24 mpls 1002 unicast [r3 22:58:56.352] * (100) [AS64512i]
        via 203.0.113.3 on ens19 mpls 1003
bird>
```
## 4.2 Check Default IPv4 Table
R1：
```
bird> show route table master4
Table master4:
203.0.113.2/32 mpls 1000 unicast [r3 22:58:56.355] * (100) [AS64513i]
        via 203.0.113.3 on ens19 mpls 1000
203.0.113.1/32       unreachable [static1 22:33:27.446] * (200)
bird>
```
R2：
```
bird> show route table master4
Table master4:
203.0.113.2/32       unreachable [static1 22:32:38.874] * (200)
203.0.113.1/32 mpls 1000 unicast [r3 22:58:56.352] * (100) [AS64512i]
        via 203.0.113.3 on ens19 mpls 1001
bird>
```
## 4.3 Check VRF IPv4 Table
R1：
```
bird> show route table vrf_blue4
Table vrf_blue4:
192.168.1.0/24       unreachable [static2 22:58:48.918] * (200)
192.168.2.0/24       unicast [vpn_blue4 23:20:55.236] * (80/0)
        via 203.0.113.3 on ens19 mpls 1002
bird>
```
R2：
```
bird> show route table vrf_blue4
Table vrf_blue4:
192.168.1.0/24       unicast [vpn_blue4 23:20:55.219] * (80/0)
        via 203.0.113.3 on ens19 mpls 1003
192.168.2.0/24       unreachable [static2 22:54:38.777] * (200)
bird>
```
## 4.4 Check connectivity between PC1 and PC2
PC1：
```
root@pc1:~# ping -c 4 192.168.2.2
PING 192.168.2.2 (192.168.2.2) 56(84) bytes of data.
64 bytes from 192.168.2.2: icmp_seq=1 ttl=61 time=5.53 ms
64 bytes from 192.168.2.2: icmp_seq=2 ttl=61 time=5.03 ms
64 bytes from 192.168.2.2: icmp_seq=3 ttl=61 time=3.73 ms
64 bytes from 192.168.2.2: icmp_seq=4 ttl=61 time=5.97 ms

--- 192.168.2.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 3.729/5.063/5.965/0.838 ms
root@pc1:~# traceroute 192.168.2.2
traceroute to 192.168.2.2 (192.168.2.2), 30 hops max, 60 byte packets
 1  192.168.1.1 (192.168.1.1)  5.787 ms  6.165 ms *
 2  * * *
 3  * * *
 4  192.168.2.2 (192.168.2.2)  36.865 ms  37.489 ms  44.775 ms
root@pc1:~#
```
PC2：
```
root@pc2:~# ping -c 4 192.168.1.2
PING 192.168.1.2 (192.168.1.2) 56(84) bytes of data.
64 bytes from 192.168.1.2: icmp_seq=1 ttl=61 time=21.7 ms
64 bytes from 192.168.1.2: icmp_seq=2 ttl=61 time=4.35 ms
64 bytes from 192.168.1.2: icmp_seq=3 ttl=61 time=13.6 ms
64 bytes from 192.168.1.2: icmp_seq=4 ttl=61 time=4.67 ms

--- 192.168.1.2 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3007ms
rtt min/avg/max/mdev = 4.352/11.098/21.731/7.181 ms
root@pc2:~# traceroute 192.168.1.2
traceroute to 192.168.1.2 (192.168.1.2), 30 hops max, 60 byte packets
 1  192.168.2.1 (192.168.2.1)  17.272 ms  17.125 ms  17.175 ms
 2  * * *
 3  * * *
 4  192.168.1.2 (192.168.1.2)  27.517 ms  27.945 ms  32.354 ms
root@pc2:~#
```

# 5 Reference
<span id="c1">1. BIRD Team. (2023, October 7). _News Archive_. bird.network.cz. [https://bird.network.cz/?o_news/](https://bird.network.cz/?o_news/)</span>

<span id="c2">2. BIRD Team. (2023, October 7). BIRD 2.0 User’s Guide. _MPLS_, 9-10. [https://bird.network.cz/download/bird-doc-2.14.tar.gz](https://bird.network.cz/download/bird-doc-2.14.tar.gz)</span>

<span id="c3">3. James Swineson. (2020, February 22). _Use Linux as an MPLS Router_. blog.swineson.me. [https://blog.swineson.me/en/use-linux-as-an-mpls-router/](https://blog.swineson.me/en/use-linux-as-an-mpls-router/)</span>

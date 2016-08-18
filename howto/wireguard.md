To quote the [homepage](https://www.wireguard.io/):

> WireGuard is an extremely simple yet fast and modern VPN that utilizes state-of-the-art cryptography. It aims to be faster, simpler, leaner, and more useful than IPSec, while avoiding the massive headache. It intends to be considerably more performant than OpenVPN. WireGuard is designed as a general purpose VPN for running on embedded interfaces and super computers alike, fit for many different circumstances. Initially released for the Linux kernel, it plans to be cross-platform and widely deployable. It is currently under heavy development, but already it might be regarded as the most secure, easiest to use, and simplest VPN solution in the industry.

# Example configuration for dn42

Wireguard is a Layer3 VPN. In theory it allows multiple peers to be served with one interface/port, but it does internal routing based on the public key of the peers. This means you will need one interface per peering on dn42
to allow your BGP deamon instead to do routing. This approach is comparable to [openvpn p2p](/howto/openvpn)
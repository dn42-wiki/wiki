# Welcome to perchnet (AS4242422825)

## Introduction
perchnet is a network on dn42. The goal of perchnet is to facilitate learning about and experimentation with various networking technologies, and linking up multiple sites in the "hybrid" and "multivendor" cloud computing configurations.



## Background
perchnet is still being designed and deployed. Presently we consist of two physical nodes at _bri_'s residence in New York City that provide a core router and various services, and we're in the process of onboarding a virtual dedicated server graciously donated by [Evolution Host](https://evolution-host.com).

_bri_ enjoys hacking together complex systems out of weird, secondhand, obsolete hardware. As such, their on-site residence nodes are:

- macpro "home private cloud"
  - Heavily upgraded Apple "Mac Pro (early 2009)" (MacPro4,1)
    - Customized firmware
    - Proxmox 7 "testing"
    - 2x Xeon X5675 (12 cores / 24 threads)
    - 128 GB DDR3
    - 512 GB NVMe SSD
    - 2 TB NVMe SSD
    - 2 TB SATA SSD
    - 2x 120 GB SATA SSD
    - 2 TB SATA HDD
    - 1 TB SATA HDD
    - Radeon RX580
- minipve "home virtual router"
  - Apple "Mac Mini (2014)"
    - Proxmox 7
    - Intel Core i5-4278U (2 cores / 4 threads)
    - 16 GB DDR4
    - 128 GB SSD
    - 1 TB HDD

Routing software in use currently primarily consists of a custom build of VyOS 1.4-rolling and OpenWRT.


Note: "perchnet" is officially spelled in all lowercase, but due to constraints it may be written as all uppercase ("PERCHNET") instead. "Perchnet" is not to be used whenever possible. This is all due to an odd idiocyncractic quirk of the network administrator, and should probably not be questioned for the sake of maintaining one's sanity. Nobody will be chastised for using title case, but it will make _bri_ frown.
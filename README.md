# metal_franken_disk

![](https://raw.githubusercontent.com/dlotterman/metal_franken_disk/main/docs/assets/diagram.PNG)

This repository is an open pitch for an NVMe/TCP (NVMe over fabrics using TCP) product at Equinix Metal. It deploys stock Metal instances as targets equipped with high performance NVMe (namely the [m3.large.x86](https://deploy.equinix.com/product/servers/m3-large/)) drives, and uses [linux-nvme](https://github.com/linux-nvme/) to export those NVMe drives as NVMe over TCP targets. It provisions another stock Metal host, an [n3.xlarge.x86](https://deploy.equinix.com/product/servers/n3-xlarge/) as an NVMe/TCP initiator, targeting the NVMe/TCP exposed drives.

The n3.xlarge.x86, here referred to as `mfdh`, becomes a "Franken Disk" host, able to access the network attached NVMe as if they were local drives.

This effectively allows us to create a **mockup** or **lab** of what an NVMe/TCP product at Equinix Metal could look like, using m3's or other NVMe enabled configurations essentially as chassis shells to export the NVMe as Fabric targets.

There is no reason why anyone would want to do anything like this for production or serious purposes, this is simply using whats available today with the product to make an appeal for a future product.

![](https://raw.githubusercontent.com/dlotterman/metal_franken_disk/main/docs/assets/dashboard.PNG)

## Setup

* [Getting Started Doc](docs/getting_started.md)


## Product Economics

We can use public Equinix Metal pricing to build our initial understanding of potential product economics.

The Equinix Metal [m3.large.x86](https://deploy.equinix.com/product/servers/m3-large/) has 2x of Equinix Metal's 3.8TB NVMe drives. The Equinix Metal WO [m3.large.x86-opt-cs2s1](https://deploy.equinix.com/product/servers/m3-large-opt-c2s1/) is essentially the same chassis with 2x additional of the same drives. While the pricing of stock configurations is different from that of WO configurations, we can use this as a starting place to build our model.

If we take the 1-year MRC of the WO instance, and subtract that from the price of the stock instance, we should be left with the price difference between the two, which would functionally be the price difference for the 2x additional NVMe drives: `2,035 - 1,584 = 451`, where **$451** is the Equinix Metal MRC for 2x NVMe drives. Divide that by 2x for the per drive cost, **$226**.

That gets us cost per normal, local chassis drive. To mock the NVMe/TCP product this repository is pitching, we will assume an Equinix Metal "port" costs $60 MRC, and we will assume the cost for an NVMe drive with network port is $10 MRC more expensive than Metal's current NVMe drive. That gives us a per drive cost of $296.

To get the per GB cost most commonly used in Cloud platform pricing, we can divide cost per drive per month by capacity:
`296 / 3800 = 0.07` or **$0.07** cost per GB.

This falls roughly in line or significantly better compard to network attached NVMe / high performance storage products from other Cloud platforms:
- [GCP - $0.17+](https://cloud.google.com/compute/disks-image-pricing#disk)
- [AWS - $0.08+](https://aws.amazon.com/ebs/pricing/)
- [Azure - $0.22+](https://azure.microsoft.com/en-us/pricing/details/managed-disks/)


### Note-able paradigms

#### LACP / Layer-3 Networking

NVMe/TCP places markably little design requirements on the network, expressing signifcantly less network design opinion than say NFS or iSCSI. By default everything with the Metal Layer-3 BE network should work as expected.

Noteably, because each NVMe/TCP target is configured as a seperate IP+Port combination, we get relatively elegant scaling across Equinix Metal instance LACP bonds. As number of drives increases, the number of hashes for the bond will increase, and traffic should scale elegantly.


#### Storage Redundancy / Throughput

In a traditional local RAID storage setup, a CPU only has to send one write instruction to the disk, and the disk controller itself is then responsible for the east <-> west traffic responsible for RAID. This means the CPU / compute does not have to "double write" to achieve parity, instead that action / throughput is done by the storage controller itself.

In this topology, the `mfdh` host is itself a storage controller. Which means for it do do the equivalent of **RAID-1** parity, it will have to double write path volume to 2x or more seperate disks. This write path comes through the shared connectivity over Ethernet, which means for each "write", we have to do "2x writes" through a single data plane, effectively halving the capacity of the dataplane.

Design strategies that avoid full double write in favor of more modern parity strategies such as erasure coding will be minimally impacted by this paradigm.

#### Envisioned Deployment Pattern

The author envisions leveraging Equinix Metal's Open19 form factor for delivery, where an Open19 slot could contain a large number of NVMe/TCP enabled drives. Each drive would essentially just be a "Bare Metal Server", getting it's networking configuration from the same logic structures that give Custom iPXE Metal instance's DHCP leases.

# Documentation for my HomeLab setup v2
This is a writeup of my current homelab setup.
This repository will also link to any related Repos, like templates, scripts, ansible roles, etc. That is, if I have chosen to make them public.

Why Version 2? Because this setup has evolved multiple times and has been rebuilt from the ground up at least once, so I'm not comfortable calling it V1... 😉

## Hardware setup

| Node | CPU | Memory | Storage Drives |
| --------------- | --------------- | --------------- | --------------- |
| 1 | Ryzen 7 2700X | 64GB DDR4 | 1x 120G SSD <br> 1x 1TB NVMe <br> 2x 2TB SATA WD RED SSD |
| 2 | Ryzen 7 5700U | 64GB DDR4 SODIMM | 1x 120G SSD <br> 1x 1TB NVMe |
| 3 | Ryzen 9 6900HX | 64GB DDR4 SODIMM | 1x 120G SSD <br> 1x 1TB NVMe |
| 4 | Ryzen 9 6900HX | 64GB DDR4 SODIMM | 1x 120G SSD <br> 1x 1TB NVMe |
| Backupserver | Intel N100 | 16GB DDR4 SODIMM | 1x 256G SSD <br> 18TB Exos Enterprise HDD |

The first node is a desktop machine, the other ones are mini PCs.

## Hypervisor
All nodes in the cluster run Proxmox VE 9. The nodes are joined into a cluster, which allows for live migration and high availability.
The entire infrastructure is a hyperconverged setup, all nodes have the same storage available and all VMs are distributed across the cluster.

## Networking
> [!NOTE]
> This is just an overview. For details, see [Networking.md](Networking.md)
### Physical Links
Nodes are interconnected using consumer grade 2.5Gbps switches. I have a total of 3 links for each node, one is a cluster link, one upstream WAN and one is a client network downlink. The nodes only have an IP address on the cluster link. The rest is only used for Layer-2 Communication for VMs in the cluster. 

The cluster link manages PVE quorum/cluster communication. It also manages Ceph traffic and SDN traffic.
The WAN link is used by the OPNSense firewall VM for upstream traffic.
The client link is used for routing client traffic through the OPNSense to upstream WAN.

### SDN setup
The SDN is managed via Proxmox VE's built in SDN capabilities.
I am using VXLANs for VM networks. Each one acts as a virtual Layer-2 network.
All VXLANs are attached to an OPNSense firewall. This VM routes all traffic between VXLANs, as well as the WAN and client links.

External Router is an TPLink Omada AX3000 DSL, attached to the WAN link. 

## Storage
### PVE Nodes
Each node boots from the 120GB SSD. It is used by Proxmox VE.
The 1TB NVMe drive is used for Ceph OSDs. Each node has one OSD, which gives us a total of 4 OSDs in the cluster. The CRUSH map is configured for a size of 3 and a minimum size of 2.

All VMs are located on the Ceph Cluster.
Additionally, Ceph is also hosting a CephFS filesystem for ISO files and other snippets.

I also have a 2 SATA SSD ZFS stripe for my NAS on Node 1.

### Backupserver
The backupserver is hosting an SMB share for backups. It is also snapshotting my NAS every hour.

## Management Plane
> [!NOTE]
> This is a very basic overview. For details, see [Management.md](Management.md)

The management plane is accessed via a specific VPN server.
It is one of the VXLANs and has access to most SSH ports in the network, as well as management dashboards etc.

Management of VMs is done via OpenTofu, CloudInit and Ansible. For a VM deployment, I need about 10 lines of YAML, which generates the Tofu HCL, CloudInit Config and Ansible inventory, including the base setup role which handles firewalling, auto-updates, etc.

## Backups
A snapshot of each VM is taken once a day, and pushed to the Backupserver. The NAS is snapshotted hourly.
Once a Month, a backup of all VMs and the NAS is pushed to a remote Hetzner server using Restic, which is end to end encrypted.

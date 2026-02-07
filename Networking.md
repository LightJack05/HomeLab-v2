# Networking setup
My networking consists of Proxmox VE's bultin SDN capabilities.

The graphic below shows the logical and physical network setup.

<!-- TODO: Add graphic -->

## Physical Links
As you can see, there are 3 physical links leaving each server.
### Cluster Link

The cluster link connects all nodes to each other and serves:
- Quorum and corosync communication for Proxmox VE 
- Ceph cluster communication
- SDN communication for VXLANs

Physically, this is a 2.5Gbps link connected to a dedicated Zyxel 2.5Gbps switch.

This link also serves as a physical backup link with access to all nodes, as well as the OPNSense firewall VM.

### WAN Link
This is the upstream link for the OPNSense firewall VM. All internet traffic is routed through it. 
Since the upstream router only supports 1Gbps, this is a 1Gbps link connected to a consumer grade switch.
This link is mapped in Proxmox as a bridge, allowing the OPNSense to fail over to any node, while still having access to the WAN.

### Client Link
This is a downstream link for client traffic. It is a mixed 10Gbps/2.5Gbps link, connected to a QNAP switch. There is a direct 10Gbps link from my Desktop to node 1, for high throughput NAS traffic.
There is also an access point in this network for WiFi clients.
The gateway of this network is, as with any other VXLAN, the OPNSense firewall VM.

## Firewall and Routing
### Routing
Since all networks are attached directly to the OPNSense firewall, the routing setup is quite simple. It only needs an upstream gateway to the WAN.
In the WAN gateway, the networks attached to the OPNSense are defined as static routes via OPNSense.

### Firewall
Firewall policies are set up in a way that allows me to quickly add common rules for VMs.
For example, there is a Group called 'AllowHttpFromWAN' which is bound to a rule allowing HTTP traffic from WAN to the IP addresses listed in the gorup. This means adding a new HTTP server is done by simply adding the IP to the group. Similarly, there are groups for AllowSSHFromManagement, AllowHTTPFromManagement, etc.
While this seems like just a little convenience, it substantially reduces the number of rules in the firewall, as well as the time it takes to add a new server and even to debug connection issues.

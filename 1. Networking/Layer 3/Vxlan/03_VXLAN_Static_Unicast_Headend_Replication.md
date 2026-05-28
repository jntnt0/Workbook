Static ingress-replication syntax is verified against Cisco’s Nexus VXLAN design guide, which describes IR as unicast BUM replication and shows ingress-replication protocol static with peer-ip <remote vtep IP>.  

VXLAN_Static_Unicast_Headend_Replication.md

VXLAN_Static_Unicast_Headend_Replication

# VXLAN_Static_Unicast_Headend_Replication_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Static unicast headend replication | The local VTEP manually replicates BUM traffic as unicast VXLAN packets to every configured remote VTEP peer |
| BUM traffic | Broadcast, unknown unicast, and multicast traffic that must be delivered to all VTEPs participating in the same L2VNI |
| Headend replication point | The ingress VTEP does the packet copying before sending traffic into the underlay |
| No multicast dependency | Static ingress replication does not require PIM, RP, multicast groups, or multicast routing in the underlay |
| Underlay dependency | Every VTEP still needs normal unicast IP reachability to every remote VTEP loopback |
| Manual peer list | Each VNI must have the correct remote `peer-ip` list on every participating VTEP |
| Symmetry requirement | If Leaf1 lists Leaf2 but Leaf2 does not list Leaf1, flooding and learning can become one-way |
| Data-plane MAC learning | Static ingress replication does not create an EVPN control plane; MAC learning still depends on endpoint traffic |
| Scale limit | Works well for small labs and simple designs, but manual peer updates become painful as leaves and VNIs grow |
| Related labs | `vxlan-unicast-final` |
# VXLAN_Static_Unicast_Headend_Replication_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the underlay is already routed and stable | Leaf VTEPs | `show ip route` | Each leaf has underlay routes before VXLAN service configuration starts |
| 2 | Confirm local VTEP loopback exists | Leaf VTEPs | `show ip interface brief vrf all` | Local VTEP source loopback is up/up with the expected /32 address |
| 3 | Confirm remote VTEP loopback reachability | Leaf VTEPs | `ping <REMOTE_VTEP_IP> source <LOCAL_VTEP_IP>` | Each VTEP can reach every remote VTEP source address using unicast routing |
| 4 | Confirm multicast is not required for this replication model | Leaf VTEPs | `show running-config interface nve1` | The target L2VNI will use static ingress replication, not `mcast-group` |
| 5 | Enable VLAN-to-VNI support | Leaf VTEPs | `feature vn-segment-vlan-based` | NX-OS accepts `vn-segment` under VLAN configuration mode |
| 6 | Enable VXLAN NVE support | Leaf VTEPs | `feature nv overlay` | NX-OS accepts `interface nve1` configuration |
| 7 | Create the local VLAN | Leaf VTEPs | `vlan <VLAN_ID>` | VLAN exists locally on the VTEP |
| 8 | Map the VLAN to the L2VNI | Leaf VTEPs | `vn-segment <L2VNI>` | Local VLAN is bound to the intended overlay segment |
| 9 | Attach host-facing access port to the VLAN | Leaf VTEPs | `interface <HOST_PORT>` then `switchport mode access` then `switchport access vlan <VLAN_ID>` | Local endpoint is placed into the stretched VLAN |
| 10 | Create or enter the NVE interface | Leaf VTEPs | `interface nve1` | NVE interface configuration mode opens |
| 11 | Bind NVE to the local VTEP loopback | Leaf VTEPs | `source-interface loopback<VTEP_LOOPBACK_ID>` | VXLAN packets source from the correct VTEP loopback |
| 12 | Add the L2VNI under NVE | Leaf VTEPs | `member vni <L2VNI>` | L2VNI service configuration mode opens under `nve1` |
| 13 | Configure static ingress replication for the L2VNI | Leaf VTEPs | `ingress-replication protocol static` | The VTEP uses manual unicast replication instead of multicast or BGP EVPN |
| 14 | Add the remote VTEP peer for this L2VNI | Leaf VTEPs | `peer-ip <REMOTE_VTEP_IP>` | BUM traffic for this VNI is replicated to the listed remote VTEP |
| 15 | Repeat `peer-ip` for every remote VTEP that participates in the same L2VNI | Leaf VTEPs | `peer-ip <REMOTE_VTEP_IP>` | Local VTEP has a complete static replication list |
| 16 | Configure the reverse peer list on every other VTEP | Remote Leaf VTEPs | `peer-ip <LOCAL_VTEP_IP>` | Replication is symmetric across all VTEPs in the L2VNI |
| 17 | Enable the NVE interface | Leaf VTEPs | `interface nve1` then `no shutdown` | NVE interface is administratively enabled |
| 18 | Verify NVE interface state | Leaf VTEPs | `show nve interface` | `nve1` is up and sourced from the expected loopback |
| 19 | Verify the L2VNI is active | Leaf VTEPs | `show nve vni` | L2VNI appears as up under `nve1` |
| 20 | Verify static NVE peer state | Leaf VTEPs | `show nve peers` | Remote VTEP peer appears for the static replication relationship |
| 21 | Verify VLAN-to-VNI mapping | Leaf VTEPs | `show vxlan` | VLAN maps to the expected L2VNI |
| 22 | Generate endpoint traffic | Hosts | `ping <REMOTE_HOST_IP>` | ARP and ICMP traffic trigger VXLAN encapsulation and MAC learning |
| 23 | Verify MAC learning after traffic | Leaf VTEPs | `show mac address-table dynamic vlan <VLAN_ID>` | Local and remote MAC addresses appear for the stretched VLAN |
| 24 | Verify no multicast troubleshooting is needed | Leaf VTEPs | `show running-config interface nve1` | The L2VNI uses `ingress-replication protocol static`; PIM/RP checks are out of scope |
# VXLAN_Static_Unicast_Headend_Replication_Skeleton
# LEAF1 STATIC UNICAST VXLAN
conf t
feature vn-segment-vlan-based
feature nv overlay
vlan <VLAN_ID>
  name <SEGMENT_NAME>
  vn-segment <L2VNI>
interface <HOST_PORT>
  description TO_<LOCAL_HOST>
  switchport
  switchport mode access
  switchport access vlan <VLAN_ID>
  no shutdown
interface nve1
  source-interface loopback<VTEP_LOOPBACK_ID>
  member vni <L2VNI>
    ingress-replication protocol static
    peer-ip <REMOTE_LEAF2_VTEP_IP>
  no shutdown
end
# LEAF2 STATIC UNICAST VXLAN
conf t
feature vn-segment-vlan-based
feature nv overlay
vlan <VLAN_ID>
  name <SEGMENT_NAME>
  vn-segment <L2VNI>
interface <HOST_PORT>
  description TO_<LOCAL_HOST>
  switchport
  switchport mode access
  switchport access vlan <VLAN_ID>
  no shutdown
interface nve1
  source-interface loopback<VTEP_LOOPBACK_ID>
  member vni <L2VNI>
    ingress-replication protocol static
    peer-ip <REMOTE_LEAF1_VTEP_IP>
  no shutdown
end
# MULTIPLE REMOTE VTEPS ON ONE L2VNI
conf t
interface nve1
  member vni <L2VNI>
    ingress-replication protocol static
    peer-ip <REMOTE_LEAF2_VTEP_IP>
    peer-ip <REMOTE_LEAF3_VTEP_IP>
    peer-ip <REMOTE_LEAF4_VTEP_IP>
end
# DO NOT MIX THESE ON THE SAME L2VNI
# member vni <L2VNI> mcast-group <GROUP>
# member vni <L2VNI>
#   ingress-replication protocol static
# VXLAN_Static_Unicast_Headend_Replication_Verification_Commands
# Underlay unicast reachability
show ip route <REMOTE_VTEP_IP>
ping <REMOTE_VTEP_IP> source <LOCAL_VTEP_IP>
traceroute <REMOTE_VTEP_IP> source <LOCAL_VTEP_IP>
# Feature state
show feature | include nv
show feature | include vn-segment
# VLAN-to-VNI mapping
show vlan id <VLAN_ID>
show vxlan
show running-config vlan <VLAN_ID>
# NVE configuration and status
show running-config interface nve1
show nve interface
show nve vni
show nve vni <L2VNI>
show nve peers
show nve peers detail
# Host-facing access or trunk state
show running-config interface <HOST_PORT>
show interface <HOST_PORT> switchport
show interface trunk
# MAC learning
show mac address-table dynamic vlan <VLAN_ID>
show mac address-table address <HOST_MAC>
# Endpoint traffic test
ping <REMOTE_HOST_IP>
arp -a
# Confirm multicast is not part of this design
show running-config interface nve1 | include mcast-group
show ip pim neighbor
# VXLAN_Static_Unicast_Headend_Replication_Rollback
# REMOVE ONE STATIC PEER FROM ONE L2VNI
conf t
interface nve1
  member vni <L2VNI>
    no peer-ip <REMOTE_VTEP_IP>
end
# REMOVE STATIC INGRESS REPLICATION FROM THE L2VNI
conf t
interface nve1
  member vni <L2VNI>
    no ingress-replication protocol static
end
# REMOVE THE L2VNI FROM NVE
conf t
interface nve1
  no member vni <L2VNI>
end
# REMOVE VLAN-TO-VNI MAPPING BUT KEEP THE VLAN
conf t
vlan <VLAN_ID>
  no vn-segment <L2VNI>
end
# REMOVE LOCAL VLAN
conf t
no vlan <VLAN_ID>
end
# SHUT DOWN NVE WITHOUT REMOVING ALL CONFIGURATION
conf t
interface nve1
  shutdown
end
# REMOVE NVE COMPLETELY
conf t
interface nve1
  shutdown
  no member vni <L2VNI>
  no source-interface loopback<VTEP_LOOPBACK_ID>
exit
no interface nve1
end
# REMOVE VXLAN FEATURES ONLY IF NO OTHER VXLAN SERVICES EXIST
conf t
no feature nv overlay
no feature vn-segment-vlan-based
end
# VXLAN_Static_Unicast_Headend_Replication_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| `peer-ip` is configured but NVE peer does not come up | Remote VTEP loopback is not reachable | `ping <REMOTE_VTEP_IP> source <LOCAL_VTEP_IP>` | Fix underlay unicast routing to the remote VTEP loopback |
| L2VNI does not appear under NVE | VNI is mapped to VLAN but not configured under `interface nve1` | `show running-config interface nve1`; `show nve vni` | Add `member vni <L2VNI>` under `interface nve1` |
| L2VNI stays down | NVE is shut, source loopback is down, or replication method is missing | `show nve interface`; `show nve vni` | Apply `no shutdown`, fix source loopback, and configure `ingress-replication protocol static` |
| One direction works, reverse direction fails | Static peer list is missing on one VTEP | `show running-config interface nve1` on both leaves | Configure reciprocal `peer-ip` entries on every VTEP in the same L2VNI |
| ARP request leaves one VTEP but remote host never sees it | Remote VTEP is missing from the static replication list | `show running-config interface nve1`; `show nve peers` | Add the correct `peer-ip <REMOTE_VTEP_IP>` under the L2VNI |
| Remote MACs are not learned | No endpoint traffic, bad peer list, VLAN/VNI mismatch, or local host-facing VLAN issue | `show mac address-table dynamic vlan <VLAN_ID>`; `show vxlan`; `show nve peers` | Generate traffic, fix VLAN/VNI mapping, and correct peer list |
| Local MACs are not learned | Host-facing access or trunk configuration is wrong | `show interface <HOST_PORT> switchport`; `show vlan id <VLAN_ID>` | Correct access VLAN or trunk allowed VLAN configuration |
| `show vxlan` shows wrong VNI | VLAN was mapped to the wrong `vn-segment` | `show running-config vlan <VLAN_ID>` | Correct `vn-segment <L2VNI>` under the VLAN |
| Static IR was configured alongside multicast group | Replication methods were mixed for the same L2VNI | `show running-config interface nve1` | Use either `ingress-replication protocol static` or `mcast-group`, not both for the same L2VNI |
| PIM neighbors are missing | Not a fault for this note if static ingress replication is the intended design | `show running-config interface nve1` | Ignore PIM for static IR unless another VNI uses multicast flood-and-learn |
| Peer exists but no encapsulated traffic counters increase | No traffic, wrong endpoint VLAN, or peer is for the wrong VNI | `show nve peers detail`; `show mac address-table dynamic vlan <VLAN_ID>` | Verify host VLANs, generate ARP/ICMP traffic, and confirm peer is under the correct `member vni` |
| Large fabric becomes hard to maintain | Static peer lists do not scale | `show running-config interface nve1` across all leaves | Move to BGP EVPN ingress replication for dynamic peer discovery |
##### Source_Basis
# VXLAN_Static_Unicast_Headend_Replication_Mental_Model
# VXLAN_Static_Unicast_Headend_Replication_Configuration_Checklist
# VXLAN_Static_Unicast_Headend_Replication_Skeleton
# VXLAN_Static_Unicast_Headend_Replication_Verification_Commands
# VXLAN_Static_Unicast_Headend_Replication_Rollback
# VXLAN_Static_Unicast_Headend_Replication_Failure_Checks
# index of each title throughout note, not in table format


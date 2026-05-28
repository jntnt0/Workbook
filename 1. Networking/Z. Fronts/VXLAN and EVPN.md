VXLAN_EVPN_Note_Order.md

VXLAN_EVPN_Note_Order

# Source_Basis

| Source | Relevant Section | What It Supports |

|---|---|---|

| `_KB_INDEX.md` | Data Center VXLAN EVPN / NX-OS VXLAN flood-and-learn index entries | Points VXLAN EVPN to `CiscoPress_combined_part1.md`, flood-and-learn syntax to `NS-OS_combined_PDFs.md`, and EVPN consistency checks to `NS-XO_combined_epubs.md` |

| `CiscoPress_combined_part1.md` | DCCOR Ch 3, Implementing Data Center Overlay Protocols | VXLAN overview, VTEP, VNI, flood-and-learn, MP-BGP EVPN control plane, NVE verification |

| `NS-OS_combined_PDFs.md` | VXLAN Flood-and-Learn article, lines 11,612-12,599 | NX-OS flood-and-learn configuration and verification syntax |

| `NS-XO_combined_epubs.md` | VXLAN EVPN Configuration Consistency Checker | Correct EVPN operational checks: NVE loopback rules, replication method, ARP suppression constraints, L3VNI association checks |

| `CiscoPress_combined_part2.md` | L2VPN EVPN service deployment sections | EVPN route-type mental model and `show bgp l2vpn evpn summary` verification framing |

# VXLAN_EVPN_Mechanism_Note_Order

| Order | Mechanism Note | Mental Model That Must Be Addressed | Related Lab Names |

|---:|---|---|---|

| 1 | `VXLAN_Underlay_Leaf_Spine_And_VTEP_Reachability.md` | The underlay is just routed IP transport. Spines forward outer IP packets. Leaves/VTEPs source and terminate VXLAN. Overlay failure is often just broken loopback reachability, IGP, ECMP, PIM, or MTU underneath. | `vxlan-one-spine-two-leafs-final`; `vxlan-l2vpn-bgp-evpn-two-spines-four-leaves-final`; `vxlan-multicast-flood-and-learn-final` |

| 2 | `VXLAN_NVE_L2VNI_VLAN_Bridge_Domain_Mapping.md` | VLAN is local access segmentation. VNI is the overlay segment. `interface nve1` binds the source loopback to the VNI. The leaf bridges local VLAN frames into VXLAN by VLAN-to-VNI mapping. | `vxlan-one-spine-two-leafs-final`; `vxlan-unicast-final`; `vxlan-multicast-flood-and-learn-final`; `vxlan-l2vpn-bgp-evpn-two-spines-four-leaves-final` |

| 3 | `VXLAN_Static_Unicast_Headend_Replication.md` | BUM replication is manual unicast replication. No multicast tree and no EVPN control plane. It is useful for learning the mechanism, but it does not scale cleanly. | `vxlan-unicast-final` |

| 4 | `VXLAN_Multicast_Flood_And_Learn.md` | VTEPs use multicast groups for BUM flooding. Remote MAC and VTEP reachability are learned in the data plane after traffic starts. NVE peers can be absent until endpoint traffic triggers learning. | `vxlan-multicast-flood-and-learn-final` |

| 5 | `VXLAN_BGP_EVPN_L2VNI_Control_Plane.md` | EVPN replaces pure data-plane learning with BGP control-plane learning. Leaves advertise MAC/IP reachability through L2VPN EVPN. Spines usually act as route reflectors. Remote VTEPs are learned from BGP, not from blind flooding. | `vxlan-l2vpn-bgp-evpn-two-spines-four-leaves-final` |

| 6 | `VXLAN_EVPN_Route_Reflector_And_Fabric_Scaleout.md` | A two-spine, four-leaf fabric is mainly a scale and control-plane lesson. Spines reflect EVPN routes; leaves hold endpoint state. The spine should not be treated as an endpoint gateway unless explicitly configured as a VTEP. | `vxlan-l2vpn-bgp-evpn-two-spines-four-leaves-final` |

| 7 | `VXLAN_EVPN_ARP_Suppression_And_Anycast_Gateway.md` | ARP suppression is an EVPN optimization, not a base VXLAN feature. It depends on MAC/IP bindings learned through EVPN. `suppress-arp` belongs on L2VNIs only where the extended VLAN SVI is using fabric anycast gateway behavior. | `vxlan-bgp-evpn-arp-suppression-final` |

# Correct_Mental_Model_Coverage

| Mental Model | Must Be Covered In | Why It Matters |

|---|---|---|

| Underlay vs overlay separation | Notes 1, 2 | Prevents confusing tenant traffic problems with transport reachability problems |

| VTEP role | Notes 1, 2 | The VTEP is the VXLAN edge. Spines normally just route outer IP packets |

| VLAN-to-VNI mapping | Note 2 | This is the bridge between local access VLANs and overlay segmentation |

| NVE source-interface | Notes 2, 3, 4, 5 | The loopback source is the VTEP identity used for encapsulation and remote peer discovery |

| BUM replication choices | Notes 3, 4, 5 | Static unicast, multicast flood-and-learn, and EVPN-based replication are different mechanisms |

| Data-plane learning | Notes 3, 4 | Flood-and-learn builds state after traffic appears |

| Control-plane learning | Notes 5, 6 | EVPN distributes endpoint reachability before relying on blind flooding |

| EVPN route types | Notes 5, 6, 7 | Route Type 2 explains MAC/IP advertisement; Route Type 3 explains inclusive multicast/replication membership; Route Type 5 only belongs if the lab includes L3VNI/IP prefix routing |

| ARP suppression | Note 7 | ARP suppression should be taught after EVPN MAC/IP learning, not before |

| Verification order | All notes | Check underlay first, NVE second, EVPN third, endpoint tables fourth, traffic last |

# Verification_Syntax_Anchors

| Mechanism Area | Core Commands |

|---|---|

| Underlay reachability | `show ip route`; `show ip ospf neighbor`; `show isis adjacency`; `ping <remote-vtep-loopback> source <local-loopback>` |

| VXLAN feature and VNI mapping | `show vxlan`; `show nve interface`; `show nve vni` |

| NVE peers | `show nve peers`; `show nve peers detail` |

| Flood-and-learn multicast | `show ip pim neighbor`; `show ip pim rp`; `show ip mroute <multicast-group>` |

| Data-plane MAC learning | `show mac address-table dynamic vlan <vlan-id>` |

| EVPN control plane | `show bgp l2vpn evpn summary`; `show bgp l2vpn evpn` |

| vPC/VTEP consistency, if used | `show vpc`; `show vpc consistency-parameters global` |

# Lab_Attachment_Map

| Lab Name | Primary Note | Secondary Notes |

|---|---|---|

| `vxlan-one-spine-two-leafs-final` | `VXLAN_Underlay_Leaf_Spine_And_VTEP_Reachability.md` | `VXLAN_NVE_L2VNI_VLAN_Bridge_Domain_Mapping.md` |

| `vxlan-unicast-final` | `VXLAN_Static_Unicast_Headend_Replication.md` | `VXLAN_NVE_L2VNI_VLAN_Bridge_Domain_Mapping.md` |

| `vxlan-multicast-flood-and-learn-final` | `VXLAN_Multicast_Flood_And_Learn.md` | `VXLAN_Underlay_Leaf_Spine_And_VTEP_Reachability.md`; `VXLAN_NVE_L2VNI_VLAN_Bridge_Domain_Mapping.md` |

| `vxlan-l2vpn-bgp-evpn-two-spines-four-leaves-final` | `VXLAN_BGP_EVPN_L2VNI_Control_Plane.md` | `VXLAN_EVPN_Route_Reflector_And_Fabric_Scaleout.md`; `VXLAN_Underlay_Leaf_Spine_And_VTEP_Reachability.md`; `VXLAN_NVE_L2VNI_VLAN_Bridge_Domain_Mapping.md` |

| `vxlan-bgp-evpn-arp-suppression-final` | `VXLAN_EVPN_ARP_Suppression_And_Anycast_Gateway.md` | `VXLAN_BGP_EVPN_L2VNI_Control_Plane.md` |

# Notes_Not_To_Overbuild

| Topic | Keep Out Unless The Lab Actually Uses It |

|---|---|

| L3VNI and Route Type 5 | Do not force it into the L2VNI EVPN note unless the lab routes between VNIs or advertises IP prefixes |

| vPC Anycast VTEP | Only include if the lab has vPC pairs using shared secondary loopback VTEP IP |

| EVPN Multisite | Not part of these five lab names |

| Tenant Routed Multicast | Not part of these five lab names |

| ACI | VXLAN is shared terminology, but these labs are NX-OS style VXLAN/EVPN, not ACI policy modeling |

##### Source_Basis

# VXLAN_Underlay_Leaf_Spine_And_VTEP_Reachability

# VXLAN_NVE_L2VNI_VLAN_Bridge_Domain_Mapping

# VXLAN_Static_Unicast_Headend_Replication

# VXLAN_Multicast_Flood_And_Learn

# VXLAN_BGP_EVPN_L2VNI_Control_Plane

# VXLAN_EVPN_Route_Reflector_And_Fabric_Scaleout

# VXLAN_EVPN_ARP_Suppression_And_Anycast_Gateway

# Correct_Mental_Model_Coverage

# Verification_Syntax_Anchors

# Lab_Attachment_Map

# Notes_Not_To_Overbuild
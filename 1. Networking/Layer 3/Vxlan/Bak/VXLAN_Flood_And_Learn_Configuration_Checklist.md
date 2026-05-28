answers the question: In what exact order do I execute commands and verify the network state?

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm platform role | All switches | `show version` | Nexus/NX-OS platform is confirmed |
| 2 | Verify physical underlay interfaces | Spine / Core / VTEP | `show ip interface brief` | Underlay-facing interfaces show `up/up` |
| 3 | Verify underlay routing exists | Spine / Core / VTEP | `show ip route` | VTEP loopbacks and spine/core loopbacks are reachable |
| 4 | Enable PIM feature | Spine / Core / VTEP | `feature pim` | PIM feature is enabled |
| 5 | Configure RP address | Spine / Core / VTEP | `ip pim rp-address <rp-ip>` | Devices know the multicast rendezvous point |
| 6 | Configure Anycast RP member, if used | Spine / Core | `ip pim anycast-rp <anycast-rp-ip> <rp-member-ip>` | RP redundancy is configured |
| 7 | Enable PIM on underlay routed interfaces | Spine / Core / VTEP | `interface <underlay-interface>` then `ip pim sparse-mode` | PIM is active on routed underlay links |
| 8 | Enable PIM on VTEP loopback | VTEP | `interface loopback0` then `ip pim sparse-mode` | VTEP loopback participates in multicast underlay |
| 9 | Verify PIM neighbors | Spine / Core / VTEP | `show ip pim neighbor` | PIM neighbors are formed across underlay links |
| 10 | Verify RP information | Spine / Core / VTEP | `show ip pim rp` | Correct RP is shown for multicast group range |
| 11 | Enable VLAN-to-VNI feature | VTEP | `feature vn-segment-vlan-based` | NX-OS can map VLANs to VXLAN VNIs |
| 12 | Enable VXLAN overlay feature | VTEP | `feature nv overlay` | NVE/VXLAN overlay support is enabled |
| 13 | Create tenant VLAN | VTEP | `vlan <vlan-id>` | VLAN exists locally |
| 14 | Map VLAN to VNI | VTEP | `vn-segment <vni-id>` | VLAN is bound to the VXLAN segment |
| 15 | Repeat VLAN-to-VNI mapping for additional segments | VTEP | `vlan <vlan-id>` then `vn-segment <vni-id>` | Each VLAN has the intended VNI |
| 16 | Configure VTEP source loopback | VTEP | `interface loopback0` then `ip address <vtep-loopback-ip>/32` | Loopback exists as the local VTEP source |
| 17 | Advertise VTEP loopback into underlay | VTEP | `ip router ospf <process-id> area <area-id>` | VTEP loopback is reachable through the underlay |
| 18 | Enable PIM on VTEP source loopback | VTEP | `interface loopback0` then `ip pim sparse-mode` | VTEP source can participate in multicast forwarding |
| 19 | Enter NVE interface | VTEP | `interface nve1` | NVE interface configuration mode is entered |
| 20 | Set NVE source interface | VTEP | `source-interface loopback0` | NVE uses loopback0 as the VTEP source |
| 21 | Add L2 VNI to multicast group | VTEP | `member vni <vni-id> mcast-group <multicast-group>` | VNI is mapped to the multicast group |
| 22 | Add additional VNIs to the same or different multicast group | VTEP | `member vni <vni-id> mcast-group <multicast-group>` | Additional VXLAN segments are enabled |
| 23 | Bring NVE interface online | VTEP | `no shutdown` | NVE interface is administratively up |
| 24 | Verify VLAN-to-VNI mapping | VTEP | `show vxlan` | VLANs show the expected VN-segment values |
| 25 | Verify NVE VNI state | VTEP | `show nve vni` | VNI state shows `Up`, mode `DP`, type `L2` |
| 26 | Verify multicast route for VXLAN group | VTEP / Core | `show ip mroute <multicast-group>` | Multicast entries exist for the VXLAN group |
| 27 | Verify NVE interface state | VTEP | `show nve interface` | `nve1` shows `Up`, encapsulation `VXLAN`, source loopback listed |
| 28 | Check NVE peers before traffic | VTEP | `show nve peers` | In flood-and-learn, peers may be empty before host traffic starts |
| 29 | Generate host traffic across VXLAN | Host / VTEP | `ping <remote-host-ip>` | Traffic triggers data-plane learning |
| 30 | Verify NVE peer after traffic | VTEP | `show nve peers` | Remote VTEP peer appears as `Up`, learn type `DP` |
| 31 | Verify detailed NVE peer state | VTEP | `show nve peers detail` | Peer shows configured VNI and provision state `add-complete` |
| 32 | Verify VNI details | VTEP | `show nve vni <vni-id> detail` | VNI shows multicast group, state `Up`, mode `data-plane`, type `L2` |
| 33 | Verify internal VNI readiness | VTEP | `show nve internal vni <vni-id>` | Ready state shows L2 VNI flood-and-learn ready |
| 34 | Verify local and remote MAC learning | VTEP | `show mac address-table dynamic` | Local MAC appears on access/vPC port and remote MAC appears through `nve1(<peer-ip>)` |
| 35 | Verify end-to-end reachability | Host / VTEP | `ping <remote-host-ip>` | Ping succeeds across VXLAN segment |
| 36 | Verify multicast core is not acting as VTEP | Spine / Core | `show ip mroute <multicast-group>` | Core shows multicast forwarding entries, not NVE VNI state |
| 37 | Save final configuration | VTEP / Core | `copy running-config startup-config` | Configuration is saved |
| 38 | Final validation pass | VTEP | `show nve vni`, `show nve peers`, `show vxlan`, `show ip mroute <multicast-group>` | VNI, peer, VLAN-to-VNI, and multicast state all match design |


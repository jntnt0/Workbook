
DMVPN_NHRP_Multicast_PIM_Sparse_Mode.md

# DMVPN_NHRP_Multicast_PIM_Sparse_Mode_Configuration_Checklist

# Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 6, Chapter 5, DMVPN | Points DMVPN Phase 1, Phase 2, Phase 3, NHRP, mGRE, and spoke-to-spoke behavior to `All_combined_part3.md` |
| `_KB_INDEX.md` | Book 6, Chapter 14, Multicast | Supports PIM-SM, PIM-SSM, IGMP, RP, Auto-RP, BSR, multicast route state, and multicast verification |
| `All_combined_part3.md` | Book 6, Chapter 5, DMVPN | Supports NHRP multicast mapping, `ip nhrp map multicast dynamic`, spoke multicast mapping, and DMVPN phase behavior |
| `All_combined_part3.md` | Book 6, Chapter 14, Multicast | Supports `ip multicast-routing`, `ip pim sparse-mode`, `ip pim rp-address`, `show ip pim neighbor`, `show ip pim rp mapping`, `show ip igmp groups`, and `show ip mroute` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | DMVPN Tunnels | Supports modern DMVPN verification using `show dmvpn`, `show ip nhrp`, tunnel state, and routing checks before multicast is layered on |

# DMVPN_NHRP_Multicast_PIM_Sparse_Mode_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Multicast over DMVPN | Multicast control and data traffic ride across an already-working DMVPN overlay |
| DMVPN first | PIM does not fix broken underlay, mGRE, tunnel source, or NHRP |
| NHRP multicast mapping | Lets multicast packets cross an mGRE/NHRP DMVPN cloud |
| Hub multicast role | Hub usually uses `ip nhrp map multicast dynamic` to support multicast from registered spokes |
| Spoke multicast role | Spokes map multicast traffic toward the hub NBMA address |
| PIM sparse mode | Builds multicast trees only where receivers join, using an RP |
| RP | Rendezvous Point used by PIM-SM for shared tree discovery |
| RP reachability | Every multicast router must have unicast reachability to the RP address |
| Source reachability | Every multicast router must have valid unicast reachability toward the multicast source for RPF |
| IGMP | Receiver-side host signaling used to join multicast groups |
| RPF | Reverse Path Forwarding check that validates multicast traffic arrives on the correct interface |
| OIL | Outgoing Interface List showing where multicast traffic is forwarded |
| Phase 3 note | NHRP redirect and shortcut optimize unicast forwarding, but multicast still depends on PIM, RP, IGMP, RPF, and NHRP multicast mapping |
| Hard rule | If PIM neighbors do not form on the tunnel, do not troubleshoot receivers yet |

# DMVPN_NHRP_Multicast_PIM_Sparse_Mode_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm NBMA reachability first | Hub, Spokes | `ping <remote-nbma-ip>` | Underlay transport works before multicast is configured |
| 2 | Confirm tunnel interface state | Hub, Spokes | `show ip interface brief` | Tunnel interfaces show `up/up` |
| 3 | Confirm DMVPN peer state | Hub, Spokes | `show dmvpn` | DMVPN peers show up |
| 4 | Confirm NHRP mappings | Hub, Spokes | `show ip nhrp` | Hub has spokes and spokes have hub mapping |
| 5 | Confirm unicast routing works across overlay | Hub, Spokes | `show ip route` | Source, receiver, and RP networks are reachable |
| 6 | Confirm hub NHRP multicast mapping | Hub | `show running-config interface Tunnel<id>` | Hub has multicast mapping for DMVPN |
| 7 | Confirm spoke multicast mapping to hub | Spokes | `show running-config interface Tunnel<id>` | Spokes map multicast toward hub NBMA |
| 8 | Confirm Phase 3 commands if this is a Phase 3 multicast lab | Hub, Spokes | `show running-config interface Tunnel<id>` | Hub has redirect and spokes have shortcut if Phase 3 is used |
| 9 | Choose RP address | All Routers | Design check | RP address is stable and reachable |
| 10 | Confirm route to RP | All Routers | `show ip route <rp-ip>` | RP address is reachable by unicast |
| 11 | Confirm route to multicast source | All Routers | `show ip route <source-ip>` | Source address is reachable by unicast |
| 12 | Enable multicast routing globally | Hub, Spokes | See `Global_Multicast_Enablement` | Router can run multicast routing |
| 13 | Enable PIM sparse mode on hub tunnel | Hub | See `Hub_PIM_SM_Tunnel_Config` | Hub tunnel participates in PIM-SM |
| 14 | Enable PIM sparse mode on spoke tunnels | Spokes | See `Spoke_PIM_SM_Tunnel_Config` | Spoke tunnels participate in PIM-SM |
| 15 | Enable PIM on source-facing interface | Source-side Router | See `Source_Interface_PIM_Config` | Source traffic can enter the multicast domain |
| 16 | Enable PIM on receiver-facing interface | Receiver-side Router | See `Receiver_Interface_PIM_Config` | Receiver joins can build multicast state |
| 17 | Enable PIM on RP loopback if RP is local | RP Router | See `RP_Loopback_Config` | RP loopback participates correctly |
| 18 | Configure static RP on hub and spokes | Hub, Spokes | See `Static_RP_Config` | All multicast routers know the RP |
| 19 | Optionally restrict RP to selected groups | Hub, Spokes | See `Static_RP_Group_Filter_Option` | RP applies only to intended multicast groups |
| 20 | Verify PIM interfaces | Hub, Spokes | `show ip pim interface` | Tunnel and required LAN interfaces appear |
| 21 | Verify PIM neighbors over DMVPN | Hub, Spokes | `show ip pim neighbor` | PIM neighbors form over Tunnel `<id>` |
| 22 | Verify RP mapping | Hub, Spokes | `show ip pim rp mapping` | Correct RP is mapped to expected group range |
| 23 | Create test receiver join if needed | Receiver-side Router | See `Test_IGMP_Join_Config` | Router joins the test multicast group |
| 24 | Verify IGMP group membership | Receiver-side Router | `show ip igmp groups` | Receiver interface shows group membership |
| 25 | Generate multicast test traffic | Source-side Router | See `Multicast_Test_Traffic` | Source sends traffic to multicast group |
| 26 | Verify source-side multicast route state | Source-side Router | `show ip mroute <group>` | Multicast state exists for the group |
| 27 | Verify hub multicast route state | Hub | `show ip mroute <group>` | Hub shows correct incoming and outgoing interfaces |
| 28 | Verify receiver-side multicast route state | Receiver-side Router | `show ip mroute <group>` | Receiver-side OIL points toward receiver interface |
| 29 | Verify RPF toward RP | Hub, Spokes | `show ip rpf <rp-ip>` | RPF interface points toward the RP |
| 30 | Verify RPF toward source | Hub, Spokes | `show ip rpf <source-ip>` | RPF interface points toward the source |
| 31 | Verify DMVPN tunnel appears in multicast forwarding where expected | Hub, Spokes | `show ip mroute <group>` | Tunnel appears in IIF or OIL as expected |
| 32 | Verify receiver LAN appears in OIL | Receiver-side Router | `show ip mroute <group>` | Receiver-facing interface appears in outgoing list |
| 33 | Remove test IGMP join if it was only for validation | Receiver-side Router | See `Remove_Test_IGMP_Join` | Test receiver membership is removed |
| 34 | Verify multicast state ages out if no receivers remain | Receiver-side Router | `show ip igmp groups` and `show ip mroute <group>` | Group state disappears or begins aging |
| 35 | Save working multicast-over-DMVPN state | Hub, Spokes | `copy running-config startup-config` | Configuration is saved |

# DMVPN_NHRP_Multicast_PIM_Sparse_Mode_Skeleton

```text
# Hub_NHRP_Multicast_Config
conf t
interface Tunnel<id>
 ip nhrp map multicast dynamic
end
```

```text
# Spoke_NHRP_Multicast_Compact_NHS_Config
conf t
interface Tunnel<id>
 ip nhrp nhs <hub-tunnel-ip> nbma <hub-nbma-ip> multicast
end
```

```text
# Spoke_NHRP_Multicast_Separate_Mapping_Config
conf t
interface Tunnel<id>
 ip nhrp map <hub-tunnel-ip> <hub-nbma-ip>
 ip nhrp map multicast <hub-nbma-ip>
 ip nhrp nhs <hub-tunnel-ip>
end
```

```text
# Optional_Phase3_NHRP_Config
! Hub
conf t
interface Tunnel<id>
 ip nhrp redirect
end

! Spokes
conf t
interface Tunnel<id>
 ip nhrp shortcut
end
```

```text
# Global_Multicast_Enablement
conf t
ip multicast-routing
end
```

```text
# Hub_PIM_SM_Tunnel_Config
conf t
interface Tunnel<id>
 ip pim sparse-mode
end
```

```text
# Spoke_PIM_SM_Tunnel_Config
conf t
interface Tunnel<id>
 ip pim sparse-mode
end
```

```text
# Source_Interface_PIM_Config
conf t
interface <source-facing-interface>
 ip pim sparse-mode
end
```

```text
# Receiver_Interface_PIM_Config
conf t
interface <receiver-facing-interface>
 ip pim sparse-mode
end
```

```text
# RP_Loopback_Config
conf t
interface Loopback<id>
 ip address <rp-ip> <mask>
 ip pim sparse-mode
end
```

```text
# Static_RP_Config
conf t
ip pim rp-address <rp-ip>
end
```

```text
# Static_RP_Group_Filter_Option
conf t
access-list <acl-number> permit <multicast-group> <wildcard>
ip pim rp-address <rp-ip> <acl-number>
end
```

```text
# Test_IGMP_Join_Config
conf t
interface <receiver-facing-interface>
 ip igmp join-group <multicast-group>
end
```

```text
# Remove_Test_IGMP_Join
conf t
interface <receiver-facing-interface>
 no ip igmp join-group <multicast-group>
end
```

```text
# Multicast_Test_Traffic
ping <multicast-group> source <source-interface-or-source-ip>
```

```text
# PIM_SM_DMVPN_Reminder
! DMVPN must already work before PIM is added.
! PIM sparse mode requires RP reachability.
! RPF follows the unicast routing table.
! NHRP multicast mapping is required for PIM hellos and joins across DMVPN.
```

# DMVPN_NHRP_Multicast_PIM_Sparse_Mode_Verification_Commands

| Check | Device | Command | Good Output |
|---|---|---|---|
| Underlay reachability | Hub, Spokes | `ping <remote-nbma-ip>` | NBMA pings succeed |
| DMVPN state | Hub, Spokes | `show dmvpn` | Peers show up |
| NHRP mappings | Hub, Spokes | `show ip nhrp` | Expected tunnel-to-NBMA mappings exist |
| Hub multicast mapping | Hub | `show running-config interface Tunnel<id>` | Hub has `ip nhrp map multicast dynamic` |
| Spoke multicast mapping | Spokes | `show running-config interface Tunnel<id>` | Spokes map multicast toward hub NBMA |
| Unicast route to RP | Hub, Spokes | `show ip route <rp-ip>` | RP is reachable |
| Unicast route to source | Hub, Spokes | `show ip route <source-ip>` | Source is reachable |
| Multicast routing enabled | Hub, Spokes | `show running-config \| include ip multicast-routing` | `ip multicast-routing` is present |
| PIM interfaces | Hub, Spokes | `show ip pim interface` | Tunnel and required LAN interfaces appear |
| PIM tunnel detail | Hub, Spokes | `show ip pim interface Tunnel<id>` | Tunnel is PIM sparse-mode enabled |
| PIM neighbors | Hub, Spokes | `show ip pim neighbor` | Expected tunnel PIM neighbors appear |
| RP mapping | Hub, Spokes | `show ip pim rp mapping` | Correct RP is mapped to expected groups |
| IGMP receiver state | Receiver-side Router | `show ip igmp groups` | Receiver-facing interface joined the group |
| Multicast route state | Hub, Spokes | `show ip mroute <group>` | `(*,G)` or `(S,G)` state appears |
| RPF to RP | Hub, Spokes | `show ip rpf <rp-ip>` | RPF interface points toward RP |
| RPF to source | Hub, Spokes | `show ip rpf <source-ip>` | RPF interface points toward source |
| Outgoing interface list | Hub, Spokes | `show ip mroute <group>` | Expected tunnel or receiver-facing interface appears in OIL |
| Test multicast traffic | Source-side Router | `ping <group> source <source-ip>` | Multicast state and packet counters increment |
| Phase 3 unicast shortcut | Spokes | `show ip nhrp` and `show ip cef <source-or-receiver-ip>` | Shortcut appears if Phase 3 is part of the design |
| Final stability | Hub, Spokes | `show dmvpn` and `show ip nhrp` | DMVPN remains stable after multicast testing |

# DMVPN_NHRP_Multicast_PIM_Sparse_Mode_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Remove test IGMP join | Receiver-side Router | See `Rollback_Test_IGMP_Join` | Test receiver leaves multicast group |
| 2 | Remove PIM from receiver-facing interface | Receiver-side Router | See `Rollback_PIM_Receiver_Interface` | Receiver LAN stops participating in PIM |
| 3 | Remove PIM from source-facing interface | Source-side Router | See `Rollback_PIM_Source_Interface` | Source LAN stops participating in PIM |
| 4 | Remove PIM from tunnel | Hub, Spokes | See `Rollback_PIM_Tunnel` | PIM stops running over DMVPN |
| 5 | Remove static RP mapping | Hub, Spokes | See `Rollback_Static_RP` | Static RP is removed |
| 6 | Remove restricted RP ACL if used | Hub, Spokes | See `Rollback_RP_ACL` | RP group filter is removed |
| 7 | Remove PIM from RP loopback if used only for this lab | RP Router | See `Rollback_RP_Loopback_PIM` | RP loopback no longer participates in PIM |
| 8 | Disable multicast routing if fully reverting multicast | Hub, Spokes | `no ip multicast-routing` | Multicast routing is disabled |
| 9 | Keep NHRP multicast if routing protocols still need it | Hub, Spokes | Design check | EIGRP, OSPF, RIP, or other multicast-based protocols are not broken |
| 10 | Remove hub NHRP multicast only if no multicast overlay use remains | Hub | See `Rollback_Hub_NHRP_Multicast` | Hub multicast mapping is removed |
| 11 | Remove spoke NHRP multicast only if no multicast overlay use remains | Spokes | See `Rollback_Spoke_NHRP_Multicast` | Spoke multicast mapping is removed |
| 12 | Clear multicast forwarding state | Hub, Spokes | `clear ip mroute *` | Multicast route state is flushed |
| 13 | Verify multicast state is removed | Hub, Spokes | `show ip mroute` and `show ip pim neighbor` | PIM neighbors and multicast routes are gone |
| 14 | Verify DMVPN remains stable | Hub, Spokes | `show dmvpn` and `show ip nhrp` | DMVPN core remains operational |

```text
# Rollback_Test_IGMP_Join
conf t
interface <receiver-facing-interface>
 no ip igmp join-group <multicast-group>
end
```

```text
# Rollback_PIM_Receiver_Interface
conf t
interface <receiver-facing-interface>
 no ip pim sparse-mode
end
```

```text
# Rollback_PIM_Source_Interface
conf t
interface <source-facing-interface>
 no ip pim sparse-mode
end
```

```text
# Rollback_PIM_Tunnel
conf t
interface Tunnel<id>
 no ip pim sparse-mode
end
```

```text
# Rollback_Static_RP
conf t
no ip pim rp-address <rp-ip>
end
```

```text
# Rollback_RP_ACL
conf t
no access-list <acl-number>
end
```

```text
# Rollback_RP_Loopback_PIM
conf t
interface Loopback<id>
 no ip pim sparse-mode
end
```

```text
# Rollback_Hub_NHRP_Multicast
conf t
interface Tunnel<id>
 no ip nhrp map multicast dynamic
end
```

```text
# Rollback_Spoke_NHRP_Multicast
conf t
interface Tunnel<id>
 no ip nhrp map multicast <hub-nbma-ip>
end
```

# DMVPN_NHRP_Multicast_PIM_Sparse_Mode_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| No PIM neighbors over tunnel | PIM not enabled on tunnel, NHRP multicast missing, or DMVPN broken | `show ip pim neighbor` | Enable PIM on tunnel, fix NHRP multicast, or repair DMVPN |
| PIM enabled but no RP mapping | Static RP missing or wrong | `show ip pim rp mapping` | Configure correct `ip pim rp-address <rp-ip>` |
| RP mapping exists but multicast fails | RP unreachable by unicast | `show ip route <rp-ip>` | Fix unicast routing to RP |
| IGMP group does not appear | Receiver did not join or join is on wrong interface | `show ip igmp groups` | Add receiver join on correct receiver-facing interface |
| Multicast route has no OIL | No receiver join or PIM join did not propagate | `show ip mroute <group>` | Fix IGMP receiver state and PIM neighbor path |
| Multicast route has wrong incoming interface | RPF failure | `show ip rpf <source-ip>` | Fix unicast route so RPF points to correct interface |
| Source traffic never reaches RP | Source-facing interface lacks PIM or route to RP is broken | `show ip pim interface` | Enable PIM on source interface and fix RP reachability |
| Receiver never gets traffic | Receiver-facing interface lacks PIM or IGMP state | `show ip pim interface` and `show ip igmp groups` | Enable PIM and confirm receiver joins group |
| Works on hub but not spoke | Spoke multicast mapping to hub missing | `show running-config interface Tunnel<id>` | Add spoke multicast mapping to hub NBMA |
| Works on one spoke but not another | PIM, RP, RPF, or IGMP differs on that spoke | `show ip pim neighbor` and `show ip mroute` | Match multicast configuration and routing |
| Phase 3 unicast works but multicast fails | NHRP shortcut does not replace PIM or RP requirements | `show ip pim neighbor` and `show ip mroute` | Troubleshoot multicast control plane separately |
| Multicast state exists but counters do not move | Source is not sending, wrong source address, or RPF issue | `show ip mroute <group>` | Generate traffic from correct source and fix RPF |
| PIM neighbors flap | DMVPN instability, multicast mapping issue, or tunnel loss | `show dmvpn` and `show ip pim neighbor` | Fix DMVPN and NHRP multicast first |
| Large multicast packets fail | GRE or IPsec overhead issue | `ping <destination> size <size> df-bit` | Configure tunnel MTU and TCP MSS |
| Engineer blames PIM too early | DMVPN core is unstable | `show dmvpn` and `show ip nhrp` | Fix DMVPN first, then PIM sparse mode |

##### Source_Basis
# DMVPN_NHRP_Multicast_PIM_Sparse_Mode_Mental_Model
# DMVPN_NHRP_Multicast_PIM_Sparse_Mode_Configuration_Checklist
# DMVPN_NHRP_Multicast_PIM_Sparse_Mode_Skeleton
# DMVPN_NHRP_Multicast_PIM_Sparse_Mode_Verification_Commands
# DMVPN_NHRP_Multicast_PIM_Sparse_Mode_Rollback
# DMVPN_NHRP_Multicast_PIM_Sparse_Mode_Failure_Checks

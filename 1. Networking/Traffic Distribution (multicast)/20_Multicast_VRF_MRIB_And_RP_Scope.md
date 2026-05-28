
Multicast_VRF_MRIB_And_RP_Scope.md

Multicast_VRF_MRIB_And_RP_Scope

# Multicast_VRF_MRIB_And_RP_Scope_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Multicast VRF | Multicast forwarding state is isolated per VRF, just like unicast routing state |
| Per-VRF multicast control plane | PIM neighbors, IGMP receiver state, RP mapping, RPF checks, and mroute entries belong to the VRF where the interfaces live |
| Default VRF separation | Multicast state in the default VRF does not automatically apply to a tenant VRF |
| MRIB | Multicast Routing Information Base used for multicast RPF decisions |
| uRIB | Unicast routing table. If no multicast-specific route exists in the MRIB, multicast RPF may fall back to the unicast RIB |
| Per-VRF RPF | RPF is evaluated inside the same VRF as the multicast flow |
| RP scope | A PIM RP address must be reachable and configured inside the same VRF as the multicast group it serves |
| IGMP scope | Receiver membership learned on a VRF interface creates group state only in that VRF |
| PIM neighbor scope | PIM neighbors form only across interfaces in the same routing context |
| Source scope | A source in VRF BLUE is not the same forwarding context as the same IP address in VRF RED |
| Route leaking warning | Leaking unicast routes between VRFs does not automatically merge multicast state, PIM neighbors, IGMP membership, or RP mapping |
| Static mroute in VRF | `ip mroute vrf <VRF>` can be used to override multicast RPF for a source inside that VRF |
| Mroute truth | `show ip mroute vrf <VRF>` is the forwarding truth table for multicast inside that VRF |
| Failure pattern | Most VRF multicast failures are wrong VRF, missing per-VRF multicast routing, missing RP in VRF, missing PIM on VRF interface, or bad per-VRF RPF |
# Multicast_VRF_MRIB_And_RP_Scope_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify multicast VRF, source interface, receiver interface, RP address, transit interfaces, and multicast group | All multicast routers | `show vrf`<br>`show ip interface brief` | Required VRF and interfaces are known |
| 2 | Confirm the source-facing interface belongs to the correct VRF | Source-side router | `show running-config interface <SOURCE_FACING_INTERFACE>` | Interface contains `vrf forwarding <VRF_NAME>` or `ip vrf forwarding <VRF_NAME>` |
| 3 | Confirm the receiver-facing interface belongs to the correct VRF | Last-hop router | `show running-config interface <RECEIVER_FACING_INTERFACE>` | Interface belongs to `<VRF_NAME>` |
| 4 | Confirm transit interfaces belong to the correct VRF | Transit routers | `show running-config interface <TRANSIT_INTERFACE>` | Transit path uses the same VRF where multicast should run |
| 5 | Confirm per-VRF unicast reachability to the source | Last-hop, RP, and transit routers | `show ip route vrf <VRF_NAME> <SOURCE_IP>` | Route to source exists inside the VRF |
| 6 | Confirm per-VRF unicast reachability to the RP | All multicast routers in the VRF | `show ip route vrf <VRF_NAME> <RP_ADDRESS>` | Route to RP exists inside the VRF |
| 7 | Confirm per-VRF RPF path to the source | Last-hop and transit routers | `show ip rpf vrf <VRF_NAME> <SOURCE_IP>` | RPF interface points toward the source inside the VRF |
| 8 | Confirm per-VRF RPF path to the RP | Last-hop and transit routers | `show ip rpf vrf <VRF_NAME> <RP_ADDRESS>` | RPF interface points toward the RP inside the VRF |
| 9 | Enable multicast routing for the VRF | All multicast routers in the VRF | `conf t`<br>`ip multicast-routing vrf <VRF_NAME>` | Router can build multicast state for the VRF |
| 10 | Enable PIM sparse mode on the source-facing VRF interface | Source-side router | `conf t`<br>`interface <SOURCE_FACING_INTERFACE>`<br>`ip pim sparse-mode` | Source-facing interface participates in PIM inside the VRF |
| 11 | Enable PIM sparse mode on transit VRF interfaces | Transit routers | `conf t`<br>`interface <TRANSIT_INTERFACE>`<br>`ip pim sparse-mode` | PIM can form across the VRF transit path |
| 12 | Enable PIM sparse mode on receiver-facing VRF interface | Last-hop router | `conf t`<br>`interface <RECEIVER_FACING_INTERFACE>`<br>`ip pim sparse-mode` | Receiver-facing interface can process IGMP and PIM state inside the VRF |
| 13 | Enable PIM sparse mode on RP loopback if RP is local to the router | RP router | `conf t`<br>`interface <RP_LOOPBACK_INTERFACE>`<br>`ip pim sparse-mode` | RP interface participates in multicast control plane |
| 14 | Configure static RP mapping inside the VRF | All multicast routers in the VRF | `conf t`<br>`ip pim vrf <VRF_NAME> rp-address <RP_ADDRESS> [<GROUP_ACL>]` | ASM group maps to the RP inside the VRF |
| 15 | Verify per-VRF RP mapping | All multicast routers in the VRF | `show ip pim vrf <VRF_NAME> rp mapping` | Multicast group maps to `<RP_ADDRESS>` inside `<VRF_NAME>` |
| 16 | Verify per-VRF PIM interface state | All multicast routers in the VRF | `show ip pim vrf <VRF_NAME> interface` | Required VRF interfaces appear as PIM interfaces |
| 17 | Verify per-VRF PIM neighbors | Transit routers | `show ip pim vrf <VRF_NAME> neighbor` | Expected PIM neighbors appear inside the VRF |
| 18 | Configure receiver membership for the VRF multicast group | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <MULTICAST_GROUP>` | Receiver joins the multicast group on the VRF-connected segment |
| 19 | Verify per-VRF IGMP membership | Last-hop router | `show ip igmp vrf <VRF_NAME> groups` | Joined group appears on the receiver-facing VRF interface |
| 20 | Verify shared-tree state inside the VRF | Last-hop and RP routers | `show ip mroute vrf <VRF_NAME> <MULTICAST_GROUP>` | `(*,<GROUP>)` state appears inside the VRF |
| 21 | Generate multicast traffic from the source inside the same VRF | Source/test router | `ping vrf <VRF_NAME> <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Source sends multicast traffic in the correct VRF |
| 22 | Verify source-specific state inside the VRF | RP, transit, and last-hop routers | `show ip mroute vrf <VRF_NAME> <MULTICAST_GROUP>` | `(S,G)` state appears inside the VRF after source traffic starts |
| 23 | Verify MRIB/RPF behavior for the source | Last-hop and transit routers | `show ip rpf vrf <VRF_NAME> <SOURCE_IP>`<br>`show ip mroute vrf <VRF_NAME> <MULTICAST_GROUP>` | mroute Incoming Interface matches per-VRF RPF path |
| 24 | Verify OIL inside the VRF | RP, transit, and last-hop routers | `show ip mroute vrf <VRF_NAME> <MULTICAST_GROUP>` | OIL points toward downstream receiver interfaces inside the VRF |
| 25 | Verify default VRF is not accidentally carrying the tenant multicast state | All multicast routers | `show ip mroute <MULTICAST_GROUP>` | Default VRF has no unintended state for the tenant group |
| 26 | Verify packet counters increment inside the VRF | All forwarding routers | `show ip mroute vrf <VRF_NAME> <MULTICAST_GROUP>` | Packet counters increment during source traffic |
| 27 | Configure static mroute inside the VRF only if per-VRF RPF must be overridden | Affected router | `conf t`<br>`ip mroute vrf <VRF_NAME> <SOURCE_IP> <SOURCE_MASK> <RPF_NEXT_HOP_OR_INTERFACE>` | MRIB override changes RPF path for the source inside the VRF |
| 28 | Verify static mroute changed per-VRF RPF | Affected router | `show ip rpf vrf <VRF_NAME> <SOURCE_IP>` | RPF path reflects the intended MRIB override |
| 29 | Clear stale multicast state for a clean retest | All multicast routers | `clear ip mroute vrf <VRF_NAME> *` | Old VRF multicast state is removed |
| 30 | Retest receiver join and source traffic | Receiver and source routers | `show ip igmp vrf <VRF_NAME> groups`<br>`ping vrf <VRF_NAME> <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | VRF multicast state rebuilds and forwards correctly |
| 31 | Save the working VRF multicast configuration | All changed routers | `copy running-config startup-config` | Multicast VRF, MRIB, RP, and PIM baseline is preserved |
# Multicast_VRF_MRIB_And_RP_Scope_Skeleton
! =========================================================
! VARIABLES
! =========================================================
! <VRF_NAME>                    = tenant VRF name
! <RD_VALUE>                    = route distinguisher if required
! <SOURCE_IP>                   = multicast source IP inside the VRF
! <SOURCE_MASK>                 = source prefix mask for static mroute
! <SOURCE_INTERFACE_OR_IP>      = source interface or IP for multicast test
! <MULTICAST_GROUP>             = ASM multicast group
! <RP_ADDRESS>                  = RP address reachable inside the VRF
! <GROUP_ACL>                   = optional ACL matching multicast groups for RP
! <RP_LOOPBACK_INTERFACE>       = RP loopback interface
! <SOURCE_FACING_INTERFACE>     = interface toward multicast source
! <RECEIVER_FACING_INTERFACE>   = interface toward multicast receiver
! <TRANSIT_INTERFACE>           = routed transit interface in the VRF
! <RECEIVER_INTERFACE>          = test receiver interface
! <RPF_NEXT_HOP_OR_INTERFACE>   = next hop or interface for static mroute override
! =========================================================
! VRF BASELINE
! Use syntax supported by your IOS/IOS XE image.
! =========================================================
conf t
 ip vrf <VRF_NAME>
  rd <RD_VALUE>
 exit
end
! =========================================================
! ASSIGN INTERFACES TO VRF
! Warning: applying vrf forwarding removes the interface IP address.
! Reapply the IP address after assigning the VRF.
! =========================================================
conf t
 interface <SOURCE_FACING_INTERFACE>
  ip vrf forwarding <VRF_NAME>
  ip address <SOURCE_SIDE_INTERFACE_IP> <MASK>
  ip pim sparse-mode
 exit
 interface <RECEIVER_FACING_INTERFACE>
  ip vrf forwarding <VRF_NAME>
  ip address <RECEIVER_SIDE_INTERFACE_IP> <MASK>
  ip pim sparse-mode
 exit
 interface <TRANSIT_INTERFACE>
  ip vrf forwarding <VRF_NAME>
  ip address <TRANSIT_INTERFACE_IP> <MASK>
  ip pim sparse-mode
 exit
end
! =========================================================
! ENABLE MULTICAST ROUTING FOR THE VRF
! Apply on every router participating in multicast inside this VRF.
! =========================================================
conf t
 ip multicast-routing vrf <VRF_NAME>
end
! =========================================================
! RP INTERFACE
! If RP loopback is inside the VRF, make sure it belongs to
! the VRF and is reachable through the VRF routing table.
! =========================================================
conf t
 interface <RP_LOOPBACK_INTERFACE>
  ip vrf forwarding <VRF_NAME>
  ip address <RP_ADDRESS> 255.255.255.255
  ip pim sparse-mode
 exit
end
! =========================================================
! STATIC RP MAPPING INSIDE VRF
! Apply consistently on every multicast router in the VRF.
! =========================================================
conf t
 ip pim vrf <VRF_NAME> rp-address <RP_ADDRESS> [<GROUP_ACL>]
end
! =========================================================
! OPTIONAL STATIC MROUTE INSIDE VRF
! Use only when multicast RPF must differ from unicast routing.
! =========================================================
conf t
 ip mroute vrf <VRF_NAME> <SOURCE_IP> <SOURCE_MASK> <RPF_NEXT_HOP_OR_INTERFACE>
end
! =========================================================
! RECEIVER JOIN
! Interface must be connected inside the same VRF multicast domain.
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! TEST TRAFFIC
! Use vrf keyword when the source/test router supports it.
! =========================================================
ping vrf <VRF_NAME> <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5
! =========================================================
! VERIFICATION
! =========================================================
show vrf
show ip route vrf <VRF_NAME> <SOURCE_IP>
show ip route vrf <VRF_NAME> <RP_ADDRESS>
show ip pim vrf <VRF_NAME> interface
show ip pim vrf <VRF_NAME> neighbor
show ip pim vrf <VRF_NAME> rp mapping
show ip igmp vrf <VRF_NAME> groups
show ip rpf vrf <VRF_NAME> <SOURCE_IP>
show ip rpf vrf <VRF_NAME> <RP_ADDRESS>
show ip mroute vrf <VRF_NAME> <MULTICAST_GROUP>
show ip mroute <MULTICAST_GROUP>
# Multicast_VRF_MRIB_And_RP_Scope_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| VRF exists | All routers | `show vrf` | `<VRF_NAME>` exists |
| Interface VRF assignment | All routers | `show running-config interface <INTERFACE>` | Interface is assigned to `<VRF_NAME>` |
| Per-VRF multicast routing | All multicast routers | `show running-config | include ip multicast-routing vrf` | `ip multicast-routing vrf <VRF_NAME>` is present |
| Per-VRF route to source | Last-hop, RP, and transit routers | `show ip route vrf <VRF_NAME> <SOURCE_IP>` | Route exists inside the VRF |
| Per-VRF route to RP | All multicast routers | `show ip route vrf <VRF_NAME> <RP_ADDRESS>` | Route exists inside the VRF |
| Per-VRF PIM interfaces | All multicast routers | `show ip pim vrf <VRF_NAME> interface` | Required VRF interfaces appear |
| Per-VRF PIM neighbors | Transit routers | `show ip pim vrf <VRF_NAME> neighbor` | Expected neighbors appear inside the VRF |
| Per-VRF RP mapping | All multicast routers | `show ip pim vrf <VRF_NAME> rp mapping` | Test group maps to `<RP_ADDRESS>` inside the VRF |
| Per-VRF IGMP membership | Last-hop router | `show ip igmp vrf <VRF_NAME> groups` | Receiver group appears on VRF receiver-facing interface |
| Per-VRF RPF to source | Last-hop and transit routers | `show ip rpf vrf <VRF_NAME> <SOURCE_IP>` | RPF path points toward source inside the VRF |
| Per-VRF RPF to RP | Last-hop and transit routers | `show ip rpf vrf <VRF_NAME> <RP_ADDRESS>` | RPF path points toward RP inside the VRF |
| VRF mroute state | All multicast routers | `show ip mroute vrf <VRF_NAME> <MULTICAST_GROUP>` | `(*,G)` and/or `(S,G)` state exists inside the VRF |
| Incoming interface | Transit and last-hop routers | `show ip mroute vrf <VRF_NAME> <MULTICAST_GROUP>` | Incoming Interface matches VRF RPF path |
| OIL state | RP, transit, and last-hop routers | `show ip mroute vrf <VRF_NAME> <MULTICAST_GROUP>` | OIL includes downstream VRF receiver path |
| Default VRF leak check | All routers | `show ip mroute <MULTICAST_GROUP>` | No unintended tenant multicast state exists in default VRF |
| Static mroute override | Affected router | `show running-config | include ^ip mroute vrf` | VRF static mroute exists only if deliberately configured |
| Static mroute RPF result | Affected router | `show ip rpf vrf <VRF_NAME> <SOURCE_IP>` | RPF path follows intended static mroute override |
| Packet counters | All forwarding routers | `show ip mroute vrf <VRF_NAME> <MULTICAST_GROUP>` | Packet counters increment during source traffic |
| End-to-end test | Source/test router | `ping vrf <VRF_NAME> <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Receiver replies or packet counters increment |
# Multicast_VRF_MRIB_And_RP_Scope_Rollback
! =========================================================
! REMOVE RECEIVER JOIN
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  no ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! REMOVE VRF STATIC MROUTE IF USED
! =========================================================
conf t
 no ip mroute vrf <VRF_NAME> <SOURCE_IP> <SOURCE_MASK> <RPF_NEXT_HOP_OR_INTERFACE>
end
! =========================================================
! REMOVE VRF RP MAPPING
! =========================================================
conf t
 no ip pim vrf <VRF_NAME> rp-address <RP_ADDRESS>
 no ip pim vrf <VRF_NAME> rp-address <RP_ADDRESS> <GROUP_ACL>
end
! =========================================================
! REMOVE PIM FROM VRF INTERFACES
! Only do this if no other multicast labs depend on them.
! =========================================================
conf t
 interface <SOURCE_FACING_INTERFACE>
  no ip pim sparse-mode
 exit
 interface <RECEIVER_FACING_INTERFACE>
  no ip pim sparse-mode
 exit
 interface <TRANSIT_INTERFACE>
  no ip pim sparse-mode
 exit
 interface <RP_LOOPBACK_INTERFACE>
  no ip pim sparse-mode
 exit
end
! =========================================================
! DISABLE MULTICAST ROUTING FOR THE VRF
! Only do this if no other multicast services depend on it.
! =========================================================
conf t
 no ip multicast-routing vrf <VRF_NAME>
end
! =========================================================
! OPTIONAL INTERFACE VRF REMOVAL
! Warning: removing VRF forwarding also removes interface IP address.
! Use only in a lab cleanup.
! =========================================================
conf t
 interface <SOURCE_FACING_INTERFACE>
  no ip vrf forwarding <VRF_NAME>
 exit
 interface <RECEIVER_FACING_INTERFACE>
  no ip vrf forwarding <VRF_NAME>
 exit
 interface <TRANSIT_INTERFACE>
  no ip vrf forwarding <VRF_NAME>
 exit
 interface <RP_LOOPBACK_INTERFACE>
  no ip vrf forwarding <VRF_NAME>
 exit
end
! =========================================================
! OPTIONAL VRF REMOVAL
! Use only if the VRF was created solely for this lab.
! =========================================================
conf t
 no ip vrf <VRF_NAME>
end
! =========================================================
! CLEAR MULTICAST STATE AND DEBUGGING
! Command support varies by platform.
! =========================================================
clear ip mroute vrf <VRF_NAME> *
clear ip igmp vrf <VRF_NAME> group *
undebug all
# Multicast_VRF_MRIB_And_RP_Scope_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| `show ip mroute vrf <VRF>` is empty | Multicast routing not enabled for the VRF or no source/receiver state exists | VRF multicast routers | `show running-config | include ip multicast-routing vrf`<br>`show ip igmp vrf <VRF_NAME> groups` | Configure `ip multicast-routing vrf <VRF_NAME>` and verify receiver/source state |
| PIM interface missing in VRF output | PIM not enabled on the VRF interface or interface is in wrong VRF | Affected router | `show ip pim vrf <VRF_NAME> interface`<br>`show running-config interface <INTERFACE>` | Assign interface to correct VRF and configure `ip pim sparse-mode` |
| PIM neighbor missing | Transit interfaces are in different VRFs, PIM missing, or L3 reachability is broken | Transit routers | `show ip pim vrf <VRF_NAME> neighbor`<br>`show ip interface brief vrf <VRF_NAME>` | Put both ends in same VRF and enable PIM |
| RP mapping missing inside VRF | Static RP configured in default VRF instead of tenant VRF | All VRF routers | `show ip pim vrf <VRF_NAME> rp mapping`<br>`show running-config | include rp-address` | Configure `ip pim vrf <VRF_NAME> rp-address <RP_ADDRESS>` |
| RP reachable in default VRF but not tenant VRF | RP route exists only in default routing table | All VRF routers | `show ip route <RP_ADDRESS>`<br>`show ip route vrf <VRF_NAME> <RP_ADDRESS>` | Add RP reachability inside the VRF |
| Source reachable in default VRF but not tenant VRF | Source route exists only in default routing table | Last-hop and RP routers | `show ip route <SOURCE_IP>`<br>`show ip route vrf <VRF_NAME> <SOURCE_IP>` | Add or leak the route into the correct VRF |
| Receiver group missing | Receiver join happened on wrong interface or wrong VRF | Last-hop router | `show ip igmp vrf <VRF_NAME> groups`<br>`show running-config interface <RECEIVER_INTERFACE>` | Configure receiver join on the correct VRF-connected interface |
| `(*,G)` exists but no `(S,G)` state | Source traffic not arriving, source-side PIM missing, or RP/source RPF is broken | RP and source-side routers | `show ip mroute vrf <VRF_NAME> <GROUP>`<br>`show ip rpf vrf <VRF_NAME> <SOURCE_IP>` | Generate source traffic in the VRF and fix PIM/RPF |
| Incoming interface is wrong | Per-VRF RPF path points to wrong interface | Transit or last-hop router | `show ip rpf vrf <VRF_NAME> <SOURCE_IP>`<br>`show ip mroute vrf <VRF_NAME> <GROUP>` | Correct VRF unicast route or configure deliberate `ip mroute vrf` override |
| OIL is empty | No downstream receiver interest in that VRF | RP or transit router | `show ip mroute vrf <VRF_NAME> <GROUP>`<br>`show ip igmp vrf <VRF_NAME> groups` | Restore receiver join and downstream PIM state |
| Default VRF shows the multicast state instead of tenant VRF | Interface or source test ran in default VRF | Source/test router and multicast routers | `show ip mroute <GROUP>`<br>`show ip mroute vrf <VRF_NAME> <GROUP>` | Move interface/test command into the correct VRF |
| Ping to multicast group fails | Source test did not use VRF keyword or source interface is wrong | Source/test router | `show ip interface brief vrf <VRF_NAME>` | Use `ping vrf <VRF_NAME> <GROUP> source <SOURCE_INTERFACE_OR_IP>` |
| Route leak exists but multicast still fails | Unicast route leak does not create PIM, IGMP, RP, or multicast forwarding state across VRFs | Border router | `show ip route vrf <VRF_NAME>`<br>`show ip pim vrf <VRF_NAME> rp mapping` | Build multicast separately in each VRF or use a supported multicast VPN/inter-VRF design |
| Static mroute override has no effect | Wrong VRF, source prefix, or next hop/interface configured | Affected router | `show running-config | include ^ip mroute vrf`<br>`show ip rpf vrf <VRF_NAME> <SOURCE_IP>` | Correct the `ip mroute vrf` source prefix and RPF next hop/interface |
| Applying VRF forwarding removed IP address | IOS behavior: VRF assignment clears interface IP | Affected router | `show running-config interface <INTERFACE>` | Reapply the interface IP address after assigning the VRF |
| Clear command fails | Platform uses different VRF clear syntax | Affected router | `clear ip mroute ?`<br>`clear ip igmp ?` | Use platform-supported clear syntax or wait for state aging |
| Debug floods console | Multicast debug left enabled | Any router | `show debugging` | Run `undebug all` |
##### Source_Basis
# Multicast_VRF_MRIB_And_RP_Scope_Mental_Model
# Multicast_VRF_MRIB_And_RP_Scope_Configuration_Checklist
# Multicast_VRF_MRIB_And_RP_Scope_Skeleton
# Multicast_VRF_MRIB_And_RP_Scope_Verification_Commands
# Multicast_VRF_MRIB_And_RP_Scope_Rollback
# Multicast_VRF_MRIB_And_RP_Scope_Failure_Checks

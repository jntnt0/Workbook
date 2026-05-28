
MSDP_Source_Active_Exchange_And_Anycast_RP.md

MSDP_Source_Active_Exchange_And_Anycast_RP

# MSDP_Source_Active_Exchange_And_Anycast_RP_Mental_Model
| Concept | Operational Meaning |
|---|---|
| MSDP | Multicast Source Discovery Protocol advertises active multicast source information between RPs |
| Source Active message | SA message carries source IP, multicast group, and originating RP address |
| MSDP transport | MSDP sessions use TCP port 639 |
| RP-only origination | The RP for a local `(S,G)` is the router that originates SA messages for that source/group |
| Anycast RP | Multiple RP routers share the same RP address so routers can use the nearest RP |
| Anycast RP problem | Source may register to one RP while receiver joins toward another RP |
| MSDP solution | MSDP peers between RPs so each RP learns active sources known by the other RPs |
| Same RP address | All Anycast RP routers use the same RP address for PIM RP mapping |
| Unique MSDP address | MSDP peering must use unique router loopbacks, not the shared Anycast RP address |
| Source-side RP | The nearest RP to the source receives PIM register state and originates SA messages |
| Receiver-side RP | The nearest RP to the receiver learns remote source state from MSDP and can build forwarding toward the source |
| Peer-RPF | MSDP loop prevention checks whether an SA arrived from the logically correct MSDP peer |
| Mesh group | MSDP mesh group reduces SA reflooding and avoids peer-RPF checks inside the mesh group |
| BGP relationship | For non-mesh or interdomain MSDP, BGP or mBGP topology should mirror MSDP topology so peer-RPF works |
| ASM only | MSDP helps Any Source Multicast with RPs. SSM does not use RPs or MSDP |
| Verification truth | `show ip msdp peer`, `show ip msdp sa-cache`, `show ip pim rp mapping`, and `show ip mroute` prove the design |
# MSDP_Source_Active_Exchange_And_Anycast_RP_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify Anycast RP routers, shared RP address, unique MSDP loopbacks, source segment, receiver segment, and multicast group | All multicast routers | `show ip interface brief` | RP loopbacks, MSDP peering loopbacks, source-facing interfaces, and receiver-facing interfaces are known |
| 2 | Confirm the shared Anycast RP address plan | RP routers | `show running-config interface <ANYCAST_RP_LOOPBACK>` | Each RP will use the same `<ANYCAST_RP_ADDRESS>` on the Anycast RP loopback |
| 3 | Confirm unique MSDP peering address plan | RP routers | `show running-config interface <MSDP_SOURCE_LOOPBACK>` | Each RP has a unique loopback for MSDP and BGP peering |
| 4 | Configure the shared Anycast RP loopback on each RP | RP routers | `conf t`<br>`interface <ANYCAST_RP_LOOPBACK>`<br>`ip address <ANYCAST_RP_ADDRESS> 255.255.255.255` | Every RP owns the same Anycast RP address |
| 5 | Configure a unique MSDP peering loopback on each RP | RP routers | `conf t`<br>`interface <MSDP_SOURCE_LOOPBACK>`<br>`ip address <LOCAL_MSDP_LOOPBACK_IP> 255.255.255.255` | Each RP has a unique MSDP source address |
| 6 | Advertise the Anycast RP loopback into unicast routing | RP routers | `router <IGP_PROCESS>`<br>`network <ANYCAST_RP_ADDRESS> 0.0.0.0 area <AREA>` | Routers can route to the nearest Anycast RP address |
| 7 | Advertise each unique MSDP loopback into unicast routing | RP routers | `router <IGP_PROCESS>`<br>`network <LOCAL_MSDP_LOOPBACK_IP> 0.0.0.0 area <AREA>` | RPs can reach each other's unique MSDP peering loopbacks |
| 8 | Verify reachability to remote MSDP loopbacks | RP routers | `ping <REMOTE_MSDP_LOOPBACK_IP> source <LOCAL_MSDP_LOOPBACK_IP>` | Remote MSDP loopback is reachable from the local MSDP source address |
| 9 | Verify route to the shared Anycast RP address from non-RP routers | Non-RP multicast routers | `show ip route <ANYCAST_RP_ADDRESS>` | Route points toward the nearest RP |
| 10 | Enable multicast routing globally | All multicast routers | `conf t`<br>`ip multicast-routing` | Routers can build multicast control-plane and forwarding state |
| 11 | Enable PIM sparse mode on source-facing interfaces | First-hop/source-side routers | `conf t`<br>`interface <SOURCE_FACING_INTERFACE>`<br>`ip pim sparse-mode` | Source-side interface participates in PIM-SM |
| 12 | Enable PIM sparse mode on receiver-facing interfaces | Last-hop/receiver-side routers | `conf t`<br>`interface <RECEIVER_FACING_INTERFACE>`<br>`ip pim sparse-mode` | Receiver-facing interface can process IGMP joins |
| 13 | Enable PIM sparse mode on multicast transit interfaces | All multicast routers | `conf t`<br>`interface <TRANSIT_INTERFACE>`<br>`ip pim sparse-mode` | PIM neighbors can form across transit links |
| 14 | Configure all multicast routers to use the shared Anycast RP address | All multicast routers | `conf t`<br>`ip pim rp-address <ANYCAST_RP_ADDRESS> [<GROUP_ACL>]` | All routers map the ASM group to the same RP identity |
| 15 | Verify RP mapping | All multicast routers | `show ip pim rp mapping` | Multicast group maps to `<ANYCAST_RP_ADDRESS>` |
| 16 | Verify each router routes to its nearest Anycast RP | All multicast routers | `show ip route <ANYCAST_RP_ADDRESS>` | Different parts of the network may route to different physical RPs |
| 17 | Configure MSDP peer from RP1 to RP2 using unique loopbacks | RP1 | `conf t`<br>`ip msdp peer <RP2_MSDP_LOOPBACK_IP> connect-source <MSDP_SOURCE_LOOPBACK> remote-as <REMOTE_AS>` | RP1 initiates MSDP peering to RP2 using unique source loopback |
| 18 | Configure MSDP peer from RP2 to RP1 using unique loopbacks | RP2 | `conf t`<br>`ip msdp peer <RP1_MSDP_LOOPBACK_IP> connect-source <MSDP_SOURCE_LOOPBACK> remote-as <REMOTE_AS>` | RP2 initiates MSDP peering to RP1 using unique source loopback |
| 19 | Configure additional MSDP peers for every RP pair in the Anycast RP set | RP routers | `ip msdp peer <REMOTE_RP_MSDP_LOOPBACK_IP> connect-source <MSDP_SOURCE_LOOPBACK> remote-as <REMOTE_AS>` | All Anycast RPs have MSDP sessions to the other RPs |
| 20 | Add MSDP mesh-group entries for intra-domain Anycast RP mesh | RP routers | `conf t`<br>`ip msdp mesh-group <MSDP_MESH_GROUP_NAME> <REMOTE_RP_MSDP_LOOPBACK_IP>` | SA flooding is optimized inside the RP mesh |
| 21 | Verify MSDP peer sessions | RP routers | `show ip msdp peer` | MSDP peers are established |
| 22 | Verify TCP 639 connectivity if session is not established | RP routers | `show tcp brief | include 639` | MSDP TCP session exists or connection attempt is visible |
| 23 | Configure receiver membership for the ASM group | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <MULTICAST_GROUP>` | Receiver joins the Anycast RP backed ASM group |
| 24 | Verify receiver membership on the receiver-side last-hop router | Last-hop router | `show ip igmp groups` | Joined group appears on receiver-facing interface |
| 25 | Verify receiver-side shared-tree state toward nearest RP | Receiver-side routers | `show ip mroute <MULTICAST_GROUP>` | `(*,<GROUP>)` state exists toward the local nearest Anycast RP |
| 26 | Generate multicast traffic from the source near a different RP | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Source sends ASM multicast traffic |
| 27 | Verify source-side RP receives local source state | Source-side RP | `show ip mroute <MULTICAST_GROUP>` | `(S,G)` appears for the active local source |
| 28 | Verify source-side RP creates MSDP SA cache entry | Source-side RP | `show ip msdp sa-cache` | SA entry exists for `<SOURCE_IP>,<MULTICAST_GROUP>` |
| 29 | Verify receiver-side RP learns remote source from MSDP | Receiver-side RP | `show ip msdp sa-cache` | SA entry learned from MSDP peer appears for the remote source/group |
| 30 | Verify MSDP-created multicast state on receiver-side RP | Receiver-side RP | `show ip mroute <MULTICAST_GROUP>` | `(S,G)` appears and may show MSDP-created state flag |
| 31 | Verify receiver-side RP can build toward the source | Receiver-side RP | `show ip rpf <SOURCE_IP>` | RPF path points toward the multicast source |
| 32 | Verify traffic reaches receiver through Anycast RP and MSDP learning | Source/test router and multicast routers | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5`<br>`show ip mroute <MULTICAST_GROUP>` | Receiver replies or packet counters increment along the forwarding path |
| 33 | Verify SA exchange in both directions if both sides have sources | Both RP routers | `show ip msdp sa-cache` | Each RP learns local and remote active sources as expected |
| 34 | Use controlled MSDP debugging only if SA exchange is unclear | RP routers | `debug ip msdp` | MSDP peer, SA, and peer-RPF activity is visible |
| 35 | Disable debugging after test | RP routers | `undebug all` | Debugging is disabled |
| 36 | Save the working Anycast RP and MSDP configuration | All changed routers | `copy running-config startup-config` | Anycast RP and MSDP baseline is preserved |
# MSDP_Source_Active_Exchange_And_Anycast_RP_Skeleton
! =========================================================
! VARIABLES
! =========================================================
! <ANYCAST_RP_ADDRESS>           = shared RP address used on all Anycast RP routers
! <ANYCAST_RP_LOOPBACK>          = loopback carrying shared Anycast RP address
! <MSDP_SOURCE_LOOPBACK>         = unique loopback used for MSDP peering
! <LOCAL_MSDP_LOOPBACK_IP>       = local unique MSDP source IP
! <REMOTE_RP_MSDP_LOOPBACK_IP>   = remote RP unique MSDP peering IP
! <RP1_MSDP_LOOPBACK_IP>         = RP1 unique MSDP peering IP
! <RP2_MSDP_LOOPBACK_IP>         = RP2 unique MSDP peering IP
! <REMOTE_AS>                    = remote AS for MSDP peering
! <IGP_PROCESS>                  = IGP process used for loopback reachability
! <AREA>                         = OSPF area if using OSPF
! <GROUP_ACL>                    = optional RP group ACL
! <MSDP_MESH_GROUP_NAME>         = MSDP mesh group name
! <SOURCE_FACING_INTERFACE>      = first-hop router interface toward source
! <RECEIVER_FACING_INTERFACE>    = last-hop router interface toward receiver
! <TRANSIT_INTERFACE>            = routed multicast transit interface
! <RECEIVER_INTERFACE>           = test receiver interface
! <MULTICAST_GROUP>              = ASM multicast group
! <SOURCE_IP>                    = multicast source IP
! <SOURCE_INTERFACE_OR_IP>       = source interface or IP used for multicast test traffic
! =========================================================
! RP ROUTER LOOPBACKS
! Configure same Anycast RP address on all RP routers.
! Configure unique MSDP source loopback on each RP router.
! =========================================================
conf t
 interface <ANYCAST_RP_LOOPBACK>
  ip address <ANYCAST_RP_ADDRESS> 255.255.255.255
 exit
 interface <MSDP_SOURCE_LOOPBACK>
  ip address <LOCAL_MSDP_LOOPBACK_IP> 255.255.255.255
 exit
end
! =========================================================
! LOOPBACK REACHABILITY
! Example shown with OSPF.
! Advertise both the shared Anycast RP address and unique MSDP loopback.
! =========================================================
conf t
 router ospf <IGP_PROCESS>
  network <ANYCAST_RP_ADDRESS> 0.0.0.0 area <AREA>
  network <LOCAL_MSDP_LOOPBACK_IP> 0.0.0.0 area <AREA>
 exit
end
! =========================================================
! GLOBAL MULTICAST ENABLEMENT
! Apply on all multicast routers
! =========================================================
conf t
 ip multicast-routing
end
! =========================================================
! PIM SPARSE MODE BASELINE
! Apply on routed source-facing, receiver-facing, and transit interfaces.
! =========================================================
conf t
 interface <SOURCE_FACING_INTERFACE>
  ip pim sparse-mode
 exit
 interface <RECEIVER_FACING_INTERFACE>
  ip pim sparse-mode
 exit
 interface <TRANSIT_INTERFACE>
  ip pim sparse-mode
 exit
end
! =========================================================
! ANYCAST RP MAPPING
! Apply consistently on all multicast routers.
! All routers use the same RP identity.
! =========================================================
conf t
 ip pim rp-address <ANYCAST_RP_ADDRESS> [<GROUP_ACL>]
end
! =========================================================
! MSDP PEERING BETWEEN ANYCAST RPS
! Use unique loopback addresses.
! Do not source MSDP from the shared Anycast RP address.
! =========================================================
conf t
 ip msdp peer <REMOTE_RP_MSDP_LOOPBACK_IP> connect-source <MSDP_SOURCE_LOOPBACK> remote-as <REMOTE_AS>
end
! =========================================================
! MSDP MESH GROUP
! Use for intra-domain Anycast RP full mesh.
! Add one line per remote RP peer.
! =========================================================
conf t
 ip msdp mesh-group <MSDP_MESH_GROUP_NAME> <REMOTE_RP_MSDP_LOOPBACK_IP>
end
! =========================================================
! OPTIONAL DEFAULT PEER
! Use only for a single upstream MSDP peer design.
! Not normally needed for Anycast RP full mesh.
! =========================================================
conf t
 ip msdp default-peer <REMOTE_RP_MSDP_LOOPBACK_IP>
end
! =========================================================
! RECEIVER JOIN
! Use join-group when a router is emulating a receiver.
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! TEST TRAFFIC
! =========================================================
ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5
! =========================================================
! VERIFICATION
! =========================================================
show ip pim rp mapping
show ip route <ANYCAST_RP_ADDRESS>
show ip route <REMOTE_RP_MSDP_LOOPBACK_IP>
show ip pim neighbor
show ip msdp peer
show ip msdp sa-cache
show ip igmp groups
show ip mroute <MULTICAST_GROUP>
show ip rpf <SOURCE_IP>
show tcp brief | include 639
! =========================================================
! CONTROLLED DEBUG
! Use briefly only during MSDP troubleshooting.
! =========================================================
debug ip msdp
undebug all
# MSDP_Source_Active_Exchange_And_Anycast_RP_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| Shared Anycast RP loopback | RP routers | `show ip interface brief | include <ANYCAST_RP_LOOPBACK>` | Each RP has the same `<ANYCAST_RP_ADDRESS>` configured |
| Unique MSDP loopback | RP routers | `show ip interface brief | include <MSDP_SOURCE_LOOPBACK>` | Each RP has a unique MSDP source address |
| Route to Anycast RP | All multicast routers | `show ip route <ANYCAST_RP_ADDRESS>` | Route points toward nearest Anycast RP |
| Route to remote MSDP loopback | RP routers | `show ip route <REMOTE_RP_MSDP_LOOPBACK_IP>` | Remote unique MSDP loopback is reachable |
| Loopback sourced ping | RP routers | `ping <REMOTE_RP_MSDP_LOOPBACK_IP> source <LOCAL_MSDP_LOOPBACK_IP>` | Remote RP MSDP loopback replies |
| Multicast routing enabled | All multicast routers | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| PIM interface state | All multicast routers | `show ip pim interface` | Source, receiver, and transit interfaces appear |
| PIM neighbor state | Transit routers | `show ip pim neighbor` | Expected PIM neighbors appear |
| RP mapping | All multicast routers | `show ip pim rp mapping` | ASM group maps to the shared Anycast RP address |
| MSDP peer config | RP routers | `show running-config | include ip msdp peer` | Peers use remote unique loopbacks and local connect-source loopback |
| MSDP mesh group config | RP routers | `show running-config | include ip msdp mesh-group` | Remote RP peers appear in mesh group |
| MSDP peer state | RP routers | `show ip msdp peer` | MSDP session is established |
| TCP session | RP routers | `show tcp brief | include 639` | MSDP TCP session exists |
| Local SA cache | Source-side RP | `show ip msdp sa-cache` | Local source/group appears after source traffic starts |
| Remote SA cache | Receiver-side RP | `show ip msdp sa-cache` | Remote source/group learned from MSDP appears |
| Receiver membership | Last-hop router | `show ip igmp groups` | Receiver group appears on receiver-facing interface |
| Source-side mroute state | Source-side RP | `show ip mroute <MULTICAST_GROUP>` | `(S,G)` appears for active source |
| Receiver-side mroute state | Receiver-side RP | `show ip mroute <MULTICAST_GROUP>` | MSDP-learned `(S,G)` state appears after SA learning and receiver interest |
| RPF toward source | Receiver-side RP and last-hop routers | `show ip rpf <SOURCE_IP>` | RPF path points toward multicast source |
| Packet counters | All forwarding routers | `show ip mroute <MULTICAST_GROUP>` | Packet counters increment during source traffic |
| End-to-end multicast test | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Receiver replies when `ip igmp join-group` is used and forwarding is correct |
| Debug cleanup | RP routers | `show debugging`<br>`undebug all` | No MSDP debug remains active |
# MSDP_Source_Active_Exchange_And_Anycast_RP_Rollback
! =========================================================
! REMOVE RECEIVER JOIN
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  no ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! REMOVE MSDP DEFAULT PEER IF USED
! =========================================================
conf t
 no ip msdp default-peer <REMOTE_RP_MSDP_LOOPBACK_IP>
end
! =========================================================
! REMOVE MSDP MESH GROUP ENTRY
! Repeat for each remote RP peer.
! =========================================================
conf t
 no ip msdp mesh-group <MSDP_MESH_GROUP_NAME> <REMOTE_RP_MSDP_LOOPBACK_IP>
end
! =========================================================
! REMOVE MSDP PEER
! =========================================================
conf t
 no ip msdp peer <REMOTE_RP_MSDP_LOOPBACK_IP>
end
! =========================================================
! REMOVE ANYCAST RP MAPPING
! =========================================================
conf t
 no ip pim rp-address <ANYCAST_RP_ADDRESS>
 no ip pim rp-address <ANYCAST_RP_ADDRESS> <GROUP_ACL>
end
! =========================================================
! REMOVE PIM FROM LAB INTERFACES
! Only do this if no other multicast labs depend on these interfaces.
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
end
! =========================================================
! REMOVE LOOPBACK ADDRESSES IF THEY WERE CREATED ONLY FOR THIS LAB
! Be careful with shared Anycast RP loopback removal.
! =========================================================
conf t
 interface <ANYCAST_RP_LOOPBACK>
  no ip address <ANYCAST_RP_ADDRESS> 255.255.255.255
 exit
 interface <MSDP_SOURCE_LOOPBACK>
  no ip address <LOCAL_MSDP_LOOPBACK_IP> 255.255.255.255
 exit
end
! =========================================================
! REMOVE GLOBAL MULTICAST ROUTING
! Only do this if no other multicast services depend on it.
! =========================================================
conf t
 no ip multicast-routing
end
! =========================================================
! CLEAR MULTICAST AND MSDP STATE
! Command support varies by platform.
! =========================================================
clear ip mroute *
clear ip igmp group *
clear ip msdp sa-cache
clear ip msdp peer *
undebug all
# MSDP_Source_Active_Exchange_And_Anycast_RP_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| MSDP peer does not establish | No route to remote unique loopback, wrong connect-source, remote-as mismatch, or TCP 639 blocked | RP routers | `show ip msdp peer`<br>`show ip route <REMOTE_RP_MSDP_LOOPBACK_IP>`<br>`show tcp brief | include 639` | Fix loopback reachability, connect-source, remote-as, or path filtering |
| MSDP sourced from shared Anycast RP address | Session instability or impossible peer uniqueness because multiple routers own the same address | RP routers | `show running-config | include ip msdp peer` | Source MSDP from unique loopback, not the Anycast RP loopback |
| RP mapping is inconsistent | Not all routers use the same Anycast RP address or group ACL differs | All multicast routers | `show ip pim rp mapping` | Configure consistent `ip pim rp-address <ANYCAST_RP_ADDRESS>` on all routers |
| Source registers to one RP but receiver joins another and traffic fails | MSDP SA exchange between Anycast RPs is broken | RP routers | `show ip msdp peer`<br>`show ip msdp sa-cache` | Restore MSDP peering and SA propagation |
| Source-side RP has `(S,G)` but remote RP has no SA | MSDP peer down, SA blocked, or peer-RPF failure | RP routers | `show ip msdp peer`<br>`show ip msdp sa-cache`<br>`debug ip msdp` | Fix MSDP session, mesh group, or BGP peer-RPF design |
| SA received but not accepted | Peer-RPF failure because MSDP topology does not match BGP topology | RP routers | `show ip msdp peer`<br>`debug ip msdp` | Use MSDP mesh group inside the Anycast RP domain or align MSDP and BGP topology |
| SA cache exists but traffic still fails | RPF toward source or PIM forwarding path is broken | Receiver-side RP and transit routers | `show ip rpf <SOURCE_IP>`<br>`show ip mroute <MULTICAST_GROUP>` | Fix unicast routing or multicast RPF path toward source |
| Receiver group missing | Receiver did not join or joined on wrong interface | Last-hop router and receiver | `show ip igmp groups`<br>`show running-config interface <RECEIVER_INTERFACE>` | Configure `ip igmp join-group <MULTICAST_GROUP>` on correct receiver interface |
| Source-side RP never creates local `(S,G)` | Source traffic not reaching first-hop router or PIM-SM/register path broken | Source-side RP and first-hop router | `show ip mroute <MULTICAST_GROUP>`<br>`show ip pim rp mapping` | Generate source traffic and verify PIM-SM source registration to Anycast RP |
| Receiver-side RP has SA but no receiver forwarding | Receiver shared tree or OIL is missing | Receiver-side RP and last-hop router | `show ip mroute <MULTICAST_GROUP>`<br>`show ip igmp groups` | Fix receiver join and downstream PIM/OIL state |
| Anycast RP route points to unexpected RP | IGP metric to shared RP address selects a different nearest RP | Non-RP routers | `show ip route <ANYCAST_RP_ADDRESS>` | Adjust IGP metrics if deterministic nearest-RP behavior is required |
| Duplicate unique MSDP loopback exists | Unique MSDP peering loopbacks were accidentally duplicated | RP routers | `show ip interface brief`<br>`show ip route <LOCAL_MSDP_LOOPBACK_IP>` | Assign unique MSDP loopback addresses |
| MSDP mesh group missing on one RP | SA flooding or peer-RPF behavior inconsistent inside RP mesh | RP routers | `show running-config | include mesh-group` | Configure matching `ip msdp mesh-group` entries for all RP peers |
| Using default peer in full-mesh Anycast RP design | Default peer is intended for simple single-peer designs, not a full Anycast RP mesh | RP routers | `show running-config | include default-peer` | Remove default peer unless the lab specifically tests single-peer behavior |
| SSM group expected to use MSDP | SSM does not use RP or MSDP | All multicast routers | `show running-config | include ip pim ssm`<br>`show ip pim rp mapping` | Use ASM group for MSDP/Anycast RP lab or use SSM-specific note |
| PIM dense mode expected to use MSDP | MSDP is for PIM-SM RP-based ASM, not dense-mode flood-and-prune | All multicast routers | `show ip pim interface` | Use PIM sparse mode and RP mapping |
| SA exists but packet counters stay at zero | No receiver interest, source stopped, or RPF/OIL broken | RP and last-hop routers | `show ip msdp sa-cache`<br>`show ip mroute <MULTICAST_GROUP>` | Restore receiver join, source traffic, and forwarding path |
| Debug floods the console | MSDP debug left enabled | RP routers | `show debugging` | Run `undebug all` |
##### Source_Basis
# MSDP_Source_Active_Exchange_And_Anycast_RP_Mental_Model
# MSDP_Source_Active_Exchange_And_Anycast_RP_Configuration_Checklist
# MSDP_Source_Active_Exchange_And_Anycast_RP_Skeleton
# MSDP_Source_Active_Exchange_And_Anycast_RP_Verification_Commands
# MSDP_Source_Active_Exchange_And_Anycast_RP_Rollback
# MSDP_Source_Active_Exchange_And_Anycast_RP_Failure_Checks


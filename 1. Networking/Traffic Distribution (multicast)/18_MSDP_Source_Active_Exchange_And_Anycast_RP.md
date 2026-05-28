
MSDP_Source_Active_Exchange_And_Anycast_RP.md

MSDP_Source_Active_Exchange_And_Anycast_RP

# MSDP_Source_Active_Exchange_And_Anycast_RP_Mental_Model
| Concept                    | Operational Meaning                                                                                                                    |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| MSDP                       | Multicast Source Discovery Protocol lets RPs exchange active `(S,G)` source information                                                |
| Source Active message      | MSDP SA messages advertise source IP, multicast group, and originating RP                                                              |
| TCP transport              | MSDP uses TCP port 639 between peers                                                                                                   |
| RP-only source origination | The RP that knows about a local active source originates the SA message                                                                |
| SA transit                 | Non-originating MSDP peers can forward SA messages but do not create them for sources they do not own                                  |
| Anycast RP                 | Multiple routers use the same RP address so receivers and sources choose the closest RP through unicast routing                        |
| Anycast RP problem         | A source may register to one RP while a receiver joins toward another RP                                                               |
| MSDP solution              | MSDP synchronizes SA state between the Anycast RPs so each RP learns about sources registered to the others                            |
| Anycast RP address         | Same loopback IP configured on all RP routers and used as the PIM RP address                                                           |
| MSDP peering address       | Unique loopback IP per RP used for MSDP and BGP peering. Do not peer MSDP using the shared Anycast RP address                          |
| Static RP mapping          | All multicast routers point to the same Anycast RP address with `ip pim rp-address <ANYCAST_RP_ADDRESS>`                               |
| IGP role                   | IGP advertises both the shared Anycast RP address and each RP router's unique peering loopback                                         |
| Closest RP selection       | Routers reach the shared RP address through the nearest RP based on unicast routing                                                    |
| MSDP mesh group            | A full-mesh MSDP core can use mesh groups to avoid unnecessary SA flooding between mesh peers                                          |
| Peer-RPF                   | MSDP loop prevention check that accepts SAs only from the peer logically closest to the originating RP                                 |
| BGP relationship           | MSDP peer-RPF works best when MSDP peering mirrors BGP or multicast BGP topology                                                       |
| Default peer               | `ip msdp default-peer` bypasses peer-RPF logic for simple single-peer MSDP designs                                                     |
| Verification truth         | `show ip msdp peer` proves TCP session state. `show ip msdp sa-cache` proves SA exchange. `show ip mroute` proves multicast forwarding |
# MSDP_Source_Active_Exchange_And_Anycast_RP_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify Anycast RP routers, unique MSDP loopbacks, shared Anycast RP address, source segment, receiver segment, and multicast group | All multicast routers | `show ip interface brief` | Shared RP loopbacks, unique peering loopbacks, source-facing interfaces, receiver-facing interfaces, and transit links are known |
| 2 | Confirm each RP has a unique MSDP peering loopback | RP routers | `show ip interface brief` | Each RP has a unique loopback for MSDP peering |
| 3 | Confirm each RP has the same Anycast RP loopback address | RP routers | `show ip interface brief` | Each RP has the same `/32` Anycast RP address configured |
| 4 | Confirm unicast reachability to the shared Anycast RP address | All multicast routers | `show ip route <ANYCAST_RP_ADDRESS>` | Route points to the nearest Anycast RP |
| 5 | Confirm unicast reachability to each unique MSDP peering loopback | RP routers | `show ip route <REMOTE_MSDP_PEER_ADDRESS>` | Each RP can reach every remote MSDP peer loopback |
| 6 | Confirm source reachability for multicast RPF | RP and last-hop routers | `show ip route <SOURCE_IP>`<br>`show ip rpf <SOURCE_IP>` | Routers have valid route and RPF path toward the multicast source |
| 7 | Enable multicast routing globally | All multicast routers | `conf t`<br>`ip multicast-routing` | Routers can build multicast control-plane and forwarding state |
| 8 | Configure the shared Anycast RP loopback address | RP routers | `conf t`<br>`interface <ANYCAST_RP_LOOPBACK>`<br>`ip address <ANYCAST_RP_ADDRESS> 255.255.255.255`<br>`ip pim sparse-mode` | Each RP owns the same RP address and participates in PIM |
| 9 | Configure the unique MSDP peering loopback | RP routers | `conf t`<br>`interface <UNIQUE_MSDP_LOOPBACK>`<br>`ip address <LOCAL_MSDP_PEER_ADDRESS> 255.255.255.255` | Each RP has a unique peering address |
| 10 | Advertise the shared Anycast RP address into the IGP | RP routers | `router <IGP_PROCESS>`<br>`network <ANYCAST_RP_ADDRESS> 0.0.0.0 area <AREA>` | All routers learn the shared RP address through the IGP |
| 11 | Advertise the unique MSDP peering loopback into the IGP | RP routers | `router <IGP_PROCESS>`<br>`network <LOCAL_MSDP_PEER_ADDRESS> 0.0.0.0 area <AREA>` | Other RPs can route to the unique MSDP peering address |
| 12 | Enable PIM sparse mode on RP transit interfaces | RP routers | `conf t`<br>`interface <TRANSIT_INTERFACE>`<br>`ip pim sparse-mode` | RP routers participate in PIM on multicast transit links |
| 13 | Enable PIM sparse mode on source-facing interface | First-hop/source-side router | `conf t`<br>`interface <SOURCE_FACING_INTERFACE>`<br>`ip pim sparse-mode` | Source-side router can register sources to the nearest RP |
| 14 | Enable PIM sparse mode on receiver-facing interface | Last-hop router | `conf t`<br>`interface <RECEIVER_FACING_INTERFACE>`<br>`ip pim sparse-mode` | Last-hop router can process IGMP joins and send PIM joins toward RP |
| 15 | Configure static RP mapping to the Anycast RP address | All multicast routers | `conf t`<br>`ip pim rp-address <ANYCAST_RP_ADDRESS> [<GROUP_ACL>]` | All routers use the shared Anycast RP address as the RP |
| 16 | Verify RP mapping | All multicast routers | `show ip pim rp mapping` | Test group maps to `<ANYCAST_RP_ADDRESS>` |
| 17 | Verify PIM interface state | All multicast routers | `show ip pim interface` | RP, transit, source-facing, and receiver-facing interfaces appear |
| 18 | Verify PIM neighbor state | Transit routers | `show ip pim neighbor` | Expected PIM neighbors appear |
| 19 | Configure MSDP peer from RP1 to RP2 using unique peering loopbacks | RP1 | `conf t`<br>`ip msdp peer <RP2_UNIQUE_MSDP_ADDRESS> connect-source <RP1_UNIQUE_MSDP_LOOPBACK> remote-as <REMOTE_AS>` | RP1 initiates MSDP TCP session to RP2 |
| 20 | Configure MSDP peer from RP2 to RP1 using unique peering loopbacks | RP2 | `conf t`<br>`ip msdp peer <RP1_UNIQUE_MSDP_ADDRESS> connect-source <RP2_UNIQUE_MSDP_LOOPBACK> remote-as <REMOTE_AS>` | RP2 initiates MSDP TCP session to RP1 |
| 21 | Configure additional MSDP peers for all Anycast RPs | RP routers | `ip msdp peer <REMOTE_MSDP_PEER_ADDRESS> connect-source <LOCAL_UNIQUE_MSDP_LOOPBACK> remote-as <REMOTE_AS>` | All Anycast RPs have MSDP peer statements for the full mesh or intended topology |
| 22 | Configure MSDP mesh group for full-mesh Anycast RP core | RP routers | `conf t`<br>`ip msdp mesh-group <MSDP_MESH_GROUP_NAME> <REMOTE_MSDP_PEER_ADDRESS>` | Mesh group reduces SA reflooding between full-mesh MSDP peers |
| 23 | Verify MSDP peer configuration | RP routers | `show running-config | include ^ip msdp` | MSDP peers and mesh group entries are present |
| 24 | Verify MSDP TCP session state | RP routers | `show ip msdp peer` | MSDP peer state is established |
| 25 | Verify MSDP summary if supported | RP routers | `show ip msdp summary` | Peers are up and SA counters are visible |
| 26 | Configure receiver membership for the test group | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <MULTICAST_GROUP>` | Receiver joins the multicast group |
| 27 | Verify receiver membership on last-hop router | Last-hop router | `show ip igmp groups` | Test group appears on receiver-facing interface |
| 28 | Generate multicast traffic from a source near RP1 | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Source registers to the nearest Anycast RP |
| 29 | Verify source registration on the nearest RP | Nearest RP | `show ip mroute <MULTICAST_GROUP>` | `(S,G)` state appears for the local source |
| 30 | Verify SA creation on source-side RP | Source-side RP | `show ip msdp sa-cache` | SA entry exists for `<SOURCE_IP>,<MULTICAST_GROUP>` |
| 31 | Verify SA exchange on remote RP | Remote RP | `show ip msdp sa-cache` | Remote RP learns SA entry from MSDP peer |
| 32 | Verify MSDP-created multicast state on remote RP | Remote RP | `show ip mroute <MULTICAST_GROUP>` | `(S,G)` state may show MSDP-created indication or source-aware state |
| 33 | Verify receiver-side RP can connect receivers to remote source | Receiver-side RP and last-hop router | `show ip mroute <MULTICAST_GROUP>` | OIL points toward receivers and RPF path points toward source |
| 34 | Verify end-to-end multicast forwarding | Source/test router and multicast routers | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5`<br>`show ip mroute <MULTICAST_GROUP>` | Receiver replies or packet counters increment |
| 35 | Verify peer-RPF inputs | RP routers | `show ip route <ORIGINATING_RP_ADDRESS>`<br>`show ip msdp peer` | MSDP peer topology and routing to originating RP are sane |
| 36 | Use controlled MSDP debugging only if SA exchange is unclear | RP routers | `debug ip msdp` | MSDP peer, SA, and peer-RPF events are visible |
| 37 | Disable debugging after the test | RP routers | `undebug all` | Debugging is disabled |
| 38 | Save the working Anycast RP and MSDP configuration | All changed routers | `copy running-config startup-config` | Anycast RP and MSDP baseline is preserved |
# MSDP_Source_Active_Exchange_And_Anycast_RP_Skeleton
! =========================================================
! VARIABLES
! =========================================================
! <ANYCAST_RP_ADDRESS>          = shared RP address configured on all Anycast RPs
! <ANYCAST_RP_LOOPBACK>         = loopback holding shared RP address
! <UNIQUE_MSDP_LOOPBACK>        = unique loopback used for MSDP peering
! <LOCAL_MSDP_PEER_ADDRESS>     = local unique MSDP loopback address
! <REMOTE_MSDP_PEER_ADDRESS>    = remote unique MSDP loopback address
! <RP1_UNIQUE_MSDP_ADDRESS>     = RP1 unique MSDP peering address
! <RP2_UNIQUE_MSDP_ADDRESS>     = RP2 unique MSDP peering address
! <RP1_UNIQUE_MSDP_LOOPBACK>    = RP1 unique MSDP loopback interface
! <RP2_UNIQUE_MSDP_LOOPBACK>    = RP2 unique MSDP loopback interface
! <REMOTE_AS>                   = remote AS for MSDP peer statement, same AS for iMSDP-style lab
! <MSDP_MESH_GROUP_NAME>        = MSDP mesh group name
! <IGP_PROCESS>                 = IGP process, example ospf 100
! <AREA>                        = OSPF area, example 0
! <TRANSIT_INTERFACE>           = routed PIM transit interface
! <SOURCE_FACING_INTERFACE>     = first-hop router interface toward multicast source
! <RECEIVER_FACING_INTERFACE>   = last-hop router interface toward receivers
! <RECEIVER_INTERFACE>          = test receiver interface
! <GROUP_ACL>                   = optional RP group ACL
! <MULTICAST_GROUP>             = test multicast group
! <SOURCE_IP>                   = multicast source IP
! <SOURCE_INTERFACE_OR_IP>      = source interface or source IP used for test traffic
! <ORIGINATING_RP_ADDRESS>      = RP address in received SA originator field
! =========================================================
! GLOBAL MULTICAST ENABLEMENT
! Apply on all multicast routers
! =========================================================
conf t
 ip multicast-routing
end
! =========================================================
! ANYCAST RP ROUTER BASELINE
! Apply on every Anycast RP router.
! The Anycast RP address is shared.
! The MSDP peering address is unique.
! =========================================================
conf t
 interface <ANYCAST_RP_LOOPBACK>
  ip address <ANYCAST_RP_ADDRESS> 255.255.255.255
  ip pim sparse-mode
 exit
 interface <UNIQUE_MSDP_LOOPBACK>
  ip address <LOCAL_MSDP_PEER_ADDRESS> 255.255.255.255
 exit
 interface <TRANSIT_INTERFACE>
  ip pim sparse-mode
 exit
end
! =========================================================
! IGP ADVERTISEMENT
! Advertise both the shared RP address and the unique MSDP address.
! Do not let the shared Anycast RP address become the router ID.
! =========================================================
conf t
 router <IGP_PROCESS>
  network <ANYCAST_RP_ADDRESS> 0.0.0.0 area <AREA>
  network <LOCAL_MSDP_PEER_ADDRESS> 0.0.0.0 area <AREA>
 exit
end
! =========================================================
! STATIC RP MAPPING
! Apply consistently on all multicast routers.
! =========================================================
conf t
 ip pim rp-address <ANYCAST_RP_ADDRESS> [<GROUP_ACL>]
end
! =========================================================
! PIM ON SOURCE-SIDE ROUTER
! =========================================================
conf t
 interface <SOURCE_FACING_INTERFACE>
  ip pim sparse-mode
 exit
 interface <TRANSIT_INTERFACE>
  ip pim sparse-mode
 exit
 ip pim rp-address <ANYCAST_RP_ADDRESS> [<GROUP_ACL>]
end
! =========================================================
! PIM ON LAST-HOP ROUTER
! =========================================================
conf t
 interface <TRANSIT_INTERFACE>
  ip pim sparse-mode
 exit
 interface <RECEIVER_FACING_INTERFACE>
  ip pim sparse-mode
 exit
 ip pim rp-address <ANYCAST_RP_ADDRESS> [<GROUP_ACL>]
end
! =========================================================
! MSDP PEERING
! Configure on each Anycast RP toward every other Anycast RP.
! Use unique loopbacks, not the shared Anycast RP loopback.
! =========================================================
conf t
 ip msdp peer <REMOTE_MSDP_PEER_ADDRESS> connect-source <UNIQUE_MSDP_LOOPBACK> remote-as <REMOTE_AS>
end
! =========================================================
! MSDP MESH GROUP
! Use for full-mesh Anycast RP core designs.
! Configure each remote MSDP peer as a mesh-group member.
! =========================================================
conf t
 ip msdp mesh-group <MSDP_MESH_GROUP_NAME> <REMOTE_MSDP_PEER_ADDRESS>
end
! =========================================================
! SIMPLE SINGLE-PEER MSDP DEFAULT PEER
! Use only in simple single-upstream or single-peer designs.
! Do not use this blindly in a full Anycast RP mesh.
! =========================================================
conf t
 ip msdp default-peer <REMOTE_MSDP_PEER_ADDRESS>
end
! =========================================================
! RECEIVER JOIN
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! TEST SOURCE TRAFFIC
! =========================================================
ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5
! =========================================================
! VERIFICATION
! =========================================================
show ip pim rp mapping
show ip pim interface
show ip pim neighbor
show ip msdp peer
show ip msdp summary
show ip msdp sa-cache
show ip igmp groups
show ip mroute <MULTICAST_GROUP>
show ip rpf <SOURCE_IP>
show ip route <ANYCAST_RP_ADDRESS>
show ip route <REMOTE_MSDP_PEER_ADDRESS>
! =========================================================
! CONTROLLED DEBUG
! Use briefly only during SA exchange testing.
! =========================================================
debug ip msdp
undebug all
# MSDP_Source_Active_Exchange_And_Anycast_RP_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| Multicast routing enabled | All multicast routers | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| Shared Anycast RP address | RP routers | `show ip interface brief` | Same `/32` RP address exists on every Anycast RP |
| Unique MSDP loopback | RP routers | `show ip interface brief` | Each RP has a different unique MSDP peering address |
| Route to Anycast RP | All multicast routers | `show ip route <ANYCAST_RP_ADDRESS>` | Route exists and points to nearest RP |
| Route to MSDP peer | RP routers | `show ip route <REMOTE_MSDP_PEER_ADDRESS>` | Route exists to remote MSDP peering address |
| PIM RP mapping | All multicast routers | `show ip pim rp mapping` | Test group maps to `<ANYCAST_RP_ADDRESS>` |
| PIM interface state | All multicast routers | `show ip pim interface` | Required RP, transit, source-facing, and receiver-facing interfaces appear |
| PIM neighbor state | Transit routers | `show ip pim neighbor` | Expected PIM neighbors appear |
| MSDP peer config | RP routers | `show running-config | include ^ip msdp` | MSDP peer and mesh group commands are present |
| MSDP peer state | RP routers | `show ip msdp peer` | Peer is established and using expected local and remote addresses |
| MSDP summary | RP routers | `show ip msdp summary` | Peer is up and counters are visible if supported |
| MSDP SA cache | RP routers | `show ip msdp sa-cache` | SA entries appear for active remote sources |
| Receiver membership | Last-hop router | `show ip igmp groups` | Receiver group appears on receiver-facing interface |
| Local source registration | Source-side RP | `show ip mroute <MULTICAST_GROUP>` | `(S,G)` state appears for source registered to local RP |
| Remote SA learning | Receiver-side RP | `show ip msdp sa-cache` | Remote source and group appear in SA cache |
| MSDP-created state | Receiver-side RP | `show ip mroute <MULTICAST_GROUP>` | Remote source information is visible and may show MSDP-created state |
| RPF toward source | RP and last-hop routers | `show ip rpf <SOURCE_IP>` | RPF path points toward multicast source |
| OIL toward receiver | Receiver-side RP and last-hop router | `show ip mroute <MULTICAST_GROUP>` | OIL points toward receiver path |
| Packet counters | Forwarding routers | `show ip mroute <MULTICAST_GROUP>` | Packet counters increment during source traffic |
| End-to-end multicast test | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Receiver replies when using `ip igmp join-group` and forwarding is correct |
| Peer-RPF troubleshooting input | RP routers | `show ip route <ORIGINATING_RP_ADDRESS>`<br>`show ip msdp peer` | Route and peer topology support SA acceptance |
| MSDP debug | RP routers | `debug ip msdp` | SA messages, peer events, and peer-RPF behavior appear during controlled testing |
| Debug cleanup | All routers | `show debugging`<br>`undebug all` | No multicast or MSDP debug remains active |
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
 no ip msdp default-peer <REMOTE_MSDP_PEER_ADDRESS>
end
! =========================================================
! REMOVE MSDP MESH GROUP
! =========================================================
conf t
 no ip msdp mesh-group <MSDP_MESH_GROUP_NAME> <REMOTE_MSDP_PEER_ADDRESS>
end
! =========================================================
! REMOVE MSDP PEER
! =========================================================
conf t
 no ip msdp peer <REMOTE_MSDP_PEER_ADDRESS> connect-source <UNIQUE_MSDP_LOOPBACK> remote-as <REMOTE_AS>
end
! =========================================================
! REMOVE STATIC RP MAPPING
! =========================================================
conf t
 no ip pim rp-address <ANYCAST_RP_ADDRESS>
 no ip pim rp-address <ANYCAST_RP_ADDRESS> <GROUP_ACL>
end
! =========================================================
! REMOVE PIM FROM ANYCAST RP LOOPBACK
! Only do this if no other multicast labs depend on it.
! =========================================================
conf t
 interface <ANYCAST_RP_LOOPBACK>
  no ip pim sparse-mode
 exit
end
! =========================================================
! REMOVE SHARED ANYCAST RP ADDRESS
! Only do this in a lab cleanup.
! =========================================================
conf t
 interface <ANYCAST_RP_LOOPBACK>
  no ip address <ANYCAST_RP_ADDRESS> 255.255.255.255
 exit
end
! =========================================================
! REMOVE UNIQUE MSDP LOOPBACK ADDRESS
! Only do this in a lab cleanup.
! =========================================================
conf t
 interface <UNIQUE_MSDP_LOOPBACK>
  no ip address <LOCAL_MSDP_PEER_ADDRESS> 255.255.255.255
 exit
end
! =========================================================
! REMOVE PIM FROM TRANSIT, SOURCE, AND RECEIVER INTERFACES
! Only do this if no other multicast labs depend on them.
! =========================================================
conf t
 interface <TRANSIT_INTERFACE>
  no ip pim sparse-mode
 exit
 interface <SOURCE_FACING_INTERFACE>
  no ip pim sparse-mode
 exit
 interface <RECEIVER_FACING_INTERFACE>
  no ip pim sparse-mode
 exit
end
! =========================================================
! REMOVE GLOBAL MULTICAST ROUTING
! Only do this if no other multicast features depend on it.
! =========================================================
conf t
 no ip multicast-routing
end
! =========================================================
! CLEAR STATE AND DEBUGGING
! =========================================================
clear ip mroute *
clear ip igmp group *
clear ip msdp sa-cache *
undebug all
# MSDP_Source_Active_Exchange_And_Anycast_RP_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| MSDP peer will not establish | No route to peer, wrong connect-source, ACL blocking TCP 639, or peer address mismatch | RP routers | `show ip msdp peer`<br>`show ip route <REMOTE_MSDP_PEER_ADDRESS>` | Fix routing to unique peering loopback and correct `connect-source` |
| MSDP peer sourced from shared Anycast RP address | Peering uses the Anycast RP loopback instead of unique loopback | RP routers | `show running-config | include ^ip msdp peer` | Reconfigure MSDP peer with unique loopback as `connect-source` |
| RP mapping missing | Static RP not configured consistently or group ACL does not match | All multicast routers | `show ip pim rp mapping`<br>`show running-config | include rp-address` | Configure `ip pim rp-address <ANYCAST_RP_ADDRESS>` consistently |
| Routers pick different RP addresses | Anycast RP address is not identical across RPs or static RP mapping differs | All multicast routers | `show ip pim rp mapping`<br>`show ip interface brief` | Use the same Anycast RP `/32` on all RP routers and same RP mapping everywhere |
| Traffic registers to one RP but receivers join another and fail | MSDP SA exchange missing between Anycast RPs | RP routers | `show ip msdp peer`<br>`show ip msdp sa-cache` | Establish MSDP peering between the Anycast RPs |
| SA cache empty on source-side RP | Source is not sending, source-side PIM is missing, or source did not register | Source-side RP | `show ip mroute <MULTICAST_GROUP>`<br>`show ip pim interface` | Generate source traffic and fix source-side PIM/RPF |
| SA cache empty on remote RP | MSDP peer down, SA filtered, peer-RPF failed, or mesh issue | Remote RP | `show ip msdp peer`<br>`show ip msdp sa-cache`<br>`debug ip msdp` | Fix MSDP session, filters, or peer-RPF topology |
| SA received but not accepted | Peer-RPF failure | RP router | `debug ip msdp`<br>`show ip route <ORIGINATING_RP_ADDRESS>` | Align MSDP peering with BGP or routing topology, or use mesh group/default-peer only when appropriate |
| SA loops or excessive SA flooding | Full mesh lacks mesh-group or topology is poorly designed | RP routers | `show running-config | include mesh-group`<br>`show ip msdp sa-cache` | Configure `ip msdp mesh-group` for full-mesh Anycast RP core |
| Default peer breaks expected policy | `ip msdp default-peer` bypasses normal peer-RPF expectation in a topology that needs explicit peer control | RP router | `show running-config | include default-peer` | Remove default peer unless this is a simple single-peer design |
| Anycast RP route is unstable | IGP path to shared Anycast RP address flaps | All multicast routers | `show ip route <ANYCAST_RP_ADDRESS>` | Stabilize IGP, loopback advertisement, and route metrics |
| BGP or IGP router ID collision | Shared Anycast RP address used as router ID | RP routers | `show ip protocols`<br>`show ip bgp summary` | Set explicit unique router IDs and use unique loopbacks for BGP/MSDP peering |
| Receiver group missing | Receiver did not send IGMP join or joined on wrong interface | Last-hop router and receiver | `show ip igmp groups`<br>`show running-config interface <RECEIVER_INTERFACE>` | Configure `ip igmp join-group <MULTICAST_GROUP>` on correct receiver interface |
| PIM neighbors missing | PIM not enabled or transit connectivity is broken | Transit routers | `show ip pim neighbor`<br>`show ip interface brief` | Enable `ip pim sparse-mode` and fix transit reachability |
| Incoming interface is wrong | RPF path to source points to wrong interface | RP and last-hop routers | `show ip rpf <SOURCE_IP>`<br>`show ip mroute <MULTICAST_GROUP>` | Correct unicast routing or multicast RPF path |
| OIL empty on receiver-side RP | Receiver join did not reach RP or RP mapping/RPF is broken | Receiver-side RP | `show ip mroute <MULTICAST_GROUP>`<br>`show ip igmp groups` | Fix receiver joins, RP mapping, and PIM join path |
| Packet counters stay at zero | Source not sending, SA not exchanged, receiver missing, or RPF broken | Source, RP, and last-hop routers | `show ip mroute <MULTICAST_GROUP>`<br>`show ip msdp sa-cache`<br>`show ip rpf <SOURCE_IP>` | Fix source traffic, MSDP SA exchange, receiver interest, or RPF |
| Clear command rejected | Platform uses different MSDP clear syntax | RP router | `clear ip msdp ?` | Use platform-supported clear command or wait for SA cache aging |
| Debug floods the console | MSDP debug left enabled | Any router | `show debugging` | Run `undebug all` |
##### Source_Basis
# MSDP_Source_Active_Exchange_And_Anycast_RP_Mental_Model
# MSDP_Source_Active_Exchange_And_Anycast_RP_Configuration_Checklist
# MSDP_Source_Active_Exchange_And_Anycast_RP_Skeleton
# MSDP_Source_Active_Exchange_And_Anycast_RP_Verification_Commands
# MSDP_Source_Active_Exchange_And_Anycast_RP_Rollback
# MSDP_Source_Active_Exchange_And_Anycast_RP_Failure_Checks


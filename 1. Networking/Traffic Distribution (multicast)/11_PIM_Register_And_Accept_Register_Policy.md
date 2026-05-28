
PIM_Register_And_Accept_Register_Policy.md

PIM_Register_And_Accept_Register_Policy



# PIM_Register_And_Accept_Register_Policy_Mental_Model
| Concept | Operational Meaning |
|---|---|
| PIM register | In PIM sparse mode, the first-hop DR near a source encapsulates initial multicast traffic and unicasts it to the RP as PIM register traffic |
| First-hop DR | The router connected to the source LAN is responsible for registering the active source with the RP |
| RP role | The RP is the meeting point where source registration and receiver shared-tree joins meet |
| Register tunnel | IOS can auto-create PIM tunnel interfaces used for register encapsulation and decapsulation |
| Register-stop | The RP can tell the first-hop DR to stop sending register messages |
| Normal register-stop reason | The RP may send register-stop when it starts receiving native multicast traffic over the source tree or when there are no interested receivers |
| Policy register-stop reason | If an unauthorized source/group attempts to register, the RP rejects it and immediately sends register-stop |
| Accept-register policy | `ip pim accept-register` is RP-side source admission control for PIM register messages |
| ACL meaning | The extended ACL matches source address as the packet source and multicast group as the packet destination |
| Permit logic | A permitted `(S,G)` pair is allowed to register with the RP |
| Deny logic | A denied `(S,G)` pair is rejected by the RP |
| Implicit deny risk | If the ACL lacks a final permit for expected traffic, valid sources may be denied |
| RP-only placement | The policy belongs on the RP or candidate RPs that receive PIM register messages |
| Not receiver filtering | This does not block IGMP joins from receivers. It blocks source registration at the RP |
| Not complex payload filtering | The accept-register ACL should match source and multicast destination only. Do not depend on protocol or port matches for this feature |
| Directly connected RP-source caveat | If the RP is also the first-hop DR for directly connected sources, accept-register is not the right filter point. Use multicast boundary or interface policy instead |
# PIM_Register_And_Accept_Register_Policy_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the RP router and RP address | All multicast routers | `show ip pim rp mapping` | Test group maps to the expected RP |
| 2 | Identify the allowed multicast source and group range | RP router | `show ip mroute` | Existing or planned `(S,G)` pairs are known |
| 3 | Identify the source-side first-hop DR | Source-side routers | `show ip pim interface <SOURCE_LAN_INTERFACE>` | Source LAN PIM DR is known |
| 4 | Confirm multicast routing is enabled | All multicast routers | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| 5 | Enable multicast routing globally if missing | All multicast routers | `conf t`<br>`ip multicast-routing` | Router can build multicast control-plane and forwarding state |
| 6 | Confirm PIM sparse mode on source-facing interface | First-hop DR | `show running-config interface <SOURCE_LAN_INTERFACE>` | Interface contains `ip pim sparse-mode` |
| 7 | Enable PIM sparse mode on source-facing interface if missing | First-hop DR | `conf t`<br>`interface <SOURCE_LAN_INTERFACE>`<br>`ip pim sparse-mode` | Source LAN can register multicast sources |
| 8 | Confirm PIM sparse mode on RP loopback and transit interfaces | RP router | `show ip pim interface` | RP loopback and transit interfaces appear as PIM interfaces |
| 9 | Enable PIM sparse mode on RP loopback if missing | RP router | `conf t`<br>`interface <RP_LOOPBACK_INTERFACE>`<br>`ip pim sparse-mode` | RP address participates in multicast control plane |
| 10 | Configure static RP mapping if the lab does not use Auto-RP or BSR | All multicast routers | `conf t`<br>`ip pim rp-address <RP_ADDRESS> [<GROUP_ACL>]` | All routers map the test group to the RP |
| 11 | Verify RP mapping before applying register policy | All multicast routers | `show ip pim rp mapping` | `<MULTICAST_GROUP>` maps to `<RP_ADDRESS>` |
| 12 | Confirm source-side route to the RP | First-hop DR | `show ip route <RP_ADDRESS>` | First-hop DR can unicast PIM register traffic to the RP |
| 13 | Confirm RP route to the source | RP router | `show ip route <SOURCE_IP>` | RP has a route back toward the source |
| 14 | Create an extended ACL for allowed register traffic | RP router | `conf t`<br>`ip access-list extended <REGISTER_FILTER_ACL>` | Extended ACL configuration mode opens |
| 15 | Permit trusted source/group register traffic | RP router | `permit ip <ALLOWED_SOURCE_NETWORK> <SOURCE_WILDCARD> <ALLOWED_MULTICAST_GROUP_RANGE> <GROUP_WILDCARD>` | Trusted `(S,G)` register traffic is permitted |
| 16 | Deny untrusted source/group register traffic if testing explicit denial | RP router | `deny ip <DENIED_SOURCE_NETWORK> <SOURCE_WILDCARD> <DENIED_MULTICAST_GROUP_RANGE> <GROUP_WILDCARD>` | Untrusted `(S,G)` register traffic is denied |
| 17 | Add final permit for all other expected register traffic if policy is deny-specific | RP router | `permit ip any any` | Unrelated valid register traffic is not accidentally blocked |
| 18 | Exit ACL configuration mode | RP router | `exit` | Router returns to global configuration mode |
| 19 | Apply accept-register policy on the RP | RP router | `ip pim accept-register list <REGISTER_FILTER_ACL>` | RP filters PIM register messages using the ACL |
| 20 | Apply the same policy on every candidate RP if multiple RPs may receive registers | Candidate RP routers | `ip pim accept-register list <REGISTER_FILTER_ACL>` | All candidate RPs enforce the same register policy |
| 21 | Verify accept-register configuration | RP router | `show running-config | include accept-register` | `ip pim accept-register list <REGISTER_FILTER_ACL>` is present |
| 22 | Verify ACL contents and order | RP router | `show access-lists <REGISTER_FILTER_ACL>` | Permit and deny entries match intended `(S,G)` policy |
| 23 | Clear old multicast state before policy testing | All multicast routers | `clear ip mroute *` | Stale `(S,G)` and `(*,G)` state is removed |
| 24 | Create receiver interest for an allowed group | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <ALLOWED_MULTICAST_GROUP>` | Receiver joins the allowed multicast group |
| 25 | Verify receiver shared-tree state | Last-hop router and RP | `show ip mroute <ALLOWED_MULTICAST_GROUP>` | `(*,<GROUP>)` state exists toward the RP |
| 26 | Generate allowed source traffic | Allowed source or first-hop test router | `ping <ALLOWED_MULTICAST_GROUP> source <ALLOWED_SOURCE_IP_OR_INTERFACE> repeat 5` | Trusted source begins sending multicast traffic |
| 27 | Verify allowed source registers successfully | RP router | `show ip mroute <ALLOWED_MULTICAST_GROUP>` | `(S,G)` state appears for the allowed source |
| 28 | Verify PIM register tunnel behavior | First-hop DR and RP router | `show ip pim tunnel` | First-hop router shows PIM encap tunnel and RP shows decap behavior if platform displays it |
| 29 | Verify traffic forwards to receiver | Source/test router and multicast routers | `ping <ALLOWED_MULTICAST_GROUP> source <ALLOWED_SOURCE_IP_OR_INTERFACE> repeat 5`<br>`show ip mroute <ALLOWED_MULTICAST_GROUP>` | Receiver replies or packet counters increment |
| 30 | Generate denied source traffic for the denied group | Denied source or first-hop test router | `ping <DENIED_MULTICAST_GROUP> source <DENIED_SOURCE_IP_OR_INTERFACE> repeat 5` | Untrusted source attempts to register |
| 31 | Verify denied source does not create accepted forwarding state on the RP | RP router | `show ip mroute <DENIED_MULTICAST_GROUP>` | Denied `(S,G)` is absent, short-lived, or has no useful OIL toward receivers |
| 32 | Check PIM statistics during allowed and denied tests | RP and first-hop DR | `show ip traffic | section PIM` | PIM register and register-stop activity changes during testing |
| 33 | Use controlled PIM debugging only if output verification is not clear | RP router | `debug ip pim` | Register or register-stop behavior is visible during the test |
| 34 | Disable debugging immediately after the test | RP router | `undebug all` | Debugging is disabled |
| 35 | Save the working register policy | RP and candidate RP routers | `copy running-config startup-config` | PIM accept-register policy is preserved |
# PIM_Register_And_Accept_Register_Policy_Skeleton
! =========================================================
! VARIABLES
! =========================================================
! <RP_ADDRESS>                         = RP loopback or stable RP address
! <RP_LOOPBACK_INTERFACE>              = RP loopback interface
! <GROUP_ACL>                          = optional RP group ACL
! <SOURCE_LAN_INTERFACE>               = first-hop DR interface toward source LAN
! <RECEIVER_INTERFACE>                 = receiver/test router interface
! <REGISTER_FILTER_ACL>                = extended ACL used by accept-register
! <ALLOWED_SOURCE_NETWORK>             = trusted source network
! <DENIED_SOURCE_NETWORK>              = untrusted source network
! <SOURCE_WILDCARD>                    = source wildcard mask
! <ALLOWED_MULTICAST_GROUP_RANGE>      = allowed multicast group range
! <DENIED_MULTICAST_GROUP_RANGE>       = denied multicast group range
! <GROUP_WILDCARD>                     = multicast group wildcard mask
! <ALLOWED_MULTICAST_GROUP>            = allowed test group
! <DENIED_MULTICAST_GROUP>             = denied test group
! <ALLOWED_SOURCE_IP_OR_INTERFACE>     = trusted source address or interface
! <DENIED_SOURCE_IP_OR_INTERFACE>      = untrusted source address or interface
! =========================================================
! GLOBAL MULTICAST BASELINE
! Apply on every multicast router
! =========================================================
conf t
 ip multicast-routing
end
! =========================================================
! FIRST-HOP DR BASELINE
! =========================================================
conf t
 interface <SOURCE_LAN_INTERFACE>
  ip pim sparse-mode
 exit
 ip pim rp-address <RP_ADDRESS> [<GROUP_ACL>]
end
! =========================================================
! RP ROUTER BASELINE
! =========================================================
conf t
 interface <RP_LOOPBACK_INTERFACE>
  ip pim sparse-mode
 exit
 ip pim rp-address <RP_ADDRESS> [<GROUP_ACL>]
end
! =========================================================
! OTHER MULTICAST ROUTERS
! =========================================================
conf t
 ip pim rp-address <RP_ADDRESS> [<GROUP_ACL>]
end
! =========================================================
! RP REGISTER FILTER ACL
! Extended ACL source field = multicast source
! Extended ACL destination field = multicast group
! =========================================================
conf t
 ip access-list extended <REGISTER_FILTER_ACL>
  permit ip <ALLOWED_SOURCE_NETWORK> <SOURCE_WILDCARD> <ALLOWED_MULTICAST_GROUP_RANGE> <GROUP_WILDCARD>
  deny ip <DENIED_SOURCE_NETWORK> <SOURCE_WILDCARD> <DENIED_MULTICAST_GROUP_RANGE> <GROUP_WILDCARD>
  permit ip any any
 exit
end
! =========================================================
! APPLY ACCEPT-REGISTER POLICY ON RP
! Apply on all candidate RPs that may receive PIM registers.
! =========================================================
conf t
 ip pim accept-register list <REGISTER_FILTER_ACL>
end
! =========================================================
! RECEIVER JOIN FOR ALLOWED TEST
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  ip igmp join-group <ALLOWED_MULTICAST_GROUP>
 exit
end
! =========================================================
! OPTIONAL RECEIVER JOIN FOR DENIED TEST
! Use only when proving the RP blocks source registration even when
! receiver interest exists.
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  ip igmp join-group <DENIED_MULTICAST_GROUP>
 exit
end
! =========================================================
! CLEAR STATE BEFORE TESTING
! =========================================================
clear ip mroute *
! =========================================================
! ALLOWED SOURCE TEST
! =========================================================
ping <ALLOWED_MULTICAST_GROUP> source <ALLOWED_SOURCE_IP_OR_INTERFACE> repeat 5
! =========================================================
! DENIED SOURCE TEST
! =========================================================
ping <DENIED_MULTICAST_GROUP> source <DENIED_SOURCE_IP_OR_INTERFACE> repeat 5
! =========================================================
! VERIFICATION
! =========================================================
show access-lists <REGISTER_FILTER_ACL>
show running-config | include accept-register
show ip pim rp mapping
show ip pim tunnel
show ip mroute <ALLOWED_MULTICAST_GROUP>
show ip mroute <DENIED_MULTICAST_GROUP>
show ip traffic | section PIM
! =========================================================
! CONTROLLED DEBUG
! Use briefly only if show commands are not enough.
! =========================================================
debug ip pim
undebug all
# PIM_Register_And_Accept_Register_Policy_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| Multicast routing enabled | All multicast routers | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| PIM interface state | All multicast routers | `show ip pim interface` | Required source, transit, RP, and receiver interfaces appear |
| RP mapping | All multicast routers | `show ip pim rp mapping` | Test groups map to `<RP_ADDRESS>` |
| Route from first-hop DR to RP | First-hop DR | `show ip route <RP_ADDRESS>` | Route to RP exists |
| Route from RP to source | RP router | `show ip route <SOURCE_IP>` | Route to source exists |
| Register filter applied | RP router | `show running-config | include accept-register` | `ip pim accept-register list <REGISTER_FILTER_ACL>` appears |
| Register ACL contents | RP router | `show access-lists <REGISTER_FILTER_ACL>` | ACL permits trusted `(S,G)` and denies untrusted `(S,G)` as intended |
| Receiver interest | Last-hop router | `show ip igmp groups` | Receiver group appears on receiver-facing interface |
| Shared-tree state | RP and last-hop router | `show ip mroute <MULTICAST_GROUP>` | `(*,G)` state exists when receivers have joined |
| Allowed source registration | RP router | `show ip mroute <ALLOWED_MULTICAST_GROUP>` | `(ALLOWED_SOURCE,G)` state appears and forwards as expected |
| Denied source registration | RP router | `show ip mroute <DENIED_MULTICAST_GROUP>` | Denied `(S,G)` does not remain as valid forwarding state |
| First-hop register tunnel | First-hop DR | `show ip pim tunnel` | PIM encap tunnel toward RP appears if platform displays it |
| RP decap tunnel | RP router | `show ip pim tunnel` | PIM decap tunnel appears if platform displays it |
| PIM statistics | RP and first-hop DR | `show ip traffic | section PIM` | Register and register-stop counters change during tests |
| Allowed traffic counters | RP and transit routers | `show ip mroute <ALLOWED_MULTICAST_GROUP>` | Packet counters increment |
| Denied traffic forwarding | RP and last-hop router | `show ip mroute <DENIED_MULTICAST_GROUP>` | No receiver forwarding occurs for denied source/group |
| Controlled debug | RP router | `debug ip pim` | Register and register-stop behavior is visible during active testing |
| Debug cleanup | RP router | `show debugging`<br>`undebug all` | No multicast debug remains active |
# PIM_Register_And_Accept_Register_Policy_Rollback
! =========================================================
! REMOVE TEST RECEIVER JOINS
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  no ip igmp join-group <ALLOWED_MULTICAST_GROUP>
  no ip igmp join-group <DENIED_MULTICAST_GROUP>
 exit
end
! =========================================================
! REMOVE ACCEPT-REGISTER POLICY FROM RP
! =========================================================
conf t
 no ip pim accept-register list <REGISTER_FILTER_ACL>
end
! =========================================================
! REMOVE REGISTER FILTER ACL
! =========================================================
conf t
 no ip access-list extended <REGISTER_FILTER_ACL>
end
! =========================================================
! REMOVE STATIC RP CONFIGURATION IF USED ONLY FOR THIS LAB
! =========================================================
conf t
 no ip pim rp-address <RP_ADDRESS>
 no ip pim rp-address <RP_ADDRESS> <GROUP_ACL>
end
! =========================================================
! REMOVE PIM FROM SOURCE LAN INTERFACE
! Only do this if no other multicast labs depend on it.
! =========================================================
conf t
 interface <SOURCE_LAN_INTERFACE>
  no ip pim sparse-mode
 exit
end
! =========================================================
! REMOVE PIM FROM RP LOOPBACK
! Only do this if no other multicast labs depend on it.
! =========================================================
conf t
 interface <RP_LOOPBACK_INTERFACE>
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
! CLEAR MULTICAST STATE AND DEBUGGING
! =========================================================
clear ip mroute *
clear ip igmp group *
undebug all
# PIM_Register_And_Accept_Register_Policy_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Accept-register command is rejected | Platform syntax does not support route-map or only supports ACL form | RP router | `ip pim accept-register ?` | Use supported syntax, usually `ip pim accept-register list <ACL>` on IOS XE switching platforms |
| Valid source is blocked | ACL entry missing or ACL order is wrong | RP router | `show access-lists <REGISTER_FILTER_ACL>` | Add a permit for the valid source/group before broader deny entries |
| All sources are blocked | ACL lacks final permit and implicit deny catches everything else | RP router | `show access-lists <REGISTER_FILTER_ACL>` | Add `permit ip any any` if the policy is deny-specific |
| Denied source still forwards | Policy is not applied on the active RP or wrong RP is selected | RP and all routers | `show running-config | include accept-register`<br>`show ip pim rp mapping` | Apply policy on the actual RP and fix RP mapping |
| Denied source still forwards when directly connected to RP | RP is also the first-hop DR for that source | RP router | `show ip pim interface <SOURCE_LAN_INTERFACE>`<br>`show ip mroute <GROUP>` | Use `ip multicast boundary` or an interface policy instead |
| ACL matches wrong values | ACL was written for receiver IPs instead of source and group | RP router | `show access-lists <REGISTER_FILTER_ACL>` | Rewrite ACL so source field matches multicast source and destination field matches multicast group |
| No register tunnel appears | No active source traffic, no RP mapping, or platform hides tunnel output | First-hop DR and RP | `show ip pim tunnel`<br>`show ip mroute <GROUP>` | Generate source traffic and verify RP mapping |
| No `(S,G)` appears for allowed source | Source traffic is not reaching first-hop DR or register is blocked | First-hop DR and RP | `show ip mroute <GROUP>`<br>`show ip traffic | section PIM` | Verify source traffic, ACL permit, and route to RP |
| RP mapping missing | Static RP, Auto-RP, or BSR not configured correctly | All multicast routers | `show ip pim rp mapping` | Configure consistent RP mapping |
| First-hop DR cannot register to RP | No unicast route from first-hop DR to RP | First-hop DR | `show ip route <RP_ADDRESS>` | Fix unicast routing to RP |
| RP cannot build source path | No unicast route or RPF path from RP to source | RP router | `show ip route <SOURCE_IP>`<br>`show ip rpf <SOURCE_IP>` | Fix routing or multicast RPF path toward source |
| Receiver joined but traffic fails | Register policy may be fine, but receiver tree or RPF is broken | Last-hop router and RP | `show ip igmp groups`<br>`show ip mroute <GROUP>`<br>`show ip rpf <SOURCE_IP>` | Fix receiver IGMP, PIM joins, RP path, or source RPF |
| Register-stop seen for denied source | Expected behavior for unauthorized source/group | RP router | `show ip traffic | section PIM`<br>`debug ip pim` | No fix needed if denial is intended |
| Register-stop seen for allowed source with no receivers | Normal PIM behavior when RP has no active shared tree for the group | RP router | `show ip mroute <GROUP>` | Add receiver interest or accept that traffic is stopped |
| PIM debug floods the console | Debug left enabled during testing | Any router | `show debugging` | Run `undebug all` |
##### Source_Basis
# PIM_Register_And_Accept_Register_Policy_Mental_Model
# PIM_Register_And_Accept_Register_Policy_Configuration_Checklist
# PIM_Register_And_Accept_Register_Policy_Skeleton
# PIM_Register_And_Accept_Register_Policy_Verification_Commands
# PIM_Register_And_Accept_Register_Policy_Rollback
# PIM_Register_And_Accept_Register_Policy_Failure_Checks

Cisco’s command reference confirms that ip pim accept-register is configured on candidate RPs to filter PIM register messages, that the ACL or route-map should filter on source and multicast destination, and that unauthorized registers cause an immediate register-stop.   IOS XE switching syntax also documents the ACL-only form, ip pim [vrf <name>] accept-register list <access-list>.  


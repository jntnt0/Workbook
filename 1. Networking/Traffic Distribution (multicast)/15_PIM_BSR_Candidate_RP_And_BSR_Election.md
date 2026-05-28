

PIM_BSR_Candidate_RP_And_BSR_Election.md

PIM_BSR_Candidate_RP_And_BSR_Election

# PIM_BSR_Candidate_RP_And_BSR_Election_Mental_Model
| Concept                      | Operational Meaning                                                                                                                              |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| BSR                          | Bootstrap Router distributes RP-set information through the PIM domain                                                                           |
| Candidate BSR                | Router that participates in BSR election and may become the elected Bootstrap Router                                                             |
| Candidate RP                 | Router that advertises itself as an available RP for one or more multicast group ranges                                                          |
| RP-set                       | List of Candidate RPs and group ranges collected by the elected BSR and advertised to all PIM routers                                            |
| BSR election                 | Higher BSR priority wins. If priority ties, the higher BSR address wins                                                                          |
| Candidate RP preference      | Lower Candidate RP priority is preferred when routers select an RP from the RP-set                                                               |
| BSR versus Auto-RP           | BSR is native PIMv2. Auto-RP is Cisco proprietary. Do not mix both in the same multicast domain unless the lab specifically tests interoperation |
| No dense fallback dependency | Unlike classic Auto-RP, BSR does not require dense-mode flooding of 224.0.1.39 or 224.0.1.40                                                     |
| BSR control group            | BSR uses PIMv2 bootstrap messaging, commonly associated with 224.0.0.13 hop-by-hop PIM control                                                   |
| Candidate RP advertisement   | Candidate RP advertises itself to the elected BSR                                                                                                |
| Bootstrap message            | Elected BSR floods RP-set information through the PIM domain                                                                                     |
| Hash mask length             | Controls RP load-sharing granularity across group ranges when multiple RPs exist                                                                 |
| Group-list ACL               | Defines which multicast groups a Candidate RP is willing to serve                                                                                |
| Verification truth           | `show ip pim bsr-router` proves the elected BSR. `show ip pim rp mapping` proves the RP-set and selected RP information                          |
| Failure pattern              | If BSR election works but RP mapping is missing, Candidate RP advertisement, group-list ACL, or PIM reachability is usually broken               |
# PIM_BSR_Candidate_RP_And_BSR_Election_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify Candidate RP routers, Candidate BSR routers, source routers, receiver routers, and multicast group range | All multicast routers | `show ip interface brief` | RP loopbacks, BSR loopbacks, transit links, source-facing interfaces, and receiver-facing interfaces are known |
| 2 | Confirm unicast reachability to Candidate RP loopbacks | All multicast routers | `show ip route <CANDIDATE_RP_ADDRESS>` | All routers have routes to Candidate RP addresses |
| 3 | Confirm unicast reachability to Candidate BSR loopbacks | All multicast routers | `show ip route <CANDIDATE_BSR_ADDRESS>` | All routers have routes to Candidate BSR addresses |
| 4 | Confirm source reachability for multicast RPF | RP and last-hop routers | `show ip route <SOURCE_IP>` | Routers have routes toward the multicast source |
| 5 | Remove or avoid static RP and Auto-RP overlap in the same test domain | All multicast routers | `show running-config | include rp-address|send-rp|bsr-candidate|rp-candidate` | No unintended static RP or Auto-RP configuration conflicts with BSR |
| 6 | Enable multicast routing globally | All multicast routers | `conf t`<br>`ip multicast-routing` | Router can build multicast control-plane and forwarding state |
| 7 | Enable PIM sparse mode on Candidate RP loopbacks | Candidate RP routers | `conf t`<br>`interface <CANDIDATE_RP_LOOPBACK>`<br>`ip pim sparse-mode` | Candidate RP loopback participates in PIM |
| 8 | Enable PIM sparse mode on Candidate BSR loopbacks | Candidate BSR routers | `conf t`<br>`interface <CANDIDATE_BSR_LOOPBACK>`<br>`ip pim sparse-mode` | Candidate BSR loopback participates in PIM |
| 9 | Enable PIM sparse mode on transit interfaces | All multicast transit routers | `conf t`<br>`interface <TRANSIT_INTERFACE>`<br>`ip pim sparse-mode` | PIM neighbors can form across routed links |
| 10 | Enable PIM sparse mode on source-facing interface | First-hop/source-side router | `conf t`<br>`interface <SOURCE_FACING_INTERFACE>`<br>`ip pim sparse-mode` | Source-side router participates in PIM-SM |
| 11 | Enable PIM sparse mode on receiver-facing interface | Last-hop router | `conf t`<br>`interface <RECEIVER_FACING_INTERFACE>`<br>`ip pim sparse-mode` | Last-hop router can process IGMP joins and PIM joins |
| 12 | Verify PIM interface state | All multicast routers | `show ip pim interface` | Required loopbacks, transit, source-facing, and receiver-facing interfaces appear |
| 13 | Verify PIM neighbor state | Transit routers | `show ip pim neighbor` | Expected PIM neighbors appear |
| 14 | Create Candidate RP group-list ACL | Candidate RP routers | `conf t`<br>`access-list <BSR_GROUP_ACL> permit <MULTICAST_GROUP_RANGE> <GROUP_WILDCARD>` | ACL matches multicast groups this RP should serve |
| 15 | Configure primary Candidate RP with lower preferred RP priority | Primary Candidate RP | `conf t`<br>`ip pim rp-candidate <PRIMARY_RP_SOURCE_INTERFACE> group-list <BSR_GROUP_ACL> interval <RP_ADV_INTERVAL> priority <PRIMARY_RP_PRIORITY>` | Primary Candidate RP advertises itself for the selected group range |
| 16 | Configure backup Candidate RP with higher RP priority if deterministic backup is required | Backup Candidate RP | `conf t`<br>`ip pim rp-candidate <BACKUP_RP_SOURCE_INTERFACE> group-list <BSR_GROUP_ACL> interval <RP_ADV_INTERVAL> priority <BACKUP_RP_PRIORITY>` | Backup Candidate RP advertises itself but loses if primary has lower RP priority |
| 17 | Configure primary Candidate BSR with higher BSR priority | Primary Candidate BSR | `conf t`<br>`ip pim bsr-candidate <PRIMARY_BSR_SOURCE_INTERFACE> <HASH_MASK_LENGTH> <PRIMARY_BSR_PRIORITY>` | Primary Candidate BSR is preferred in BSR election |
| 18 | Configure backup Candidate BSR with lower BSR priority | Backup Candidate BSR | `conf t`<br>`ip pim bsr-candidate <BACKUP_BSR_SOURCE_INTERFACE> <HASH_MASK_LENGTH> <BACKUP_BSR_PRIORITY>` | Backup Candidate BSR participates but should not win while primary is present |
| 19 | Verify elected BSR | All multicast routers | `show ip pim bsr-router` | Same BSR address appears across the multicast domain |
| 20 | Verify local Candidate BSR status | Candidate BSR routers | `show ip pim bsr-router` | Candidate BSR routers show local candidate information |
| 21 | Verify RP-set and Candidate RP advertisements | All multicast routers | `show ip pim rp mapping` | Candidate RPs appear via bootstrap with group range, priority, holdtime, and info source |
| 22 | Verify selected RP behavior for the test group | All multicast routers | `show ip pim rp mapping` | Test group maps to the intended RP based on BSR RP-set selection |
| 23 | Enable controlled BSR debugging only if election or RP-set is unclear | Candidate RP and Candidate BSR routers | `debug ip pim bsr` | Candidate RP advertisements and bootstrap messages are visible |
| 24 | Disable debugging after confirming behavior | Candidate RP and Candidate BSR routers | `undebug all` | Debugging is disabled |
| 25 | Configure receiver membership for a BSR-learned group | Receiver/test router | `conf t`<br>`interface <RECEIVER_INTERFACE>`<br>`ip igmp join-group <MULTICAST_GROUP>` | Receiver joins a group covered by BSR RP-set |
| 26 | Verify receiver membership | Last-hop router | `show ip igmp groups` | Joined group appears on the receiver-facing interface |
| 27 | Generate multicast traffic from the source | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Source sends multicast traffic to the BSR-learned group |
| 28 | Verify sparse-mode multicast state | RP, transit, and last-hop routers | `show ip mroute <MULTICAST_GROUP>` | `(*,G)` and/or `(S,G)` state exists |
| 29 | Verify RPF toward RP and source | Last-hop, RP, and transit routers | `show ip rpf <SELECTED_RP_ADDRESS>`<br>`show ip rpf <SOURCE_IP>` | RPF paths are valid for shared-tree and source-tree state |
| 30 | Test BSR failover by removing or shutting the primary Candidate BSR only in lab conditions | Primary Candidate BSR | `conf t`<br>`no ip pim bsr-candidate <PRIMARY_BSR_SOURCE_INTERFACE> <HASH_MASK_LENGTH> <PRIMARY_BSR_PRIORITY>` | Backup Candidate BSR becomes elected after convergence |
| 31 | Verify backup BSR election | All multicast routers | `show ip pim bsr-router` | Backup BSR address appears as the elected BSR |
| 32 | Restore primary Candidate BSR if needed | Primary Candidate BSR | `conf t`<br>`ip pim bsr-candidate <PRIMARY_BSR_SOURCE_INTERFACE> <HASH_MASK_LENGTH> <PRIMARY_BSR_PRIORITY>` | Primary BSR participates again and may reclaim election if priority is higher |
| 33 | Save the working BSR configuration | All changed routers | `copy running-config startup-config` | BSR, Candidate RP, and multicast forwarding baseline is preserved |
# PIM_BSR_Candidate_RP_And_BSR_Election_Skeleton
! =========================================================
! VARIABLES
! =========================================================
! <CANDIDATE_RP_ADDRESS>          = Candidate RP loopback address
! <CANDIDATE_RP_LOOPBACK>         = Candidate RP loopback interface
! <CANDIDATE_BSR_ADDRESS>         = Candidate BSR loopback address
! <CANDIDATE_BSR_LOOPBACK>        = Candidate BSR loopback interface
! <PRIMARY_RP_SOURCE_INTERFACE>   = primary Candidate RP source interface
! <BACKUP_RP_SOURCE_INTERFACE>    = backup Candidate RP source interface
! <PRIMARY_BSR_SOURCE_INTERFACE>  = primary Candidate BSR source interface
! <BACKUP_BSR_SOURCE_INTERFACE>   = backup Candidate BSR source interface
! <TRANSIT_INTERFACE>             = routed interface between PIM routers
! <SOURCE_FACING_INTERFACE>       = first-hop router interface toward source
! <RECEIVER_FACING_INTERFACE>     = last-hop router interface toward receiver LAN
! <RECEIVER_INTERFACE>            = test receiver interface
! <BSR_GROUP_ACL>                 = standard ACL matching multicast group range
! <MULTICAST_GROUP_RANGE>         = multicast range advertised by Candidate RP
! <GROUP_WILDCARD>                = wildcard mask for multicast group range
! <MULTICAST_GROUP>               = test multicast group inside advertised range
! <SOURCE_IP>                     = multicast source IP
! <SOURCE_INTERFACE_OR_IP>        = source interface or IP for test traffic
! <SELECTED_RP_ADDRESS>           = RP selected from BSR RP-set
! <HASH_MASK_LENGTH>              = BSR hash mask length, commonly 0 in simple labs
! <RP_ADV_INTERVAL>               = Candidate RP advertisement interval, example 30
! <PRIMARY_RP_PRIORITY>           = lower value preferred, example 0
! <BACKUP_RP_PRIORITY>            = higher value less preferred, example 200
! <PRIMARY_BSR_PRIORITY>          = higher value preferred, example 100
! <BACKUP_BSR_PRIORITY>           = lower value less preferred, example 0
! =========================================================
! GLOBAL MULTICAST ENABLEMENT
! Apply on every multicast router
! =========================================================
conf t
 ip multicast-routing
end
! =========================================================
! PIM SPARSE MODE BASELINE
! Apply on Candidate RP loopbacks, Candidate BSR loopbacks,
! transit links, source-facing interfaces, and receiver-facing interfaces.
! =========================================================
conf t
 interface <CANDIDATE_RP_LOOPBACK>
  ip pim sparse-mode
 exit
 interface <CANDIDATE_BSR_LOOPBACK>
  ip pim sparse-mode
 exit
 interface <TRANSIT_INTERFACE>
  ip pim sparse-mode
 exit
 interface <SOURCE_FACING_INTERFACE>
  ip pim sparse-mode
 exit
 interface <RECEIVER_FACING_INTERFACE>
  ip pim sparse-mode
 exit
end
! =========================================================
! CANDIDATE RP GROUP RANGE
! Standard ACL matches multicast groups.
! =========================================================
conf t
 access-list <BSR_GROUP_ACL> permit <MULTICAST_GROUP_RANGE> <GROUP_WILDCARD>
end
! =========================================================
! PRIMARY CANDIDATE RP
! Lower RP priority is preferred.
! =========================================================
conf t
 ip pim rp-candidate <PRIMARY_RP_SOURCE_INTERFACE> group-list <BSR_GROUP_ACL> interval <RP_ADV_INTERVAL> priority <PRIMARY_RP_PRIORITY>
end
! =========================================================
! BACKUP CANDIDATE RP
! Higher RP priority is less preferred.
! =========================================================
conf t
 ip pim rp-candidate <BACKUP_RP_SOURCE_INTERFACE> group-list <BSR_GROUP_ACL> interval <RP_ADV_INTERVAL> priority <BACKUP_RP_PRIORITY>
end
! =========================================================
! PRIMARY CANDIDATE BSR
! Higher BSR priority is preferred.
! If BSR priority ties, higher BSR address wins.
! =========================================================
conf t
 ip pim bsr-candidate <PRIMARY_BSR_SOURCE_INTERFACE> <HASH_MASK_LENGTH> <PRIMARY_BSR_PRIORITY>
end
! =========================================================
! BACKUP CANDIDATE BSR
! =========================================================
conf t
 ip pim bsr-candidate <BACKUP_BSR_SOURCE_INTERFACE> <HASH_MASK_LENGTH> <BACKUP_BSR_PRIORITY>
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
! TEST TRAFFIC
! =========================================================
ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5
! =========================================================
! VERIFICATION
! =========================================================
show ip pim interface
show ip pim neighbor
show ip pim bsr-router
show ip pim rp mapping
show ip igmp groups
show ip mroute <MULTICAST_GROUP>
show ip rpf <SELECTED_RP_ADDRESS>
show ip rpf <SOURCE_IP>
! =========================================================
! CONTROLLED BSR DEBUG
! Use briefly only during election or RP-set testing.
! =========================================================
debug ip pim bsr
undebug all
# PIM_BSR_Candidate_RP_And_BSR_Election_Verification_Commands
| Check | Device | Command | Good Output |
|---|---|---|---|
| Multicast routing enabled | All multicast routers | `show running-config | include ip multicast-routing` | `ip multicast-routing` is present |
| PIM interface state | All multicast routers | `show ip pim interface` | Candidate RP loopback, Candidate BSR loopback, transit, source-facing, and receiver-facing interfaces appear |
| PIM neighbor state | Transit routers | `show ip pim neighbor` | Expected PIM neighbors appear on routed transit links |
| Candidate RP configuration | Candidate RP routers | `show running-config | include rp-candidate` | Candidate RP commands show source interface, group-list, interval, and priority |
| Candidate BSR configuration | Candidate BSR routers | `show running-config | include bsr-candidate` | Candidate BSR commands show source interface, hash mask, and priority |
| Group-list ACL | Candidate RP routers | `show access-lists <BSR_GROUP_ACL>` | ACL permits intended multicast group range |
| Elected BSR | All multicast routers | `show ip pim bsr-router` | Same BSR address appears across routers |
| Local Candidate BSR role | Candidate BSR routers | `show ip pim bsr-router` | Router shows candidate BSR status or elected BSR status |
| RP-set distribution | All multicast routers | `show ip pim rp mapping` | Candidate RPs appear via bootstrap |
| Candidate RP priority | All multicast routers | `show ip pim rp mapping` | Primary RP has lower preferred priority than backup RP |
| BSR info source | All multicast routers | `show ip pim rp mapping` | Info source points to elected BSR address via bootstrap |
| BSR holdtime | All multicast routers | `show ip pim bsr-router` | BSR expiration or next bootstrap timer is present |
| Receiver membership | Last-hop router | `show ip igmp groups` | Receiver group appears on receiver-facing interface |
| Shared-tree state | Last-hop, transit, and RP routers | `show ip mroute <MULTICAST_GROUP>` | `(*,G)` state appears for the BSR-learned group |
| Source-specific state | RP, transit, and last-hop routers | `show ip mroute <MULTICAST_GROUP>` | `(S,G)` state appears after source traffic starts |
| RPF toward selected RP | Last-hop and transit routers | `show ip rpf <SELECTED_RP_ADDRESS>` | RPF path points toward selected RP |
| RPF toward source | RP, transit, and last-hop routers | `show ip rpf <SOURCE_IP>` | RPF path points toward multicast source |
| Packet counters | All forwarding routers | `show ip mroute <MULTICAST_GROUP>` | Packet counters increment during test traffic |
| End-to-end multicast test | Source/test router | `ping <MULTICAST_GROUP> source <SOURCE_INTERFACE_OR_IP> repeat 5` | Receiver replies if using `ip igmp join-group` and forwarding is correct |
| BSR debug | Candidate RP and Candidate BSR routers | `debug ip pim bsr` | Candidate RP advertisement and bootstrap messages appear during controlled test |
| Debug cleanup | All routers | `show debugging`<br>`undebug all` | No multicast debug remains active |
# PIM_BSR_Candidate_RP_And_BSR_Election_Rollback
! =========================================================
! REMOVE RECEIVER JOIN
! =========================================================
conf t
 interface <RECEIVER_INTERFACE>
  no ip igmp join-group <MULTICAST_GROUP>
 exit
end
! =========================================================
! REMOVE PRIMARY CANDIDATE RP
! =========================================================
conf t
 no ip pim rp-candidate <PRIMARY_RP_SOURCE_INTERFACE> group-list <BSR_GROUP_ACL> interval <RP_ADV_INTERVAL> priority <PRIMARY_RP_PRIORITY>
end
! =========================================================
! REMOVE BACKUP CANDIDATE RP
! =========================================================
conf t
 no ip pim rp-candidate <BACKUP_RP_SOURCE_INTERFACE> group-list <BSR_GROUP_ACL> interval <RP_ADV_INTERVAL> priority <BACKUP_RP_PRIORITY>
end
! =========================================================
! REMOVE PRIMARY CANDIDATE BSR
! =========================================================
conf t
 no ip pim bsr-candidate <PRIMARY_BSR_SOURCE_INTERFACE> <HASH_MASK_LENGTH> <PRIMARY_BSR_PRIORITY>
end
! =========================================================
! REMOVE BACKUP CANDIDATE BSR
! =========================================================
conf t
 no ip pim bsr-candidate <BACKUP_BSR_SOURCE_INTERFACE> <HASH_MASK_LENGTH> <BACKUP_BSR_PRIORITY>
end
! =========================================================
! REMOVE BSR GROUP ACL
! =========================================================
conf t
 no access-list <BSR_GROUP_ACL>
end
! =========================================================
! REMOVE PIM FROM CANDIDATE RP LOOPBACK
! Only do this if no other multicast labs depend on it.
! =========================================================
conf t
 interface <CANDIDATE_RP_LOOPBACK>
  no ip pim sparse-mode
 exit
end
! =========================================================
! REMOVE PIM FROM CANDIDATE BSR LOOPBACK
! Only do this if no other multicast labs depend on it.
! =========================================================
conf t
 interface <CANDIDATE_BSR_LOOPBACK>
  no ip pim sparse-mode
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
! CLEAR MULTICAST STATE AND DEBUGGING
! =========================================================
clear ip mroute *
clear ip igmp group *
undebug all
# PIM_BSR_Candidate_RP_And_BSR_Election_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| No elected BSR appears | Candidate BSR missing, PIM not enabled, or PIM domain is broken | All multicast routers | `show ip pim bsr-router`<br>`show ip pim neighbor` | Configure `ip pim bsr-candidate`, enable PIM, and fix PIM neighbor reachability |
| Wrong router becomes BSR | Higher BSR priority or higher source IP exists on another Candidate BSR | Candidate BSR routers | `show ip pim bsr-router` | Set intended BSR with higher BSR priority |
| BSR election ties unexpectedly | Same BSR priority on multiple candidates | Candidate BSR routers | `show ip pim bsr-router` | Use explicit BSR priorities instead of relying on IP tiebreak |
| Candidate RP does not appear in RP mapping | Candidate RP advertisement missing or not reaching elected BSR | Candidate RP and BSR | `show running-config | include rp-candidate`<br>`debug ip pim bsr` | Configure Candidate RP correctly and verify route/PIM reachability to BSR |
| RP-set exists only on some routers | Bootstrap messages are not propagating through the PIM domain | Transit routers | `show ip pim neighbor`<br>`show ip pim bsr-router` | Fix PIM adjacency and multicast control-plane path |
| Wrong RP selected for group | Candidate RP priority, group-list, or hash selection does not match intended design | All multicast routers | `show ip pim rp mapping`<br>`show access-lists <BSR_GROUP_ACL>` | Correct RP priority, group-list ACL, or RP design |
| Backup RP wins over primary RP | Backup has lower RP priority value or group-list overlap is wrong | All multicast routers | `show ip pim rp mapping` | Give primary the lower RP priority value |
| Candidate RP command fails | Source interface missing PIM or unsupported command syntax on platform | Candidate RP | `show ip pim interface`<br>`ip pim rp-candidate ?` | Enable PIM on source interface or use platform-supported syntax |
| Candidate BSR command fails | Source interface missing PIM or unsupported command syntax on platform | Candidate BSR | `show ip pim interface`<br>`ip pim bsr-candidate ?` | Enable PIM on source interface or use platform-supported syntax |
| Group range missing from RP-set | Candidate RP group-list ACL does not match multicast group | Candidate RP | `show access-lists <BSR_GROUP_ACL>` | Correct ACL to match the multicast group range |
| Static RP overrides expected BSR behavior | Static RP remains configured | All multicast routers | `show running-config | include rp-address` | Remove static RP or keep BSR and static RP scopes clearly separated |
| Auto-RP and BSR conflict | Auto-RP is still configured in the same domain | All multicast routers | `show running-config | include send-rp`<br>`show ip pim rp mapping` | Remove Auto-RP unless the lab specifically tests interoperation |
| Receiver group missing | Receiver did not join or joined on wrong interface | Last-hop router and receiver | `show ip igmp groups`<br>`show running-config interface <RECEIVER_INTERFACE>` | Configure `ip igmp join-group <MULTICAST_GROUP>` on correct receiver interface |
| RP mapping exists but multicast traffic fails | IGMP, RPF, source traffic, or PIM forwarding path is broken | RP, source-side, and last-hop routers | `show ip igmp groups`<br>`show ip rpf <SOURCE_IP>`<br>`show ip mroute <MULTICAST_GROUP>` | Fix receiver join, RPF path, source traffic, or PIM forwarding |
| Incoming interface is wrong | RPF route to RP or source points to wrong interface | Transit and last-hop routers | `show ip rpf <SELECTED_RP_ADDRESS>`<br>`show ip rpf <SOURCE_IP>` | Correct unicast routing or use deliberate multicast RPF correction |
| BSR failover does not occur quickly | Bootstrap holdtime has not expired or backup cannot win election | All multicast routers | `show ip pim bsr-router` | Wait for expiration, clear state in lab, or correct backup BSR priority/reachability |
| Debug floods the console | BSR debug left enabled | Any router | `show debugging` | Run `undebug all` |
##### Source_Basis
# PIM_BSR_Candidate_RP_And_BSR_Election_Mental_Model
# PIM_BSR_Candidate_RP_And_BSR_Election_Configuration_Checklist
# PIM_BSR_Candidate_RP_And_BSR_Election_Skeleton
# PIM_BSR_Candidate_RP_And_BSR_Election_Verification_Commands
# PIM_BSR_Candidate_RP_And_BSR_Election_Rollback
# PIM_BSR_Candidate_RP_And_BSR_Election_Failure_Checks


The project source covers the DUAL and feasible-successor base, but the exact EIGRP LFA FRR syntax is pulled from Cisco IOS XE because the local EIGRP source does not fully spell out the FRR command set. Cisco documents EIGRP LFA FRR as named-mode, IPv4-only, point-to-point protected, configured under topology base with fast-reroute per-prefix {all | route-map <name>}, and verified with show ip eigrp topology frr.  

EIGRP_LFA_Fast_Reroute_Per_Prefix.md

EIGRP_LFA_Fast_Reroute_Per_Prefix

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 3: EIGRP for IP | Points EIGRP DUAL, feasible successors, topology table, convergence, and troubleshooting to `All_combined_part3.md` |
| `All_combined_part3.md` | Book 3, Chapter 1: EIGRP Concepts and Technology | Supports DUAL, successor, feasible successor, topology table, and fast convergence behavior |
| `All_combined_part3.md` | Book 3, Chapter 2: Advanced EIGRP Concepts, Data Structures and Algorithms | Supports DUAL passive/active behavior, feasible successor promotion, and query avoidance |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | EIGRP Path Metric Calculation and Topology Table | Supports feasible successors as loop-free backup paths selected by EIGRP |
| `Cisco IOS XE 17.x IP Routing Configuration Guide` | EIGRP Loop-Free Alternate Fast Reroute | Supports `fast-reroute per-prefix {all | route-map <name>}` under named-mode EIGRP `topology base` |
| `Cisco IOS XE 17.x IP Routing Configuration Guide` | Configuring LFA FRRs per Prefix | Supports verification with `show ip eigrp topology frr` |
| `Cisco IOS XE 17.x IP Routing Configuration Guide` | Restrictions for EIGRP Loop-Free Alternate Fast Reroute | Supports the caveats that IPv6 is not supported and only point-to-point reachable paths are protected |
# EIGRP_LFA_Fast_Reroute_Per_Prefix_Mental_Model
| Concept | Operational Meaning |
|---|---|
| EIGRP LFA FRR | Precomputes repair paths so traffic can move to a backup path quickly after a failure |
| Per-prefix FRR | EIGRP computes backup protection separately for each prefix instead of treating every prefix behind a link the same |
| Successor | Primary EIGRP path currently used for forwarding |
| Feasible successor | Loop-free backup path that can be used as a repair path |
| LFA | Loop-Free Alternate; in EIGRP this maps to a feasible successor usable as a backup |
| Repair path | Backup path installed for fast reroute after primary path failure |
| Protected prefix | Prefix for which EIGRP has calculated an FRR repair path |
| Unprotected prefix | Prefix with no eligible feasible successor or no supported repair path |
| Named-mode requirement | Cisco IOS XE EIGRP LFA FRR is configured under named EIGRP address-family `topology base` |
| Classic-mode limitation | Do not expect `router eigrp <AS>` classic mode to expose the FRR syntax |
| `fast-reroute per-prefix all` | Enables FRR calculation for all eligible prefixes |
| `fast-reroute per-prefix route-map <name>` | Enables FRR only for prefixes matched by the route-map |
| FRR verification | `show ip eigrp topology frr` is the main command for seeing configured LFAs |
| DUAL relationship | FRR does not replace DUAL; it uses DUAL’s feasible successor logic as the fast repair source |
| Operational warning | FRR cannot invent a backup path. The topology must already provide an eligible loop-free alternate |
| Platform caveat | Verify feature support on the actual IOS XE image because CML images may not support every hardware-platform command |
# EIGRP_LFA_Fast_Reroute_Per_Prefix_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm EIGRP-facing interfaces are operational | All routers | `show ip interface brief` | EIGRP-facing interfaces are `up/up` with correct IPv4 addresses |
| 2 | Confirm the topology has redundant paths before enabling FRR | All routers | `show ip route eigrp` | Target prefixes are reachable through EIGRP and the design has alternate paths |
| 3 | Confirm EIGRP neighbors are stable | All routers | `show ip eigrp neighbors` | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| 4 | Confirm the router is using named-mode EIGRP | FRR router | `show running-config | section router eigrp` | Output shows `router eigrp <PROCESS_NAME>` and an IPv4 address family |
| 5 | Confirm the real EIGRP AS number | FRR router | `show ip protocols` | EIGRP address-family AS number is confirmed |
| 6 | Confirm baseline topology entries before FRR | FRR router | `show ip eigrp topology` | Expected prefixes appear in passive state |
| 7 | Pick a target prefix for FRR validation | FRR router | `show ip route <PREFIX>` | Target route is installed through the expected EIGRP successor |
| 8 | Confirm whether a feasible successor already exists | FRR router | `show ip eigrp topology <PREFIX>/<LENGTH>` | Backup path appears if an eligible feasible successor exists |
| 9 | Show all EIGRP candidate paths if no backup appears in normal output | FRR router | `show ip eigrp topology all-links` | All known paths appear, including non-feasible paths |
| 10 | Enter global configuration mode | FRR router | `configure terminal` | Router enters global configuration mode |
| 11 | Enter the named EIGRP process | FRR router | `router eigrp <PROCESS_NAME>` | Router enters named EIGRP configuration mode |
| 12 | Enter the IPv4 EIGRP address family | FRR router | `address-family ipv4 autonomous-system <AS_NUMBER>` | Router enters address-family mode |
| 13 | Enter the base topology | FRR router | `topology base` | Router enters EIGRP topology configuration mode |
| 14 | Enable FRR for all eligible prefixes | FRR router | `fast-reroute per-prefix all` | EIGRP enables per-prefix FRR for all eligible topology entries |
| 15 | Exit configuration mode | FRR router | `end` | Router returns to privileged EXEC mode |
| 16 | Verify FRR topology output | FRR router | `show ip eigrp topology frr` | Protected prefixes and their repair paths appear |
| 17 | Verify the target prefix still has the correct primary route | FRR router | `show ip route <PREFIX>` | Primary route remains through the intended successor |
| 18 | Verify EIGRP topology remains passive | FRR router | `show ip eigrp topology <PREFIX>/<LENGTH>` | Prefix remains passive with successor and backup information |
| 19 | Verify no active routes exist after enabling FRR | FRR router | `show ip eigrp topology active` | No active prefixes are listed |
| 20 | Build a prefix-list if FRR should apply only to selected prefixes | FRR router | `ip prefix-list <FRR_PREFIX_LIST> seq 5 permit <PREFIX>/<LENGTH>` | Prefix-list matches only the protected prefix |
| 21 | Build a route-map for selective FRR | FRR router | `route-map <FRR_ROUTE_MAP> permit 10` then `match ip address prefix-list <FRR_PREFIX_LIST>` | Route-map matches the selected FRR prefixes |
| 22 | Verify selective FRR match logic | FRR router | `show ip prefix-list <FRR_PREFIX_LIST>` and `show route-map <FRR_ROUTE_MAP>` | Prefix-list and route-map match the intended prefixes |
| 23 | Replace all-prefix FRR with selective FRR if required | FRR router | `router eigrp <PROCESS_NAME>` then `address-family ipv4 autonomous-system <AS_NUMBER>` then `topology base` then `fast-reroute per-prefix route-map <FRR_ROUTE_MAP>` | FRR applies only to prefixes matched by the route-map |
| 24 | Verify selective FRR protection | FRR router | `show ip eigrp topology frr` | Only matched eligible prefixes show FRR repair paths |
| 25 | Confirm unsupported or nonmatched prefixes are not protected | FRR router | `show ip eigrp topology frr` | Nonmatched prefixes are absent from FRR output |
| 26 | Perform a controlled primary-link failure in the lab | FRR router | `interface <PRIMARY_PATH_INTERFACE>` then `shutdown` | Primary path fails intentionally |
| 27 | Verify traffic uses the repair path | FRR router or host | `ping <PROTECTED_PREFIX_IP>` | Ping continues or experiences minimal loss depending on platform and lab timing |
| 28 | Verify the new forwarding path | FRR router or host | `traceroute <PROTECTED_PREFIX_IP>` | Path moves through the backup next hop |
| 29 | Verify EIGRP does not get stuck active | FRR router | `show ip eigrp topology active` | No stuck active route remains |
| 30 | Restore the failed interface | FRR router | `interface <PRIMARY_PATH_INTERFACE>` then `no shutdown` | Interface returns to `up/up` |
| 31 | Verify neighbor recovery after restoration | FRR router | `show ip eigrp neighbors` | Neighbor returns with `Q Cnt` of `0` |
| 32 | Verify FRR protection still exists after restoration | FRR router | `show ip eigrp topology frr` | Protected prefixes still show repair paths |
| 33 | Verify final routing table state | FRR router | `show ip route <PREFIX>` | Prefix returns to expected primary successor |
| 34 | Save the configuration | Changed routers | `write memory` | Configuration is saved |
# EIGRP_LFA_Fast_Reroute_Per_Prefix_Skeleton
! Baseline verification
show ip interface brief
show ip eigrp neighbors
show ip protocols
show running-config | section router eigrp
show ip route eigrp
show ip route <PREFIX>
show ip eigrp topology
show ip eigrp topology <PREFIX>/<LENGTH>
show ip eigrp topology all-links
show ip eigrp topology active
! Enable EIGRP LFA FRR for all eligible prefixes
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 autonomous-system <AS_NUMBER>
   topology base
    fast-reroute per-prefix all
   exit-af-topology
  exit-address-family
 end
! Verify FRR
show ip eigrp topology frr
show ip route <PREFIX>
show ip eigrp topology <PREFIX>/<LENGTH>
show ip eigrp topology active
! Optional selective FRR match logic
configure terminal
 ip prefix-list <FRR_PREFIX_LIST> seq 5 permit <PREFIX>/<LENGTH>
 route-map <FRR_ROUTE_MAP> permit 10
  match ip address prefix-list <FRR_PREFIX_LIST>
end
! Apply selective FRR
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 autonomous-system <AS_NUMBER>
   topology base
    fast-reroute per-prefix route-map <FRR_ROUTE_MAP>
   exit-af-topology
  exit-address-family
 end
! Verify selective FRR
show ip prefix-list <FRR_PREFIX_LIST>
show route-map <FRR_ROUTE_MAP>
show ip eigrp topology frr
! Controlled lab failure
configure terminal
 interface <PRIMARY_PATH_INTERFACE>
  shutdown
 end
show ip route <PREFIX>
show ip eigrp topology active
ping <PROTECTED_PREFIX_IP>
traceroute <PROTECTED_PREFIX_IP>
! Restore primary path
configure terminal
 interface <PRIMARY_PATH_INTERFACE>
  no shutdown
 end
! Final verification
show ip interface brief
show ip eigrp neighbors
show ip eigrp topology frr
show ip route <PREFIX>
show ip eigrp topology active
ping <PROTECTED_PREFIX_IP>
traceroute <PROTECTED_PREFIX_IP>
! Save
write memory
# EIGRP_LFA_Fast_Reroute_Per_Prefix_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state before FRR testing | EIGRP-facing interfaces are `up/up` |
| `show ip eigrp neighbors` | Confirms EIGRP adjacency health | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| `show ip protocols` | Confirms named EIGRP process, AS, network statements, and K-values | Values match the intended EIGRP design |
| `show running-config | section router eigrp` | Confirms named-mode FRR placement | Shows `fast-reroute per-prefix all` or `fast-reroute per-prefix route-map <name>` under `topology base` |
| `show ip route eigrp` | Confirms EIGRP routes are installed | Internal routes appear as `D`; external routes appear as `D EX` |
| `show ip route <PREFIX>` | Confirms primary RIB winner | Prefix points to the expected successor before failure |
| `show ip eigrp topology <PREFIX>/<LENGTH>` | Confirms successor and possible feasible successor state | Prefix is passive and has expected primary and backup path data |
| `show ip eigrp topology all-links` | Shows all known EIGRP candidate paths | Backup candidates are visible for comparison |
| `show ip eigrp topology frr` | Confirms FRR repair-path calculation | Protected prefixes and LFAs appear |
| `show ip eigrp topology active` | Confirms DUAL stability | No active prefixes remain after convergence |
| `show ip prefix-list <FRR_PREFIX_LIST>` | Verifies selective FRR match object | Prefix-list matches only intended prefixes |
| `show route-map <FRR_ROUTE_MAP>` | Verifies selective FRR route-map | Route-map references the intended prefix-list |
| `show logging | include EIGRP|DUAL|FRR|NBRCHANGE` | Checks adjacency and convergence events | No recurring instability after FRR configuration |
| `ping <PROTECTED_PREFIX_IP>` | Confirms data-plane reachability | Ping succeeds before and after controlled failure |
| `traceroute <PROTECTED_PREFIX_IP>` | Confirms forwarding path | Path uses primary path before failure and backup path after failure |
# EIGRP_LFA_Fast_Reroute_Per_Prefix_Rollback
! Remove all-prefix FRR
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 autonomous-system <AS_NUMBER>
   topology base
    no fast-reroute per-prefix all
   exit-af-topology
  exit-address-family
 end
! Remove selective FRR
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 autonomous-system <AS_NUMBER>
   topology base
    no fast-reroute per-prefix route-map <FRR_ROUTE_MAP>
   exit-af-topology
  exit-address-family
 end
! Remove route-map if created only for FRR selection
configure terminal
 no route-map <FRR_ROUTE_MAP>
end
! Remove prefix-list if created only for FRR selection
configure terminal
 no ip prefix-list <FRR_PREFIX_LIST>
end
! Restore any interface shut down during testing
configure terminal
 interface <PRIMARY_PATH_INTERFACE>
  no shutdown
 end
! Verify rollback
show running-config | section router eigrp
show ip prefix-list <FRR_PREFIX_LIST>
show route-map <FRR_ROUTE_MAP>
show ip interface brief
show ip eigrp neighbors
show ip eigrp topology frr
show ip route <PREFIX>
show ip eigrp topology active
ping <PROTECTED_PREFIX_IP>
traceroute <PROTECTED_PREFIX_IP>
! Expected result:
! FRR configuration is removed.
! `show ip eigrp topology frr` no longer shows configured FRR protection.
! Normal EIGRP successor and feasible-successor behavior remains.
! Interfaces and neighbors are restored.
write memory
# EIGRP_LFA_Fast_Reroute_Per_Prefix_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| `fast-reroute` command is unavailable | Router is in classic EIGRP mode | `show running-config | section router eigrp` | Use named-mode EIGRP and configure FRR under address-family `topology base` |
| `fast-reroute` command is still unavailable | Platform or IOS XE image does not support EIGRP LFA FRR | `fast-reroute ?` under `topology base` | Use a supported IOS XE image or skip FRR in that CML node |
| `show ip eigrp topology frr` shows no protected prefixes | No eligible feasible successor or LFA exists | `show ip eigrp topology <PREFIX>/<LENGTH>` and `show ip eigrp topology all-links` | Add redundancy or adjust topology/metrics so a loop-free backup exists |
| FRR protects some prefixes but not others | Per-prefix eligibility differs | `show ip eigrp topology frr` | Validate each prefix separately; FRR is per-prefix, not guaranteed for every route |
| Selective FRR protects no prefixes | Route-map does not match the intended prefixes | `show route-map <FRR_ROUTE_MAP>` and `show ip prefix-list <FRR_PREFIX_LIST>` | Correct prefix-list and route-map match logic |
| All prefixes are protected when only selected prefixes were intended | `fast-reroute per-prefix all` still configured | `show running-config | section router eigrp` | Replace `all` with `route-map <FRR_ROUTE_MAP>` |
| Prefix-list matches nothing | Wrong prefix or length | `show ip route <PREFIX>` and `show ip prefix-list <FRR_PREFIX_LIST>` | Match the exact EIGRP prefix and prefix length |
| Route-map exists but has no effect | FRR still references a different route-map or `all` | `show running-config | section router eigrp` | Reference the correct route-map under `fast-reroute per-prefix route-map <name>` |
| FRR repair path is not used during failure | Failure did not affect the protected primary path or repair path was not installed | `show ip route <PREFIX>` before and after failure | Fail the actual primary-path interface and confirm FRR output before testing |
| Route goes active during failure | No usable LFA existed for that prefix | `show ip eigrp topology active` | Fix topology redundancy or accept normal DUAL recomputation |
| Traffic still drops too long during failover | CML timing, interface failure detection, or no data-plane repair path | `show logging | include EIGRP|DUAL|NBRCHANGE` | Verify actual feature support and test with point-to-point eligible paths |
| FRR output exists but ping fails | Return path or data-plane issue | Remote `show ip route <SOURCE_PREFIX>` | Fix reverse route, ACL, NAT, or interface issue |
| IPv6 prefix does not get FRR protection | EIGRP LFA FRR feature is IPv4-only in the documented IOS XE module | `show ipv6 route` and `show ip eigrp topology frr` | Use IPv4 for this mechanism lab |
| Multipoint path is not protected | Feature protects paths reachable through point-to-point interfaces | `show interfaces <INTERFACE>` and topology review | Test on point-to-point links |
| FRR works in documentation but not in CML | CML image lacks command support or hardware-specific feature behavior | `show version` and CLI help under `topology base` | Use a supported IOS XE image or document as unsupported in that lab |
| EIGRP neighbor flaps after config change | Underlying adjacency issue, not FRR itself | `show ip eigrp neighbors` and `show logging | include NBRCHANGE` | Fix link, authentication, K-values, timers, or passive-interface state |
| Backup path is suboptimal | Default LFA selection chose an available but not preferred repair path | `show ip eigrp topology frr` | Use the later tie-breaker mechanism note for interface-disjoint, SRLG-disjoint, or lowest-backup-path-metric behavior |
# Index
# Source_Basis
# EIGRP_LFA_Fast_Reroute_Per_Prefix_Mental_Model
# EIGRP_LFA_Fast_Reroute_Per_Prefix_Configuration_Checklist
# EIGRP_LFA_Fast_Reroute_Per_Prefix_Skeleton
# EIGRP_LFA_Fast_Reroute_Per_Prefix_Verification_Commands
# EIGRP_LFA_Fast_Reroute_Per_Prefix_Rollback
# EIGRP_LFA_Fast_Reroute_Per_Prefix_Failure_Checks

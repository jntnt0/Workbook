Cisco’s current IOS XE EIGRP FRR doc is the authority for this exact syntax: fast-reroute tie-break {interface-disjoint | linecard-disjoint | lowest-backup-path-metric | srlg-disjoint} <priority-number> under named-mode EIGRP topology base; lower priority numbers win, and show ip eigrp topology frr verifies selected LFAs. It also documents key limits: IPv6, MPLS primary/backup paths, ECMP primary/backup paths, and GRE primary paths are not supported for this EIGRP LFA FRR feature.  

EIGRP_LFA_Tie_Breaks_Interface_And_SRLG_Disjoint.md

EIGRP_LFA_Tie_Breaks_Interface_And_SRLG_Disjoint

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 3: EIGRP for IP | Points EIGRP DUAL, feasible successors, topology table, convergence, and troubleshooting to `All_combined_part3.md` |
| `All_combined_part3.md` | Book 3, Chapter 1: EIGRP Concepts and Technology | Supports successor, feasible successor, topology table, and DUAL convergence behavior |
| `All_combined_part3.md` | Book 3, Chapter 2: Advanced EIGRP Concepts, Data Structures and Algorithms | Supports DUAL route recomputation, passive/active route behavior, and feasible successor promotion |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | EIGRP Topology Table and Feasible Successors | Supports the base logic that EIGRP backup paths must be loop-free feasible successors |
| `Cisco IOS XE IP Routing Configuration Guide` | EIGRP Loop-Free Alternate IP Fast Reroute | Supports `fast-reroute per-prefix`, `fast-reroute tie-break`, `fast-reroute load-sharing disable`, and `show ip eigrp topology frr` |
| `Cisco IOS XE IP Routing Configuration Guide` | LFA Tie-Breaking Rules | Supports tie-break attributes: `interface-disjoint`, `linecard-disjoint`, `lowest-backup-path-metric`, and `srlg-disjoint` |
| `Cisco IOS XE IP Routing Configuration Guide` | Restrictions for EIGRP LFA FRR | Supports caveats around IPv6, MPLS paths, ECMP paths, GRE primary paths, licensing, and platform support |
# EIGRP_LFA_Tie_Breaks_Interface_And_SRLG_Disjoint_Mental_Model
| Concept | Operational Meaning |
|---|---|
| EIGRP LFA FRR | Precomputes a backup path so traffic can move around a failed primary path quickly |
| Feasible successor | The loop-free backup route EIGRP can use as an LFA |
| Tie-break rule | Policy used when more than one LFA is available for the same prefix and primary path |
| Priority number | Lower number means higher priority in the LFA tie-break order |
| Interface-disjoint | Prefers LFAs that do not share the outgoing interface used by the protected primary path |
| SRLG-disjoint | Prefers LFAs that do not share the same Shared Risk Link Group as the protected primary path |
| Linecard-disjoint | Prefers LFAs that do not share the same linecard as the protected primary path |
| Lowest-backup-path-metric | Prefers the LFA with the lowest backup metric after higher-priority disjointness rules are evaluated |
| SRLG | Shared Risk Link Group, usually representing links that share a physical failure risk such as fiber, conduit, carrier path, or hardware dependency |
| Protection quality | A backup path is not automatically good just because it is loop-free; it may still share the same interface, linecard, or physical risk |
| Tie-break ordering | Put the most important failure-domain avoidance rule at the lowest priority number |
| Common sane order | `srlg-disjoint 1`, `interface-disjoint 2`, `linecard-disjoint 3`, `lowest-backup-path-metric 4` |
| Load sharing among LFAs | Default behavior may distribute protected prefixes among multiple LFAs |
| Load-sharing disable | Forces deterministic LFA selection instead of spreading prefixes among multiple backups |
| Main verification command | `show ip eigrp topology frr` shows configured feasible successors or LFAs |
| Hard truth | Tie-breakers cannot create a backup path; they only choose among LFAs that already exist |
# EIGRP_LFA_Tie_Breaks_Interface_And_SRLG_Disjoint_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm EIGRP-facing interfaces are operational | FRR router | `show ip interface brief` | EIGRP-facing interfaces are `up/up` |
| 2 | Confirm EIGRP neighbors are stable | FRR router | `show ip eigrp neighbors` | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| 3 | Confirm named-mode EIGRP is in use | FRR router | `show running-config | section router eigrp` | Output shows `router eigrp <PROCESS_NAME>` and an IPv4 address family |
| 4 | Confirm the real EIGRP AS number | FRR router | `show ip protocols` | Address-family AS number is confirmed |
| 5 | Confirm target prefixes are learned through EIGRP | FRR router | `show ip route eigrp` | Target prefixes appear as `D` or `D EX` |
| 6 | Confirm the current primary path for a target prefix | FRR router | `show ip route <PREFIX>` | Route points to the expected successor next hop and outgoing interface |
| 7 | Confirm EIGRP has candidate backup paths | FRR router | `show ip eigrp topology <PREFIX>/<LENGTH>` | Prefix shows successor and possible feasible successor paths |
| 8 | Show all EIGRP candidate paths if backup paths are not visible | FRR router | `show ip eigrp topology all-links` | All known paths appear, including paths that failed feasibility condition |
| 9 | Confirm per-prefix FRR is already enabled or prepare to enable it | FRR router | `show running-config | section router eigrp` | `fast-reroute per-prefix all` or `fast-reroute per-prefix route-map <NAME>` is present if already enabled |
| 10 | Enter global configuration mode | FRR router | `configure terminal` | Router enters global configuration mode |
| 11 | Enter named EIGRP process | FRR router | `router eigrp <PROCESS_NAME>` | Router enters EIGRP named-mode configuration |
| 12 | Enter IPv4 address family | FRR router | `address-family ipv4 autonomous-system <AS_NUMBER>` | Router enters address-family mode |
| 13 | Enter base topology mode | FRR router | `topology base` | Router enters EIGRP topology configuration mode |
| 14 | Enable per-prefix FRR for all eligible prefixes if not already configured | FRR router | `fast-reroute per-prefix all` | EIGRP computes LFAs for all eligible prefixes |
| 15 | Optionally disable FRR load sharing for deterministic LFA selection | FRR router | `fast-reroute load-sharing disable` | EIGRP stops distributing protected prefixes among multiple LFAs |
| 16 | Configure SRLG-disjoint as highest priority if physical fate-sharing avoidance matters most | FRR router | `fast-reroute tie-break srlg-disjoint 1` | LFAs sharing SRLG risk with the primary path are avoided first |
| 17 | Configure interface-disjoint as next priority | FRR router | `fast-reroute tie-break interface-disjoint 2` | LFAs sharing the primary outgoing interface are avoided next |
| 18 | Configure linecard-disjoint if the platform has meaningful linecard failure domains | FRR router | `fast-reroute tie-break linecard-disjoint 3` | LFAs sharing the primary linecard are avoided when possible |
| 19 | Configure lowest backup metric as final tie-breaker | FRR router | `fast-reroute tie-break lowest-backup-path-metric 4` | If disjointness rules do not decide, lowest backup metric wins |
| 20 | Exit configuration mode | FRR router | `end` | Router returns to privileged EXEC mode |
| 21 | Verify FRR tie-break configuration | FRR router | `show running-config | section router eigrp` | `fast-reroute tie-break` lines appear under address-family `topology base` |
| 22 | Verify selected LFAs | FRR router | `show ip eigrp topology frr` | Protected prefixes show configured LFAs and repair paths |
| 23 | Verify the target prefix primary route remains correct | FRR router | `show ip route <PREFIX>` | Primary route still points to the intended successor |
| 24 | Verify EIGRP topology for the protected prefix | FRR router | `show ip eigrp topology <PREFIX>/<LENGTH>` | Prefix remains passive with valid successor and backup information |
| 25 | Verify no active routes remain after FRR changes | FRR router | `show ip eigrp topology active` | No active prefixes remain |
| 26 | Check logs for EIGRP instability | FRR router | `show logging | include EIGRP|DUAL|FRR|NBRCHANGE` | No repeated neighbor resets or DUAL instability |
| 27 | Perform controlled primary-link failure in a lab | FRR router | `interface <PRIMARY_PATH_INTERFACE>` then `shutdown` | Primary path fails intentionally |
| 28 | Verify traffic moves to the selected LFA | FRR router or host | `traceroute <PROTECTED_PREFIX_IP>` | Path moves through the expected backup next hop |
| 29 | Verify reachability during or after failover | FRR router or host | `ping <PROTECTED_PREFIX_IP>` | Ping succeeds or shows minimal loss depending on platform and lab timing |
| 30 | Verify route does not get stuck active after failure | FRR router | `show ip eigrp topology active` | No stuck active route remains |
| 31 | Restore the primary interface | FRR router | `interface <PRIMARY_PATH_INTERFACE>` then `no shutdown` | Interface returns to `up/up` |
| 32 | Verify neighbor recovery after restoration | FRR router | `show ip eigrp neighbors` | Neighbor returns with stable uptime and `Q Cnt` of `0` |
| 33 | Verify FRR protection still exists after restoration | FRR router | `show ip eigrp topology frr` | Protected prefixes still show LFA repair paths |
| 34 | Verify final primary route state | FRR router | `show ip route <PREFIX>` | Prefix returns to expected primary successor |
| 35 | Save the configuration | FRR router | `write memory` | Configuration is saved |
# EIGRP_LFA_Tie_Breaks_Interface_And_SRLG_Disjoint_Skeleton
! Baseline checks
show ip interface brief
show ip eigrp neighbors
show ip protocols
show running-config | section router eigrp
show ip route eigrp
show ip route <PREFIX>
show ip eigrp topology <PREFIX>/<LENGTH>
show ip eigrp topology all-links
show ip eigrp topology frr
show ip eigrp topology active
! Configure EIGRP LFA FRR tie-breaks
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 autonomous-system <AS_NUMBER>
   topology base
    fast-reroute per-prefix all
    fast-reroute load-sharing disable
    fast-reroute tie-break srlg-disjoint 1
    fast-reroute tie-break interface-disjoint 2
    fast-reroute tie-break linecard-disjoint 3
    fast-reroute tie-break lowest-backup-path-metric 4
   exit-af-topology
  exit-address-family
 end
! Verification
show running-config | section router eigrp
show ip eigrp topology frr
show ip eigrp topology <PREFIX>/<LENGTH>
show ip route <PREFIX>
show ip eigrp topology active
show logging | include EIGRP|DUAL|FRR|NBRCHANGE
! Controlled lab failure
configure terminal
 interface <PRIMARY_PATH_INTERFACE>
  shutdown
 end
show ip route <PREFIX>
show ip eigrp topology frr
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
show ip route <PREFIX>
show ip eigrp topology frr
show ip eigrp topology active
ping <PROTECTED_PREFIX_IP>
traceroute <PROTECTED_PREFIX_IP>
! Save
write memory
# EIGRP_LFA_Tie_Breaks_Interface_And_SRLG_Disjoint_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state before FRR testing | EIGRP-facing interfaces are `up/up` |
| `show ip eigrp neighbors` | Confirms EIGRP adjacency health | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| `show ip protocols` | Confirms EIGRP process and AS details | Named EIGRP address-family and AS match the design |
| `show running-config | section router eigrp` | Confirms FRR tie-break placement | Shows `fast-reroute` commands under address-family `topology base` |
| `show ip route <PREFIX>` | Confirms primary RIB winner | Prefix points to the expected successor before failure |
| `show ip route eigrp` | Confirms EIGRP-installed routes | Expected routes appear as `D` or `D EX` |
| `show ip eigrp topology <PREFIX>/<LENGTH>` | Confirms successor and feasible successor state | Prefix is passive and has valid path information |
| `show ip eigrp topology all-links` | Displays all known EIGRP paths | Shows candidate paths, including non-feasible paths |
| `show ip eigrp topology frr` | Confirms FRR repair-path calculation | Protected prefixes and selected LFAs appear |
| `show ip eigrp topology active` | Confirms DUAL stability | No active routes remain |
| `show logging | include EIGRP|DUAL|FRR|NBRCHANGE` | Checks convergence and neighbor events | No recurring instability after FRR configuration |
| `ping <PROTECTED_PREFIX_IP>` | Confirms data-plane reachability | Ping succeeds before and after controlled failure |
| `traceroute <PROTECTED_PREFIX_IP>` | Confirms repair-path forwarding | Path uses the expected backup after primary failure |
# EIGRP_LFA_Tie_Breaks_Interface_And_SRLG_Disjoint_Rollback
! Remove FRR tie-break rules
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 autonomous-system <AS_NUMBER>
   topology base
    no fast-reroute tie-break srlg-disjoint
    no fast-reroute tie-break interface-disjoint
    no fast-reroute tie-break linecard-disjoint
    no fast-reroute tie-break lowest-backup-path-metric
   exit-af-topology
  exit-address-family
 end
! Re-enable FRR load sharing if it was disabled only for this lab
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 autonomous-system <AS_NUMBER>
   topology base
    no fast-reroute load-sharing disable
   exit-af-topology
  exit-address-family
 end
! Remove all-prefix FRR if this whole FRR test is being backed out
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 autonomous-system <AS_NUMBER>
   topology base
    no fast-reroute per-prefix all
   exit-af-topology
  exit-address-family
 end
! Restore any interface shut down during failure testing
configure terminal
 interface <PRIMARY_PATH_INTERFACE>
  no shutdown
 end
! Verify rollback
show running-config | section router eigrp
show ip interface brief
show ip eigrp neighbors
show ip eigrp topology frr
show ip route <PREFIX>
show ip eigrp topology active
ping <PROTECTED_PREFIX_IP>
traceroute <PROTECTED_PREFIX_IP>
! Expected result:
! Tie-break rules are removed.
! FRR behavior returns to default LFA selection if per-prefix FRR remains enabled.
! If per-prefix FRR was removed too, normal EIGRP successor and feasible-successor behavior remains.
write memory
# EIGRP_LFA_Tie_Breaks_Interface_And_SRLG_Disjoint_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| `fast-reroute tie-break` command is unavailable | Router is in classic EIGRP mode | `show running-config | section router eigrp` | Use named-mode EIGRP and configure under address-family `topology base` |
| `fast-reroute tie-break` is still unavailable | Platform, license, or image does not support the feature | `fast-reroute ?` under `topology base` | Use a supported IOS XE image and license level |
| Tie-break rule fails to configure | Same attribute already configured | `show running-config | section router eigrp` | Remove the existing tie-break for that attribute before re-adding it |
| Tie-break priorities do not behave as expected | Priority order misunderstood | `show running-config | section router eigrp` | Use lower priority number for higher preference |
| No prefixes appear in FRR output | Per-prefix FRR is not enabled or no eligible LFAs exist | `show ip eigrp topology frr` and `show ip eigrp topology all-links` | Enable `fast-reroute per-prefix all` and confirm feasible successors exist |
| Interface-disjoint does not change selected LFA | No interface-disjoint LFA exists | `show ip eigrp topology frr` | Add real path diversity or accept the available LFA |
| SRLG-disjoint does not change selected LFA | No SRLG-disjoint LFA exists or SRLG metadata is not meaningful on the platform | `show ip eigrp topology frr` | Confirm platform support and verify the topology actually has SRLG-diverse paths |
| Linecard-disjoint does not change selected LFA | Platform has no meaningful linecard diversity in the lab | `show ip eigrp topology frr` | Use this tie-break only on platforms where linecard failure domains matter |
| Lowest metric backup wins over disjointness unexpectedly | Lowest metric has higher priority than disjointness | `show running-config | section router eigrp` | Put `lowest-backup-path-metric` at a higher number than disjointness rules |
| FRR protects some prefixes but not others | FRR is per-prefix and eligibility differs by prefix | `show ip eigrp topology frr` | Validate each prefix individually |
| Traffic still drops after primary failure | No usable LFA, unsupported path type, or slow link detection | `show ip eigrp topology frr` and `show logging | include EIGRP|DUAL|NBRCHANGE` | Confirm FRR support, primary link type, and LFA eligibility |
| FRR does not support the tested path | Primary or backup path uses MPLS, ECMP, GRE primary path, or unsupported design | `show ip route <PREFIX>` and topology review | Test with supported IPv4 point-to-point routed paths |
| Route goes active during failure | No eligible repair path was available | `show ip eigrp topology active` | Add redundancy or fix feasibility condition |
| Backup path is selected but traffic fails | Return path or data-plane issue | Remote `show ip route <SOURCE_PREFIX>` | Fix reverse routing, ACL, NAT, or interface issue |
| Load sharing still spreads prefixes across LFAs | Load sharing was not disabled | `show running-config | section router eigrp` | Configure `fast-reroute load-sharing disable` under `topology base` |
| All prefixes use one backup when load sharing was expected | Load sharing is disabled | `show running-config | section router eigrp` | Remove `fast-reroute load-sharing disable` |
| Neighbor flaps after configuration | Underlying adjacency problem, not tie-break logic | `show ip eigrp neighbors` and `show logging | include NBRCHANGE` | Fix interface state, timers, authentication, K-values, or packet loss |
| CML image accepts base EIGRP but rejects FRR syntax | Feature gap in virtual image | `show version` and CLI help | Document unsupported feature or use a different IOS XE image |
# Index
# Source_Basis
# EIGRP_LFA_Tie_Breaks_Interface_And_SRLG_Disjoint_Mental_Model
# EIGRP_LFA_Tie_Breaks_Interface_And_SRLG_Disjoint_Configuration_Checklist
# EIGRP_LFA_Tie_Breaks_Interface_And_SRLG_Disjoint_Skeleton
# EIGRP_LFA_Tie_Breaks_Interface_And_SRLG_Disjoint_Verification_Commands
# EIGRP_LFA_Tie_Breaks_Interface_And_SRLG_Disjoint_Rollback
# EIGRP_LFA_Tie_Breaks_Interface_And_SRLG_Disjoint_Failure_Checks
OSPF_External_LSAs_Forwarding_Address_And_Type5_Filtering.md

OSPF_External_LSAs_Forwarding_Address_And_Type5_Filtering

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | OSPF topic map | Points enterprise OSPF to `All_combined_part3.md` Book 6 and identifies OSPF as a main project source topic |
| `All_combined_part3.md` | Book 6, Chapter 8, external route filtering tasks | Supports `redistribute connected subnets route-map <name>`, `distribute-list prefix <name> out`, and `summary-address <prefix> <mask> not-advertise` for external route filtering |
| `All_combined_part3.md` | Book 6, Chapter 8, Type 5 filtering tasks | Confirms that OSPF `distribute-list out` only works for external routes when configured on the ASBR or originating router |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 7, LSA Type 5 External LSA | Supports Type 5 LSA behavior, ASBR role, external metric type, external route tag, and `show ip ospf database external` verification |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 7, External Summarization | Supports `summary-address <network> <mask>` under OSPF on the ASBR |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 16, OSPF Forwarding Address | Supports forwarding address behavior, default `0.0.0.0`, nondefault forwarding address conditions, and failure behavior when the forwarding address is unreachable |
| Related labs | `ospf-forward-address-final`, `ospf-forward-address-filtering-final`, `ospf-lsa-type-5-filtering-final` | Main labs for external LSA origination, forwarding address behavior, and Type 5 filtering |
# OSPF_External_LSAs_Forwarding_Address_And_Type5_Filtering_Mental_Model
| Concept | Operational Meaning |
|---|---|
| ASBR | The Autonomous System Boundary Router injects non-OSPF routes into OSPF through redistribution |
| Type 5 LSA | External LSA used to carry redistributed routes across normal OSPF areas |
| External flooding scope | Type 5 LSAs are flooded through the OSPF domain except into stub-style areas that block externals |
| Type 4 LSA | ABRs generate Type 4 ASBR summary LSAs so other areas know how to reach the ASBR that originated the Type 5 LSA |
| `O E1` route | External Type 1 route adds internal cost to reach the ASBR plus the external metric |
| `O E2` route | External Type 2 route uses the external metric first and is the default external type |
| External route tag | 32-bit tag carried in external LSAs, commonly used for redistribution loop prevention and policy marking |
| Forwarding address `0.0.0.0` | Routers forward traffic toward the ASBR |
| Nonzero forwarding address | Routers forward traffic toward the listed forwarding address instead of the ASBR |
| Forwarding address condition | Nonzero forwarding address appears when the external next-hop is reachable through an OSPF-enabled, non-passive, broadcast or nonbroadcast interface |
| Forwarding address reachability | A nonzero forwarding address must be reachable through an intra-area or interarea OSPF route, or the external LSA is ignored for RIB installation |
| Redistribution route-map | Best place to control which routes become external LSAs in the first place |
| `distribute-list out` in OSPF | Special case. It works only for external routes and must be configured on the ASBR or the router originating the external route |
| `summary-address not-advertise` | ASBR-side method to suppress matching external LSAs from the OSPF LSDB |
| Local `distribute-list in` | Filters route installation into the local RIB only. It does not remove the LSA from the LSDB |
| Stub or NSSA area interaction | Stub areas block Type 5 LSAs. NSSA uses Type 7 LSAs internally and translates to Type 5 outside the NSSA |
# OSPF_External_LSAs_Forwarding_Address_And_Type5_Filtering_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the ASBR | Design step | `ASBR: <router-name>` | Correct router is selected for redistribution and Type 5 filtering |
| 2 | Identify external prefixes to inject | Design step | `<external-prefix-list>` | Only intended non-OSPF routes are planned for redistribution |
| 3 | Identify external prefixes to block | Design step | `<blocked-prefix-list>` | Prefixes that must not become Type 5 LSAs are known |
| 4 | Confirm the target area allows Type 5 LSAs | ASBR and area routers | `show ip ospf` | Area is normal, not stub or totally stub |
| 5 | Confirm ASBR has the external source routes before redistribution | ASBR | `show ip route <external-prefix>` | External source route exists in the ASBR RIB |
| 6 | Confirm OSPF adjacencies are healthy before redistribution | ASBR | `show ip ospf neighbor` | Required neighbors are `FULL` |
| 7 | Confirm OSPF interface and area placement | ASBR | `show ip ospf interface brief` | ASBR has working OSPF interfaces in the intended area |
| 8 | Confirm current external LSDB state | ASBR and downstream routers | `show ip ospf database external` | Existing Type 5 LSAs are known before changes |
| 9 | Enter global configuration mode | ASBR | `configure terminal` | Router enters global configuration mode |
| 10 | Create prefix-list matching blocked external route | ASBR | `ip prefix-list OSPF_EXT_BLOCK seq 5 permit <blocked-prefix>/<length>` | Blocked prefix is matched for redistribution policy |
| 11 | Create route-map deny sequence for blocked prefix | ASBR | `route-map REDIST_TO_OSPF deny 10` then `match ip address prefix-list OSPF_EXT_BLOCK` | Blocked prefix is denied before entering OSPF |
| 12 | Create route-map permit sequence for all other redistributed routes | ASBR | `route-map REDIST_TO_OSPF permit 100` | Other source routes are allowed |
| 13 | Enter OSPF process | ASBR | `router ospf <process-id>` | OSPF router configuration mode opens |
| 14 | Redistribute connected routes with policy if connected routes are the source | ASBR | `redistribute connected subnets route-map REDIST_TO_OSPF` | Allowed connected routes become Type 5 LSAs |
| 15 | Redistribute static routes with policy if static routes are the source | ASBR | `redistribute static subnets route-map REDIST_TO_OSPF` | Allowed static routes become Type 5 LSAs |
| 16 | Redistribute another protocol with policy if required | ASBR | `redistribute <protocol> <id> subnets route-map REDIST_TO_OSPF` | Allowed routes from the source protocol become Type 5 LSAs |
| 17 | Set external metric type if required | ASBR | `redistribute <source> subnets metric-type 1 route-map REDIST_TO_OSPF` | External routes appear as `O E1` |
| 18 | Set external metric if required | ASBR | `redistribute <source> subnets metric <metric> route-map REDIST_TO_OSPF` | External LSA metric is deterministic |
| 19 | Set route tag if required by redistribution policy | ASBR | `redistribute <source> subnets tag <tag-value> route-map REDIST_TO_OSPF` | External route tag appears in the Type 5 LSA |
| 20 | Exit configuration mode | ASBR | `end` | Router returns to privileged EXEC mode |
| 21 | Save configuration | ASBR | `write memory` | Configuration is saved |
| 22 | Verify redistribution configuration | ASBR | `show running-config | section router ospf` | Redistribution command references the expected route-map |
| 23 | Verify route-map contents | ASBR | `show route-map REDIST_TO_OSPF` | Deny and permit sequences are present |
| 24 | Verify prefix-list contents | ASBR | `show ip prefix-list OSPF_EXT_BLOCK` | Blocked external prefix is matched |
| 25 | Verify allowed external LSAs are originated | ASBR | `show ip ospf database external` | Allowed external prefixes appear as Type 5 LSAs |
| 26 | Verify blocked prefix is not originated as Type 5 | ASBR and downstream routers | `show ip ospf database external <blocked-prefix>` | Blocked external prefix is absent from the external LSDB |
| 27 | Verify allowed external routes install downstream | Downstream routers | `show ip route <allowed-external-prefix>` | Route appears as `O E1` or `O E2` |
| 28 | Verify blocked external route is absent downstream | Downstream routers | `show ip route <blocked-prefix>` | Blocked route is not in the routing table |
| 29 | Verify Type 4 ASBR summary if ASBR is in another area | Downstream routers in other areas | `show ip ospf database asbr-summary` | ABR advertises reachability to the ASBR |
| 30 | Check forwarding address field | Downstream routers | `show ip ospf database external <external-prefix>` | Forward Address is visible as `0.0.0.0` or a reachable nonzero address |
| 31 | Confirm forwarding address reachability when nonzero | Downstream routers | `show ip route <forwarding-address>` | Forwarding address resolves through OSPF |
| 32 | Confirm data path to external destination | Downstream routers | `traceroute <external-destination>` | Traffic follows expected ASBR or forwarding address path |
| 33 | Use OSPF external `distribute-list out` only if the lab specifically requires it | ASBR only | `router ospf <process-id>` then `distribute-list prefix <PREFIX_LIST_NAME> out` | Matching external LSAs are filtered at the ASBR |
| 34 | Use `summary-address not-advertise` to suppress an external LSA from the LSDB | ASBR only | `summary-address <external-prefix> <mask> not-advertise` | Matching external Type 5 LSA is not advertised |
| 35 | Reverify LSDB after Type 5 suppression | ASBR and downstream routers | `show ip ospf database external <external-prefix>` | Suppressed prefix is absent from the external LSDB |
| 36 | Reverify downstream routing table after Type 5 suppression | Downstream routers | `show ip route <external-prefix>` | Suppressed external route is absent |
# OSPF_External_LSAs_Forwarding_Address_And_Type5_Filtering_Skeleton
! =========================================================
! ASBR redistribution with route-map filtering
! Best default method: stop unwanted prefixes before they become Type 5 LSAs.
! =========================================================
configure terminal
ip prefix-list OSPF_EXT_BLOCK seq 5 permit <BLOCKED_PREFIX>/<PREFIX_LENGTH>
route-map REDIST_TO_OSPF deny 10
 match ip address prefix-list OSPF_EXT_BLOCK
exit
route-map REDIST_TO_OSPF permit 100
exit
router ospf <PROCESS_ID>
 redistribute connected subnets route-map REDIST_TO_OSPF
end
write memory
! =========================================================
! Static route redistribution with route-map filtering
! =========================================================
configure terminal
ip prefix-list OSPF_EXT_BLOCK seq 5 permit <BLOCKED_PREFIX>/<PREFIX_LENGTH>
route-map REDIST_TO_OSPF deny 10
 match ip address prefix-list OSPF_EXT_BLOCK
exit
route-map REDIST_TO_OSPF permit 100
exit
router ospf <PROCESS_ID>
 redistribute static subnets route-map REDIST_TO_OSPF
end
write memory
! =========================================================
! Redistribute with external metric type, metric, and tag
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 redistribute <SOURCE_PROTOCOL> <SOURCE_ID> subnets metric-type 1 metric <METRIC> tag <TAG_VALUE> route-map REDIST_TO_OSPF
end
write memory
! =========================================================
! OSPF external distribute-list out
! Special OSPF case: works for external routes only, and only on the ASBR
! or the router that originated the external route.
! =========================================================
configure terminal
ip prefix-list EXT_OUT_FILTER seq 5 deny <BLOCKED_PREFIX>/<PREFIX_LENGTH>
ip prefix-list EXT_OUT_FILTER seq 100 permit 0.0.0.0/0 le 32
router ospf <PROCESS_ID>
 distribute-list prefix EXT_OUT_FILTER out
end
write memory
! =========================================================
! Suppress a Type 5 external LSA from the LSDB using not-advertise
! Configure on the ASBR.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 summary-address <BLOCKED_PREFIX> <BLOCKED_MASK> not-advertise
end
write memory
! =========================================================
! External summarization on ASBR
! This advertises one external summary instead of many specifics.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 summary-address <SUMMARY_PREFIX> <SUMMARY_MASK>
end
write memory
! =========================================================
! Forwarding address trigger pattern
! Nonzero forwarding address requires:
! 1. OSPF enabled on the ASBR interface toward the external next hop
! 2. Interface is not passive
! 3. Interface uses broadcast or nonbroadcast OSPF network type
! =========================================================
configure terminal
interface <ASBR_INTERFACE_TO_EXTERNAL_NEXT_HOP>
 ip ospf <PROCESS_ID> area <AREA_ID>
 ip ospf network broadcast
 no shutdown
exit
router ospf <PROCESS_ID>
 no passive-interface <ASBR_INTERFACE_TO_EXTERNAL_NEXT_HOP>
end
write memory
# OSPF_External_LSAs_Forwarding_Address_And_Type5_Filtering_Verification_Commands
show ip route
show ip route <external-prefix>
show ip route <forwarding-address>
show running-config | section router ospf
show running-config | include ^ip prefix-list
show ip prefix-list
show ip prefix-list <PREFIX_LIST_NAME>
show route-map
show route-map <ROUTE_MAP_NAME>
show ip protocols
show ip ospf
show ip ospf interface brief
show ip ospf interface <interface-id>
show ip ospf neighbor
show ip ospf database
show ip ospf database external
show ip ospf database external <external-prefix>
show ip ospf database asbr-summary
show ip route ospf
show ip route ospf | include E1|E2|N1|N2
ping <external-destination>
traceroute <external-destination>
# OSPF_External_LSAs_Forwarding_Address_And_Type5_Filtering_Rollback
! =========================================================
! Remove redistribution from OSPF
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no redistribute connected subnets route-map REDIST_TO_OSPF
 no redistribute static subnets route-map REDIST_TO_OSPF
 no redistribute <SOURCE_PROTOCOL> <SOURCE_ID> subnets route-map REDIST_TO_OSPF
end
write memory
! =========================================================
! Remove route-map and prefix-list after detaching redistribution
! =========================================================
configure terminal
no route-map REDIST_TO_OSPF
no ip prefix-list OSPF_EXT_BLOCK
end
write memory
! =========================================================
! Remove OSPF external distribute-list out
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no distribute-list prefix <PREFIX_LIST_NAME> out
end
write memory
! =========================================================
! Remove prefix-list used by external distribute-list
! =========================================================
configure terminal
no ip prefix-list <PREFIX_LIST_NAME>
end
write memory
! =========================================================
! Remove Type 5 suppression with not-advertise
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no summary-address <BLOCKED_PREFIX> <BLOCKED_MASK> not-advertise
end
write memory
! =========================================================
! Remove external summarization
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no summary-address <SUMMARY_PREFIX> <SUMMARY_MASK>
end
write memory
! =========================================================
! Remove OSPF from forwarding-address trigger interface
! =========================================================
configure terminal
interface <ASBR_INTERFACE_TO_EXTERNAL_NEXT_HOP>
 no ip ospf <PROCESS_ID> area <AREA_ID>
 no ip ospf network broadcast
exit
end
write memory
! =========================================================
! Lab-only OSPF reset if external LSDB output remains stale
! This disrupts adjacencies.
! =========================================================
clear ip ospf process
# OSPF_External_LSAs_Forwarding_Address_And_Type5_Filtering_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| External route never appears as Type 5 | ASBR does not have the source route | `show ip route <external-prefix>` on ASBR | Restore the connected, static, BGP, EIGRP, or source route before redistribution |
| External route never appears as Type 5 | Redistribution missing or wrong source protocol | `show running-config | section router ospf` | Configure the correct `redistribute <source> subnets` command |
| Only classful networks are redistributed | `subnets` keyword missing | `show running-config | section router ospf` | Add `subnets` to the redistribution command |
| Blocked prefix still appears as Type 5 | Route-map or prefix-list logic is wrong | `show route-map`, `show ip prefix-list` | Use a deny sequence matching the blocked prefix and a later permit sequence for the rest |
| Blocked prefix still appears as Type 5 | Filter was configured on the wrong router | `show ip ospf database external <prefix>` and check Advertising Router | Move filtering to the ASBR or router originating the external LSA |
| `distribute-list out` has no effect | Configured on non-ASBR | `show ip protocols`, `show ip ospf database external <prefix>` | Configure it on the ASBR that originates the external route |
| Route missing from local RIB but still present in LSDB | `distribute-list in` was used | `show ip ospf database external <prefix>`, `show ip route <prefix>` | Use ASBR-side redistribution policy, `distribute-list out`, or `summary-address not-advertise` if the LSA must be removed |
| Type 5 LSA is absent from stub area | Stub area blocks Type 5 LSAs by design | `show ip ospf`, `show ip ospf database external` | Use a normal area, NSSA design, or rely on default routing inside the stub area |
| NSSA route appears as Type 7 instead of Type 5 | Redistribution is occurring inside NSSA | `show ip ospf database nssa-external` | This is expected inside NSSA. Check outside NSSA for translated Type 5 |
| Type 5 exists but route is not installed | Nonzero forwarding address is unreachable | `show ip ospf database external <prefix>`, `show ip route <forwarding-address>` | Restore OSPF reachability to the forwarding address |
| Type 5 exists but traffic takes suboptimal path | Forwarding address is `0.0.0.0`, so traffic goes to ASBR | `show ip ospf database external <prefix>`, `traceroute <destination>` | Enable OSPF on the ASBR interface toward the external next hop if the design requires forwarding-address optimization |
| Forwarding address did not become nonzero | ASBR next-hop interface is passive | `show ip protocols`, `show ip ospf interface <interface-id>` | Configure `no passive-interface <interface-id>` under OSPF |
| Forwarding address did not become nonzero | Interface toward external next hop is not OSPF-enabled | `show ip ospf interface brief` | Enable OSPF on the interface with `ip ospf <process-id> area <area-id>` |
| Forwarding address did not become nonzero | Interface network type is point-to-point | `show ip ospf interface <interface-id>` | Use broadcast or nonbroadcast type if the forwarding-address behavior is required |
| Type 5 route appears as `O E2` when `O E1` was expected | Default external metric type is Type 2 | `show ip route <external-prefix>`, `show ip ospf database external <prefix>` | Configure `metric-type 1` during redistribution |
| External metric is wrong | Redistribution metric not set | `show ip ospf database external <prefix>` | Configure `metric <value>` or set metric in the redistribution route-map |
| External tag missing or wrong | Tag not configured during redistribution | `show ip ospf database external <prefix>` | Configure `tag <value>` or set tag with route-map policy |
| Type 4 ASBR summary missing in other area | ABR is not advertising reachability to ASBR | `show ip ospf database asbr-summary`, `show ip route <asbr-router-id>` | Verify ABR role, Area 0 continuity, and reachability to ASBR |
| External summary hides too much | `summary-address` range is too broad | `show ip ospf database external`, `show ip route <unused-prefix>` | Use narrower external summaries or remove the summary |
| `summary-address not-advertise` blocks more than intended | Prefix and mask match a range, not just one host | `show running-config | section router ospf` | Correct the summary prefix and mask |
| Return traffic fails even though forwarding path is optimized | Forwarding address only affects outbound path toward the external destination | `traceroute` in both directions | Verify routing from the external domain back into OSPF |
##### Source_Basis
# OSPF_External_LSAs_Forwarding_Address_And_Type5_Filtering_Mental_Model
# OSPF_External_LSAs_Forwarding_Address_And_Type5_Filtering_Configuration_Checklist
# OSPF_External_LSAs_Forwarding_Address_And_Type5_Filtering_Skeleton
# OSPF_External_LSAs_Forwarding_Address_And_Type5_Filtering_Verification_Commands
# OSPF_External_LSAs_Forwarding_Address_And_Type5_Filtering_Rollback
# OSPF_External_LSAs_Forwarding_Address_And_Type5_Filtering_Failure_Checks
# index of each title throughout note, not in table format


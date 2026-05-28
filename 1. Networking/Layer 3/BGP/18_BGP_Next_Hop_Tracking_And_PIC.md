
BGP_Next_Hop_Tracking_And_PIC.md

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` lines 115-134 | Advanced BGP Design and Implementation | Best conceptual source for BGP next-hop behavior, convergence, path selection, route reflectors, and scalable BGP design |
| `_KB_INDEX.md` lines 697 and 944 | IOS XE BGP Configuration Guide | Best practical IOS XE source for BGP configuration and verification syntax |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 22775-22820 | BGP table reading | Supports identifying BGP next hop, valid/best path state, and RIB install behavior |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 26834-26834 | Next-hop validity rule | Confirms BGP next hop must be resolvable before a path is eligible for best-path selection |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 28848-29085 | Next-hop troubleshooting | Supports troubleshooting BGP routes that appear in BGP but fail to install because the next hop is unreachable |
| `SNFC_combined_configs.md` lines 55256-55256 | BGP next-hop trigger description | Supports `bgp nexthop trigger enable` and the purpose of improving BGP response time |
| `SNFC_combined_configs.md` lines 58672-58674 | BGP next-hop trigger CLI example | Supports `bgp nexthop trigger enable` and `bgp nexthop trigger delay 5` |
| `CiscoPress_combined_part2.md` lines 9292-9301 | BGP NHT limitation and definition | Supports the mental model that NHT tracks BGP next-hop reachability and cannot properly track through only a default route |
| `CiscoPress_combined_part2.md` lines 9620-9624 | BGP PIC Edge and NHT | Supports PIC Edge relying on NHT to invalidate BGP next hops and switch to backup paths quickly |
| `CiscoPress_combined_part2.md` lines 32607-32620 | BGP PIC Core and PIC Edge distinction | Supports PIC Core for core/IGP events and PIC Edge for edge node/link failures |
| `CiscoPress_combined_part2.md` lines 32614-32619 | PIC Core default behavior | Supports the concept that modern IOS XE/XR commonly supports PIC Core through hierarchical FIB behavior |
| `CiscoPress_combined_part2.md` lines 33035-33052 | IOS XE BGP PIC Edge commands | Supports `bgp additional-paths install` and `bgp advertise-best-external` under BGP address-family configuration |
| `CiscoPress_combined_part2.md` lines 33078-33125 | PIC Edge verification | Supports checking BGP backup paths, `backup/repair`, `add-path`, RIB, and FIB backup path output |
| `CiscoPress_combined_part2.md` lines 33896-33958 | PIC Edge RIB/FIB backup path verification | Supports `show route ... detail`, backup path visibility, and FIB backup output in IOS XR-style verification |
| `CiscoPress_combined_part2.md` lines 34376-34461 | PIC Edge multipath backup verification | Supports verifying number of PIC paths, backup path, multipath, and FIB repair path behavior |
# BGP_Next_Hop_Tracking_And_PIC_Mental_Model
| Concept | Operational Meaning |
|---|---|
| BGP next hop | The IP address BGP must resolve before a route can become valid and installable |
| Next-hop reachability | A BGP path is not operational until the BGP next hop resolves through the local routing table |
| NHT | Next Hop Tracking watches next-hop reachability changes and notifies BGP faster than waiting for the periodic scanner |
| Periodic BGP scanner | Older/slower method where BGP periodically checks whether next hops are still valid |
| Event-driven convergence | NHT reacts when the RIB changes, reducing BGP convergence delay |
| Default-route weakness | If a next hop only resolves through a default route, BGP may not correctly detect specific next-hop loss |
| Summary-route weakness | If a summary still covers the failed next hop, NHT may not see the real node failure |
| Host-route resolution | Best design for NHT/PIC is specific reachability to BGP next hops, often `/32` loopbacks |
| PIC | Prefix Independent Convergence preprograms backup forwarding state so many prefixes can switch quickly after a failure |
| PIC Core | Fast convergence when the IGP path to the same BGP next hop changes |
| PIC Edge | Fast convergence when the BGP next hop itself changes, fails, or disappears |
| Hierarchical FIB | Many BGP prefixes share a next-hop pointer, so updating one pointer can repair many prefixes |
| Backup path | Non-best path precomputed and prepared for use if the primary path fails |
| Repair path | Backup path installed or programmed as the fast failover path |
| `bgp additional-paths install` | IOS XE command used in BGP PIC Edge designs to install backup paths |
| `bgp advertise-best-external` | IOS XE command that advertises a best external path and can implicitly support PIC Edge behavior |
| Multipath with PIC | Uses multiple active paths plus a protected backup path |
| NHT vs PIC | NHT detects next-hop reachability change; PIC preprograms the forwarding backup |
| PIC vs multipath | Multipath installs multiple active next hops; PIC installs backup/repair state for fast failure response |
| PIC vs Additional Paths | Additional Paths advertises extra BGP paths; PIC uses backup paths for convergence |
| Blunt rule | NHT tells BGP the next hop broke. PIC makes the failover fast. Neither one fixes a bad next-hop design, missing loopback reachability, or sloppy summarization |
# BGP_Next_Hop_Tracking_And_PIC_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm BGP neighbor state | Router | `show bgp ipv4 unicast summary` | Relevant BGP neighbors are established |
| 2 | Confirm target prefix exists in BGP | Router | `show bgp ipv4 unicast <prefix>` | Prefix appears in BGP table |
| 3 | Identify primary BGP next hop | Router | `show bgp ipv4 unicast <prefix>` | Primary next-hop IP is known |
| 4 | Identify backup BGP next hop | Router | `show bgp ipv4 unicast <prefix>` | Alternate path and backup next-hop IP are known |
| 5 | Confirm primary next-hop route | Router | `show ip route <primary-next-hop-ip>` | Specific route to primary BGP next hop exists |
| 6 | Confirm backup next-hop route | Router | `show ip route <backup-next-hop-ip>` | Specific route to backup BGP next hop exists |
| 7 | Confirm next hops are not resolved only by default route | Router | `show ip route <next-hop-ip>` | Next hop resolves through a specific route, not just `0.0.0.0/0` |
| 8 | Confirm next hops are not hidden behind only a broad summary | Router | `show ip route <next-hop-ip>` | Route resolution is specific enough to detect failure |
| 9 | Confirm sourced reachability to next hop | Router | `ping <next-hop-ip> source <local-source-ip>` | Next hop is reachable from expected source |
| 10 | Confirm CEF resolution to next hop | Router | `show ip cef <next-hop-ip>` | CEF has forwarding entry for the BGP next hop |
| 11 | Confirm current best path | Router | `show bgp ipv4 unicast <prefix>` | Best path is marked with `>` |
| 12 | Confirm route installed in RIB | Router | `show ip route <prefix>` | Prefix installs with expected active next hop |
| 13 | Confirm route installed in FIB | Router | `show ip cef <prefix>` | CEF points to expected next hop/interface |
| 14 | Confirm whether goal is NHT only or PIC Edge | Notes | `NHT` or `PIC Edge` | Mechanism target is clear before configuring |
| 15 | Confirm whether design is global IPv4 unicast or VRF/VPNv4 | Notes | `global`, `VRF`, or `VPNv4` | Correct address family context is known |
| 16 | Confirm whether platform supports explicit NHT commands | Router | `router bgp <asn>` then `address-family ipv4 unicast` then `bgp nexthop ?` | Platform exposes supported NHT syntax |
| 17 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 18 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP configuration mode |
| 19 | Enter IPv4 unicast address family | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast AFI |
| 20 | Enable BGP next-hop tracking if supported | Router | `bgp nexthop trigger enable` | BGP reacts to next-hop RIB changes |
| 21 | Set NHT trigger delay if supported | Router | `bgp nexthop trigger delay <seconds>` | NHT delay is tuned deliberately |
| 22 | Keep BGP scan time sane if scanner tuning is required | Router | `bgp scan-time <seconds>` | Scanner interval is known and not confused with NHT |
| 23 | Avoid relying on scan-time as the primary convergence tool | Notes | `Use NHT/PIC instead of only lowering scan-time` | Convergence is event-driven where possible |
| 24 | Exit IPv4 unicast AFI | Router | `exit-address-family` | Router returns to BGP config mode |
| 25 | Enter VRF address family if PIC Edge is VRF-based | Router | `address-family ipv4 vrf <vrf-name>` | Router enters BGP VRF AFI |
| 26 | Enable IOS XE PIC Edge backup-path install | Router | `bgp additional-paths install` | Router can install backup/repair BGP path for PIC Edge |
| 27 | Enable best external advertisement on backup PE if design requires it | Router | `bgp advertise-best-external` | Backup external path can be advertised instead of hidden |
| 28 | Enable iBGP multipath if PIC multipath design requires active ECMP | Router | `maximum-paths ibgp <number>` | Multiple iBGP paths can install where eligible |
| 29 | Exit VRF AFI | Router | `exit-address-family` | Router returns to BGP config mode |
| 30 | Save configuration | Router | `copy running-config startup-config` | NHT/PIC configuration is saved |
| 31 | Soft clear inbound if path eligibility changed | Router | `clear bgp ipv4 unicast <neighbor-ip> soft in` | Inbound paths are reprocessed without hard reset |
| 32 | Soft clear outbound if advertisement changed | Router | `clear bgp ipv4 unicast <neighbor-ip> soft out` | Outbound updates are refreshed |
| 33 | Verify NHT configuration | Router | `show running-config | section router bgp` | `bgp nexthop trigger` settings appear if configured |
| 34 | Verify PIC Edge configuration | Router | `show running-config | section router bgp` | `bgp additional-paths install` or `bgp advertise-best-external` appears in correct AFI |
| 35 | Verify BGP active and backup paths | Router | `show bgp ipv4 unicast <prefix>` or `show bgp vpnv4 unicast vrf <vrf-name> <prefix>` | Output shows best path plus backup/repair path where supported |
| 36 | Verify backup marker | Router | `show bgp ... <prefix>` | Output shows `backup`, `repair`, `backup/repair`, or `add-path` where supported |
| 37 | Verify RIB backup path | Router | `show ip route <prefix>` or `show ip route vrf <vrf-name> <prefix>` | RIB shows active path and backup/repair path if platform exposes it |
| 38 | Verify FIB backup path | Router | `show ip cef <prefix>` or `show ip cef vrf <vrf-name> <prefix>` | CEF shows repair/backup next-hop state if platform exposes it |
| 39 | Verify route recursion is host-specific for PIC Edge | Router | `show ip route <primary-next-hop-ip>` | Next hop resolves through specific host route, not broad default/summary |
| 40 | Test primary next-hop failure in lab | Router / Lab | `shutdown` on primary path or withdraw primary route | BGP next-hop reachability changes |
| 41 | Verify NHT reaction | Router | `show logging | include BGP|NHT|next hop|ADJCHANGE` | Logs show next-hop or BGP reaction if platform logs it |
| 42 | Verify failover route selection | Router | `show bgp ... <prefix>` | Backup/repair path becomes active or best |
| 43 | Verify RIB failover | Router | `show ip route <prefix>` | Route points to backup next hop after failure |
| 44 | Verify FIB failover | Router | `show ip cef <prefix>` | CEF points to backup next hop after failure |
| 45 | Verify traffic survives failover | Router / Host | `ping <destination> repeat <count>` or `traceroute <destination>` | Loss is reduced and path shifts to backup |
| 46 | Restore primary path | Router / Lab | `no shutdown` or restore route | Primary next hop becomes reachable again |
| 47 | Verify reconvergence after restoration | Router | `show bgp ... <prefix>` and `show ip route <prefix>` | Route returns to intended best path if policy prefers it |
# BGP_Next_Hop_Tracking_And_PIC_NHT_IOS_XE_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv4 unicast
  bgp nexthop trigger enable
  bgp nexthop trigger delay <seconds>
 exit-address-family
end
copy running-config startup-config
# BGP_Next_Hop_Tracking_And_PIC_NHT_With_Scanner_Reference_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv4 unicast
  bgp nexthop trigger enable
  bgp nexthop trigger delay <seconds>
  bgp scan-time <seconds>
 exit-address-family
end
copy running-config startup-config
# BGP_Next_Hop_Tracking_And_PIC_PIC_Edge_IOS_XE_VRF_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv4 vrf <vrf-name>
  bgp additional-paths install
 exit-address-family
end
copy running-config startup-config
# BGP_Next_Hop_Tracking_And_PIC_PIC_Edge_Best_External_IOS_XE_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv4 vrf <vrf-name>
  bgp advertise-best-external
 exit-address-family
end
copy running-config startup-config
# BGP_Next_Hop_Tracking_And_PIC_PIC_Edge_Multipath_IOS_XE_Skeleton
configure terminal
router bgp <local-asn>
 address-family ipv4 vrf <vrf-name>
  bgp additional-paths install
  maximum-paths ibgp <number>
 exit-address-family
end
copy running-config startup-config
# BGP_Next_Hop_Tracking_And_PIC_Next_Hop_Specific_Reachability_Skeleton
configure terminal
ip route <primary-bgp-next-hop-ip> 255.255.255.255 <primary-underlay-next-hop>
ip route <backup-bgp-next-hop-ip> 255.255.255.255 <backup-underlay-next-hop>
end
copy running-config startup-config
# BGP_Next_Hop_Tracking_And_PIC_OSPF_Next_Hop_Reachability_Skeleton
configure terminal
router ospf <process-id>
 network <next-hop-loopback-network> <wildcard-mask> area <area-id>
 network <transport-network> <wildcard-mask> area <area-id>
end
copy running-config startup-config
# BGP_Next_Hop_Tracking_And_PIC_Verification_Skeleton
show bgp ipv4 unicast summary
show bgp ipv4 unicast <prefix>
show bgp vpnv4 unicast vrf <vrf-name> <prefix>
show ip route <prefix>
show ip route <bgp-next-hop-ip>
show ip cef <prefix>
show ip cef <bgp-next-hop-ip>
show running-config | section router bgp
show logging | include BGP|NHT|next hop|ADJCHANGE
# BGP_Next_Hop_Tracking_And_PIC_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify BGP sessions | `show bgp ipv4 unicast summary` | Neighbors are established |
| Verify target prefix | `show bgp ipv4 unicast <prefix>` | Prefix appears in BGP table |
| Verify VPNv4/VRF prefix | `show bgp vpnv4 unicast vrf <vrf-name> <prefix>` | VRF prefix shows active and backup path where supported |
| Verify primary next hop | `show bgp ... <prefix>` | Primary BGP next hop is visible |
| Verify backup next hop | `show bgp ... <prefix>` | Backup or alternate BGP next hop is visible |
| Verify backup/repair marker | `show bgp ... <prefix>` | Output shows `backup`, `repair`, `backup/repair`, or `add-path` if PIC Edge is functioning |
| Verify next-hop route | `show ip route <bgp-next-hop-ip>` | BGP next hop resolves through a specific route |
| Verify next-hop FIB entry | `show ip cef <bgp-next-hop-ip>` | CEF can forward toward BGP next hop |
| Verify route install | `show ip route <prefix>` | Prefix installs in RIB |
| Verify route FIB | `show ip cef <prefix>` | CEF points to active next hop |
| Verify backup path in RIB if exposed | `show ip route <prefix>` or `show ip route vrf <vrf-name> <prefix>` | Output shows backup/repair path if platform exposes PIC state |
| Verify backup path in FIB if exposed | `show ip cef <prefix>` or `show ip cef vrf <vrf-name> <prefix>` | Output shows backup/repair adjacency if platform exposes PIC state |
| Verify NHT config | `show running-config | section router bgp` | `bgp nexthop trigger enable` and delay are present if configured |
| Verify PIC config | `show running-config | section router bgp` | `bgp additional-paths install` or `bgp advertise-best-external` appears in correct AFI |
| Verify path after simulated failure | `show bgp ... <prefix>` | Backup path becomes active or best |
| Verify RIB after simulated failure | `show ip route <prefix>` | Route points to backup next hop |
| Verify FIB after simulated failure | `show ip cef <prefix>` | Forwarding points to backup next hop |
| Verify logs | `show logging | include BGP|NHT|next hop|ADJCHANGE` | BGP/NHT reaction is visible if platform logs it |
# BGP_Next_Hop_Tracking_And_PIC_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify current BGP NHT/PIC config | Router | `show running-config | section router bgp` | NHT, additional-paths install, best-external, and multipath config are visible |
| 2 | Identify active and backup paths | Router | `show bgp ... <prefix>` | Active and backup path state is known before rollback |
| 3 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 4 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP configuration mode |
| 5 | Enter IPv4 unicast AFI | Router | `address-family ipv4 unicast` | Router enters IPv4 unicast AFI |
| 6 | Remove NHT delay if lab-only | Router | `no bgp nexthop trigger delay <seconds>` | NHT delay returns to default behavior |
| 7 | Disable NHT only if design requires it | Router | `no bgp nexthop trigger enable` | NHT is disabled if platform allows and rollback requires it |
| 8 | Remove scan-time tuning if lab-only | Router | `no bgp scan-time <seconds>` | BGP scanner returns to default behavior |
| 9 | Exit IPv4 unicast AFI | Router | `exit-address-family` | Router returns to BGP config mode |
| 10 | Enter VRF AFI if PIC Edge was configured there | Router | `address-family ipv4 vrf <vrf-name>` | Router enters VRF AFI |
| 11 | Remove PIC Edge backup-path install | Router | `no bgp additional-paths install` | Backup-path install behavior is removed |
| 12 | Remove best-external if lab-only | Router | `no bgp advertise-best-external` | Best external advertisement is removed |
| 13 | Remove iBGP multipath if lab-only | Router | `no maximum-paths ibgp <number>` | iBGP multipath limit is removed |
| 14 | Exit VRF AFI | Router | `exit-address-family` | Router returns to BGP config mode |
| 15 | Remove lab-only static next-hop host routes | Router | `no ip route <bgp-next-hop-ip> 255.255.255.255 <underlay-next-hop>` | Lab-only next-hop reachability route is removed |
| 16 | Soft clear inbound if path selection changed | Router | `clear bgp ipv4 unicast <neighbor-ip> soft in` | BGP paths are reprocessed |
| 17 | Verify rollback in BGP table | Router | `show bgp ... <prefix>` | Backup/repair state is removed or returns to expected baseline |
| 18 | Verify rollback in RIB | Router | `show ip route <prefix>` | Route install matches rollback design |
| 19 | Verify rollback in CEF | Router | `show ip cef <prefix>` | FIB matches rollback design |
| 20 | Save rollback | Router | `copy running-config startup-config` | Rollback is saved |
# BGP_Next_Hop_Tracking_And_PIC_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| BGP route appears but is not valid | `show bgp ipv4 unicast <prefix>` and `show ip route <next-hop-ip>` | BGP next hop is unreachable |
| Route is valid but failover is slow | `show running-config | section router bgp` | NHT not enabled/tuned, platform does not support event-driven behavior, or scanner is doing the work |
| NHT does not detect failure | `show ip route <next-hop-ip>` | Next hop still resolves through default route or broad summary |
| Failure hidden by summary | `show ip route <next-hop-ip>` | Summary still covers failed next-hop address, so BGP is not notified of specific loss |
| Failure hidden by default route | `show ip route <next-hop-ip>` | Default route keeps next hop apparently reachable |
| PIC Edge backup path missing | `show bgp ... <prefix>` | No alternate BGP path, missing `bgp additional-paths install`, or backup path filtered |
| Backup path visible but not programmed | `show ip route <prefix>` and `show ip cef <prefix>` | Platform does not expose/install repair path, or PIC Edge not enabled in correct AFI |
| `bgp additional-paths install` has no effect | `show running-config | section router bgp` | Command is under wrong address family or no eligible backup path exists |
| `bgp advertise-best-external` has no effect | `show bgp ... <prefix>` | Router has no best external path to advertise or command is under wrong AFI |
| Backup path not received by ingress PE | `show bgp ... <prefix>` | Backup PE did not advertise best external or RR/policy filtered it |
| Route reflector hides backup path | RR `show bgp ... <prefix>` | RR reflects only its best path unless Add-Path or best-external design exposes backup path |
| PIC confused with multipath | `show ip route <prefix>` | Multipath installs multiple active paths; PIC programs backup/repair path |
| PIC confused with Additional Paths | `show bgp neighbors <peer> advertised-routes` | Additional Paths advertises extra paths; PIC uses backup path for local failover |
| Backup path has unreachable next hop | `show ip route <backup-next-hop-ip>` | Backup next hop lacks specific IGP/static reachability |
| Failover works but traffic still drops | `show ip cef <prefix>` and return-path check | Forward path changes but return path, ACL, NAT, or stateful firewall path is broken |
| PIC Core expected but not visible in config | `show ip cef <prefix>` | Modern platforms may support PIC Core by default through hierarchical FIB behavior |
| Hard reset used during test | `show logging | include BGP|ADJCHANGE` | Session reset changed behavior and invalidated convergence test |
| Lab cannot prove PIC timing | Test design review | CML/virtual routers may not accurately model hardware FIB failover timing |
# Attached_Labs
| Lab Name | Attach Under This Note | Why |
|---|---|---|
| `bgp-next-hop-tracking-final` | `BGP_Next_Hop_Tracking_And_PIC.md` | Primary lab for BGP next-hop reachability tracking, event-driven convergence, and avoiding default/summary-only next-hop resolution |
| `bgp-pic-final` | `BGP_Next_Hop_Tracking_And_PIC.md` | Primary lab for Prefix Independent Convergence, backup/repair path verification, and active-to-backup failover behavior |
# Index
# Source_Basis
# BGP_Next_Hop_Tracking_And_PIC_Mental_Model
# BGP_Next_Hop_Tracking_And_PIC_Configuration_Checklist
# BGP_Next_Hop_Tracking_And_PIC_NHT_IOS_XE_Skeleton
# BGP_Next_Hop_Tracking_And_PIC_NHT_With_Scanner_Reference_Skeleton
# BGP_Next_Hop_Tracking_And_PIC_PIC_Edge_IOS_XE_VRF_Skeleton
# BGP_Next_Hop_Tracking_And_PIC_PIC_Edge_Best_External_IOS_XE_Skeleton
# BGP_Next_Hop_Tracking_And_PIC_PIC_Edge_Multipath_IOS_XE_Skeleton
# BGP_Next_Hop_Tracking_And_PIC_Next_Hop_Specific_Reachability_Skeleton
# BGP_Next_Hop_Tracking_And_PIC_OSPF_Next_Hop_Reachability_Skeleton
# BGP_Next_Hop_Tracking_And_PIC_Verification_Skeleton
# BGP_Next_Hop_Tracking_And_PIC_Verification_Commands
# BGP_Next_Hop_Tracking_And_PIC_Rollback
# BGP_Next_Hop_Tracking_And_PIC_Failure_Checks
# Attached_Labs

Blunt rule: do not document PIC as “fast BGP” in a vague way. NHT detects next-hop failure; PIC preinstalls repair state. If your next hop only resolves through a default route or a fat summary, the router may not know the real next hop died, and your fancy convergence story falls apart.

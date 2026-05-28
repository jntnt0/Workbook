


BGP_Synchronization_Legacy.md

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` lines 115-134 | Advanced BGP Design and Implementation | Best conceptual source for BGP attributes, iBGP/eBGP behavior, policy, and legacy synchronization behavior |
| `_KB_INDEX.md` lines 697 and 944 | IOS XE BGP Configuration Guide | Best practical IOS XE source for BGP configuration and verification syntax |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` line 2022 | ENARSI BGP topic list | Lists BGP synchronization as a neighbor relationship and operation topic |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` lines 23040-23065 | Early iBGP transit behavior | Explains that old transit-AS designs redistributed BGP into IGP and used synchronization to prevent blackholing |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` line 23065 | Modern status | Confirms BGP synchronization is no longer default and is not commonly used |
| `iosxe_combined_pdfs_.md` lines 33419-33419 | Synchronization with IGP | Confirms that when synchronization is enabled, iBGP-learned BGP routes must also exist in the IGP before being used |
| `iosxe_combined_pdfs_.md` lines 33517-33517 | IOS XE default setting | Confirms BGP synchronization is disabled by default |
| `All_combined_part3.md` lines 58295-58317 | Prefix synchronization definition | Defines prefix synchronization and explains when it may be disabled |
| `All_combined_part3.md` lines 58317-58317 | Operational recommendation | States it is generally good practice to disable prefix synchronization |
| `All_combined_part3.md` lines 58387-58387 | Best-path validity rule | Confirms a path is not a valid candidate if synchronization is enabled and the path is not synchronized |
| `All_combined_part3.md` lines 67523-67539 | Legacy BGP examples | Supports classic `no synchronization` under `router bgp` |
# BGP_Synchronization_Legacy_Mental_Model
| Concept | Operational Meaning |
|---|---|
| BGP synchronization | Legacy rule requiring an iBGP-learned prefix to also exist in the IGP before BGP can use or advertise it |
| Prefix synchronization | Same practical idea: the BGP prefix must be synchronized with the IGP |
| Legacy transit AS | Old design where an AS carried transit traffic and not every router in the forwarding path ran BGP |
| Modern iBGP design | All transit-path routers either run BGP, have full route visibility, or use route reflectors/confederations |
| Why synchronization existed | Prevented an edge router from advertising a route externally when middle routers had no forwarding entry |
| Why it is mostly obsolete | Modern networks do not redistribute full BGP tables into IGP and do not rely on IGP to carry Internet-scale BGP routes |
| Synchronization enabled | iBGP-learned route is not valid unless the same prefix exists in the IGP |
| Synchronization disabled | BGP can use iBGP-learned routes without requiring matching IGP route presence |
| `synchronization` | Legacy command to enable the synchronization rule |
| `no synchronization` | Classic command to disable synchronization |
| Modern default | Synchronization is disabled by default on modern IOS XE behavior |
| IGP scaling problem | Redistributing BGP into IGP is usually a bad design because IGPs are not meant to carry large BGP tables |
| iBGP full mesh requirement | Separate issue: iBGP still needs full mesh, route reflectors, or confederations to avoid route visibility problems |
| Next-hop reachability | Still required even when synchronization is disabled |
| Best-path impact | If synchronization is enabled and the route is not in the IGP, the BGP path is not a valid candidate |
| Exam/lab value | Useful for recognizing old outputs, legacy configs, and why `no synchronization` appears in older examples |
| Blunt rule | Leave synchronization disabled unless a legacy lab explicitly asks you to prove what it used to do |
# BGP_Synchronization_Legacy_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm this is a legacy synchronization lab | Notes | `BGP synchronization legacy behavior` | You are not enabling this in a normal modern design by accident |
| 2 | Confirm local AS | Router / Notes | `<local-asn>` | Local BGP AS is known |
| 3 | Confirm whether router is a transit AS router | Notes | `transit AS` or `non-transit enterprise` | Synchronization relevance is clear |
| 4 | Confirm whether all transit-path routers run BGP | Notes | `all transit routers run BGP: yes/no` | If yes, synchronization is normally unnecessary |
| 5 | Confirm whether BGP routes are redistributed into IGP | Router | `show running-config | include redistribute bgp|bgp redistribute-internal` | Dangerous legacy redistribution behavior is identified |
| 6 | Confirm BGP neighbor state | Router | `show bgp ipv4 unicast summary` | iBGP and eBGP neighbors are visible |
| 7 | Confirm iBGP-learned route exists | Router | `show bgp ipv4 unicast <prefix>` | Prefix is learned through iBGP |
| 8 | Confirm route source | Router | `show bgp ipv4 unicast <prefix>` | Route shows iBGP source, not eBGP or local origin |
| 9 | Confirm whether exact prefix exists in IGP/RIB | Router | `show ip route <prefix>` | Matching IGP or RIB route state is known |
| 10 | Confirm current synchronization state | Router | `show ip protocols` | Output shows whether IGP synchronization is enabled or disabled |
| 11 | Confirm current BGP config | Router | `show running-config | section router bgp` | `synchronization` or `no synchronization` is visible if explicitly configured |
| 12 | Confirm BGP path validity before change | Router | `show bgp ipv4 unicast <prefix>` | Current valid/best state is known |
| 13 | Confirm current route install | Router | `show ip route <prefix>` | Current RIB install state is known |
| 14 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 15 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP config mode |
| 16 | Enable synchronization only for legacy behavior testing | Router | `synchronization` | BGP requires iBGP-learned prefixes to exist in IGP before use |
| 17 | Disable synchronization for normal modern behavior | Router | `no synchronization` | BGP does not require IGP copy of iBGP-learned prefixes |
| 18 | Save configuration | Router | `copy running-config startup-config` | Configuration is saved |
| 19 | Verify synchronization state | Router | `show ip protocols` | Output shows IGP synchronization enabled or disabled |
| 20 | Verify BGP config state | Router | `show running-config | section router bgp` | Sync command state is visible if platform displays it |
| 21 | Verify iBGP route with sync enabled and no IGP match | Router | `show bgp ipv4 unicast <prefix>` | Route may fail validity/best-path eligibility if not synchronized |
| 22 | Verify route does not install if unsynchronized | Router | `show ip route <prefix>` | Prefix is absent if synchronization blocks it |
| 23 | Add exact IGP route only in a controlled legacy lab | Router | `<IGP network/redistribution/static route for exact prefix>` | Matching prefix becomes visible to the IGP/RIB |
| 24 | Verify exact IGP/RIB match after lab fix | Router | `show ip route <prefix>` | Exact prefix is present |
| 25 | Verify BGP path becomes valid after IGP match | Router | `show bgp ipv4 unicast <prefix>` | Path becomes valid and eligible if other conditions are met |
| 26 | Verify BGP route install | Router | `show ip route <prefix>` | Prefix installs if selected and valid |
| 27 | Verify eBGP advertisement behavior | Router | `show bgp ipv4 unicast neighbors <ebgp-neighbor-ip> advertised-routes` | Prefix is advertised only when valid and permitted by policy |
| 28 | Disable synchronization after lab observation | Router | `router bgp <local-asn>` then `no synchronization` | Router returns to modern expected behavior |
| 29 | Verify next-hop reachability separately | Router | `show ip route <bgp-next-hop-ip>` | Next hop is reachable regardless of synchronization state |
| 30 | Verify forwarding path | Router | `show ip cef <prefix>` | FIB points toward expected next hop when route is installed |
| 31 | Verify logs | Router | `show logging | include BGP|ADJCHANGE` | No unexpected neighbor reset occurred |
# BGP_Synchronization_Legacy_Enable_Synchronization_Skeleton
configure terminal
router bgp <local-asn>
 synchronization
end
copy running-config startup-config
# BGP_Synchronization_Legacy_Disable_Synchronization_Skeleton
configure terminal
router bgp <local-asn>
 no synchronization
end
copy running-config startup-config
# BGP_Synchronization_Legacy_Baseline_BGP_Skeleton
configure terminal
router bgp <local-asn>
 bgp router-id <router-id>
 bgp log-neighbor-changes
 no synchronization
 no bgp default ipv4-unicast
 neighbor <ibgp-neighbor-ip> remote-as <local-asn>
 neighbor <ibgp-neighbor-ip> update-source Loopback0
 neighbor <ebgp-neighbor-ip> remote-as <external-asn>
 address-family ipv4 unicast
  neighbor <ibgp-neighbor-ip> activate
  neighbor <ebgp-neighbor-ip> activate
  network <local-prefix> mask <local-mask>
 exit-address-family
end
copy running-config startup-config
# BGP_Synchronization_Legacy_Lab_IGP_Match_Skeleton
configure terminal
router ospf <process-id>
 network <matching-prefix-network> <wildcard-mask> area <area-id>
end
copy running-config startup-config
# BGP_Synchronization_Legacy_Unsafe_Redistribute_Internal_Reference
configure terminal
router bgp <local-asn>
 bgp redistribute-internal
router ospf <process-id>
 redistribute bgp <local-asn> subnets
end
copy running-config startup-config
# BGP_Synchronization_Legacy_Verification_Skeleton
show ip protocols
show running-config | section router bgp
show bgp ipv4 unicast summary
show bgp ipv4 unicast <prefix>
show ip route <prefix>
show ip route <bgp-next-hop-ip>
show ip cef <prefix>
show bgp ipv4 unicast neighbors <ebgp-neighbor-ip> advertised-routes
show logging | include BGP|ADJCHANGE
# BGP_Synchronization_Legacy_Verification_Commands
| Task | Command | Expected Result |
|---|---|---|
| Verify BGP neighbor state | `show bgp ipv4 unicast summary` | iBGP and eBGP peers are established |
| Verify synchronization state | `show ip protocols` | Output shows IGP synchronization enabled or disabled |
| Verify BGP configuration | `show running-config | section router bgp` | `synchronization` or `no synchronization` appears if explicitly configured |
| Verify iBGP-learned prefix | `show bgp ipv4 unicast <prefix>` | Prefix appears as iBGP-learned route |
| Verify BGP path validity | `show bgp ipv4 unicast <prefix>` | Path is valid only if synchronization and next-hop rules are satisfied |
| Verify exact IGP/RIB prefix | `show ip route <prefix>` | Exact prefix exists if synchronization is enabled and route must be usable |
| Verify route install | `show ip route <prefix>` | BGP route installs only if valid and selected |
| Verify next-hop reachability | `show ip route <bgp-next-hop-ip>` | BGP next hop is reachable |
| Verify forwarding entry | `show ip cef <prefix>` | CEF points toward expected next hop/interface |
| Verify eBGP advertisement | `show bgp ipv4 unicast neighbors <ebgp-neighbor-ip> advertised-routes` | Prefix is advertised only if valid and policy permits it |
| Verify no unsafe redistribution | `show running-config | include redistribute bgp|bgp redistribute-internal` | BGP-to-IGP redistribution is absent unless lab explicitly requires it |
| Verify logs | `show logging | include BGP|ADJCHANGE` | No unexpected BGP reset occurred |
# BGP_Synchronization_Legacy_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify synchronization state | Router | `show ip protocols` | Current synchronization state is known |
| 2 | Identify BGP sync config | Router | `show running-config | section router bgp` | `synchronization` or `no synchronization` is visible if configured |
| 3 | Identify lab IGP redistribution or matching route config | Router | `show running-config | include redistribute bgp|bgp redistribute-internal|network` | Legacy support config is identified |
| 4 | Enter configuration mode | Router | `configure terminal` | Router enters global configuration mode |
| 5 | Enter BGP process | Router | `router bgp <local-asn>` | Router enters BGP config mode |
| 6 | Disable synchronization | Router | `no synchronization` | Router returns to normal modern BGP behavior |
| 7 | Remove BGP internal redistribution if lab-only | Router | `no bgp redistribute-internal` | iBGP routes are no longer eligible for redistribution into IGP |
| 8 | Remove BGP-to-IGP redistribution if lab-only | Router | `router ospf <process-id>` then `no redistribute bgp <local-asn> subnets` | IGP no longer receives BGP routes |
| 9 | Remove lab-only IGP network statement if used only for sync test | Router | `router ospf <process-id>` then `no network <matching-prefix-network> <wildcard-mask> area <area-id>` | Lab-only IGP advertisement is removed |
| 10 | Verify synchronization disabled | Router | `show ip protocols` | Output shows IGP synchronization disabled |
| 11 | Verify BGP route behavior | Router | `show bgp ipv4 unicast <prefix>` | Route validity no longer depends on IGP synchronization |
| 12 | Verify route install | Router | `show ip route <prefix>` | Route installs if selected and next hop is reachable |
| 13 | Save rollback | Router | `copy running-config startup-config` | Rollback is saved |
# BGP_Synchronization_Legacy_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| iBGP route appears but is not valid | `show bgp ipv4 unicast <prefix>` and `show ip protocols` | Synchronization is enabled and the prefix is not present in the IGP |
| iBGP route does not install | `show ip route <prefix>` | Route is unsynchronized, next hop is unreachable, or another route wins |
| Route is not advertised to eBGP peer | `show bgp ipv4 unicast neighbors <ebgp-neighbor-ip> advertised-routes` | Route is not valid, synchronization blocks it, or outbound policy denies it |
| `show ip protocols` says synchronization disabled | `show ip protocols` | Expected modern behavior |
| `no synchronization` appears in old examples | `show running-config | section router bgp` | Normal legacy-template command, usually harmless if platform supports it |
| `synchronization` command is unavailable | CLI help under `router bgp` | Modern platform removed or hides legacy command |
| BGP route still fails after disabling synchronization | `show ip route <bgp-next-hop-ip>` | Next-hop reachability issue, not synchronization |
| Exact prefix exists in BGP but not IGP | `show ip route <prefix>` | Synchronization blocks route only if enabled |
| IGP has summary but BGP prefix is still blocked | `show ip route <prefix>` | Legacy synchronization expects matching route behavior, not vague reachability |
| Redistributing BGP into IGP overloads lab or design | `show ip route` and IGP database checks | Bad legacy design; IGP is carrying routes it should not carry |
| Full Internet table redistribution considered | Design review | Wrong approach; IGPs are not meant for full BGP table scale |
| Transit router cannot forward despite BGP edge route | Middle router `show ip route <prefix>` | Transit path router does not know the destination prefix |
| Disabling sync did not fix blackhole | Middle router `show ip route <prefix>` | Forwarding-path routers still lack BGP/IGP route visibility |
| Route reflector or confederation issue mistaken for sync | `show bgp ipv4 unicast summary` | iBGP route visibility problem, not synchronization |
| Route leak policy mistaken for sync | `show bgp ipv4 unicast neighbors <neighbor-ip> advertised-routes` | Outbound route policy blocks advertisement |
| Hard clear caused avoidable outage | `show logging | include BGP|ADJCHANGE` | Used hard reset instead of checking sync, route validity, and soft refresh |
# Attached_Labs
| Lab Name | Attach Under This Note | Why |
|---|---|---|
| `bgp-synchronization-final` | `BGP_Synchronization_Legacy.md` | Primary lab for legacy BGP synchronization behavior, iBGP route validity, IGP prefix dependency, and modern `no synchronization` expectation |
# Index
# Source_Basis
# BGP_Synchronization_Legacy_Mental_Model
# BGP_Synchronization_Legacy_Configuration_Checklist
# BGP_Synchronization_Legacy_Enable_Synchronization_Skeleton
# BGP_Synchronization_Legacy_Disable_Synchronization_Skeleton
# BGP_Synchronization_Legacy_Baseline_BGP_Skeleton
# BGP_Synchronization_Legacy_Lab_IGP_Match_Skeleton
# BGP_Synchronization_Legacy_Unsafe_Redistribute_Internal_Reference
# BGP_Synchronization_Legacy_Verification_Skeleton
# BGP_Synchronization_Legacy_Verification_Commands
# BGP_Synchronization_Legacy_Rollback
# BGP_Synchronization_Legacy_Failure_Checks
# Attached_Labs

Blunt rule: synchronization is legacy. Know what it does, recognize no synchronization in old configs, and move on. In modern designs, fix iBGP visibility with full mesh, route reflectors, or confederations, and fix forwarding with real next-hop reachability.
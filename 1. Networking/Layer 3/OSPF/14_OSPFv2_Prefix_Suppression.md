OSPFv2_Prefix_Suppression.md

OSPFv2_Prefix_Suppression

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | OSPF topic map | Points enterprise OSPF to `All_combined_part3.md` Book 6 and IOS XE OSPF material as project source files |
| `All_combined_part3.md` | Book 6, Chapter 8 OSPF | Supports OSPFv2 baseline process, interface participation, LSDB verification, and route verification |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | OSPF LSDB, Router LSA, Network LSA, and OSPF troubleshooting sections | Supports the LSDB mental model needed to understand why suppressing connected prefixes changes LSA contents without killing adjacency |
| Cisco IOS XE OSPF Configuration Guide | OSPF Mechanism to Exclude Connected IP Prefixes from LSA Advertisements | Provides the direct OSPFv2 syntax: `prefix-suppression`, `ip ospf prefix-suppression`, `ip ospf prefix-suppression disable`, and verification with `show ip ospf` / `show ip ospf interface`  [oai_citation:0‡Cisco](https://www.cisco.com/c/en/us/td/docs/switches/lan/cisco_ie3X00/software/16_12/b_IoT_ospf_cg/b_IoT_ospf_cg_chapter_010101.html) |
| Related lab | `ospf-prefix-suppression-final` | Main lab for suppressing connected interface prefixes from OSPFv2 LSAs while keeping OSPF adjacency and topology intact |
# OSPFv2_Prefix_Suppression_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Prefix suppression | OSPFv2 feature that excludes connected IP prefixes from OSPF LSAs while keeping the OSPF topology relationship intact |
| Main goal | Reduce the number of IP prefixes carried in LSAs, which can reduce LSDB size and improve convergence behavior in larger OSPF designs |
| Security side effect | Suppressed transit prefixes are not routed through the OSPF domain, which can reduce reachability to internal infrastructure links |
| Not adjacency suppression | OSPF neighbors still form. Hellos, adjacency, and SPF topology remain functional |
| Not route filtering | This is not the same as `distribute-list in`. Prefix suppression changes what the router originates into LSAs |
| Router LSA impact | For router LSAs, OSPF suppresses connected prefix information by excluding the stub-link prefix representation |
| Network LSA impact | For broadcast segments, the DR handles prefix suppression behavior in the network LSA |
| Global suppression | `prefix-suppression` under `router ospf` suppresses eligible connected prefixes for the OSPF process |
| Interface suppression | `ip ospf prefix-suppression` suppresses the connected prefix for one OSPF-enabled interface |
| Interface override | Interface-level prefix suppression behavior takes precedence over global process-level suppression |
| Disable override | `ip ospf prefix-suppression disable` can be used on an interface to override global suppression and keep that interface prefix advertised |
| Exceptions | Loopback, secondary IP address, and passive-interface prefixes are typically not suppressed by global prefix suppression because they are usually intended to remain reachable |
| Best use case | Suppress point-to-point or transit-link prefixes that do not need to be reachable as destinations |
| Bad use case | Do not suppress LAN, loopback, management, or service prefixes that other routers must route toward |
| Verification focus | Check both `show ip ospf` for process-level suppression and `show ip ospf interface` for per-interface suppression |
# OSPFv2_Prefix_Suppression_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify transit prefixes that do not need routing-table reachability | Design step | `<transit-prefix-list>` | Only infrastructure or transit prefixes are selected for suppression |
| 2 | Identify prefixes that must remain advertised | Design step | `<loopbacks>, <LANs>, <management>, <service prefixes>` | Critical reachable prefixes are excluded from suppression |
| 3 | Confirm OSPF baseline is healthy before changing LSA origination | All OSPF routers | `show ip ospf neighbor` | Required neighbors are `FULL` |
| 4 | Confirm OSPF-enabled interfaces | Candidate router | `show ip ospf interface brief` | Interfaces participating in OSPF are visible |
| 5 | Confirm current OSPF process status | Candidate router | `show ip ospf` | Current process state is known before suppression |
| 6 | Confirm current interface-level OSPF details | Candidate router | `show ip ospf interface <interface-id>` | Area, network type, cost, and current prefix-suppression state are known |
| 7 | Confirm current LSDB before suppression | Candidate router and downstream routers | `show ip ospf database router self-originate` | Router LSA contents are known before suppression |
| 8 | Confirm current routing-table visibility of the target transit prefix | Downstream routers | `show ip route <transit-prefix>` | Transit prefix is currently reachable before suppression |
| 9 | Enter global configuration mode for process-level suppression | Candidate router | `configure terminal` | Router enters global configuration mode |
| 10 | Enter OSPF process | Candidate router | `router ospf <process-id>` | OSPF router configuration mode opens |
| 11 | Enable process-level prefix suppression | Candidate router | `prefix-suppression` | OSPF suppresses eligible connected IP prefixes for the process |
| 12 | Exit configuration mode | Candidate router | `end` | Router returns to privileged EXEC mode |
| 13 | Save configuration | Candidate router | `write memory` | Configuration is saved |
| 14 | Verify process-level prefix suppression | Candidate router | `show ip ospf | include Prefix-suppression` | Output shows `Prefix-suppression is enabled` |
| 15 | Verify affected interfaces after global suppression | Candidate router | `show ip ospf interface` | Eligible interfaces show prefix suppression behavior |
| 16 | Enter interface configuration mode for per-interface suppression | Candidate router | `configure terminal` then `interface <interface-id>` | Interface configuration mode opens |
| 17 | Enable prefix suppression on one interface only | Candidate router | `ip ospf prefix-suppression` | OSPF suppresses the connected prefix for that interface |
| 18 | Exit configuration mode | Candidate router | `end` | Router returns to privileged EXEC mode |
| 19 | Save configuration | Candidate router | `write memory` | Configuration is saved |
| 20 | Verify per-interface prefix suppression | Candidate router | `show ip ospf interface <interface-id>` | Output shows `Prefix-suppression is enabled` for that interface |
| 21 | Exempt an interface from global suppression if its prefix must remain advertised | Candidate router | `interface <interface-id>` then `ip ospf prefix-suppression disable` | Interface overrides global suppression and keeps its prefix advertised |
| 22 | Verify exempt interface behavior | Candidate router | `show ip ospf interface <interface-id>` | Interface no longer suppresses its connected prefix |
| 23 | Recheck self-originated router LSA | Candidate router | `show ip ospf database router self-originate` | Suppressed connected prefix no longer appears as expected |
| 24 | Recheck network LSA if the interface is broadcast multiaccess | Candidate router and area routers | `show ip ospf database network` | Broadcast segment LSA behavior matches expected suppression state |
| 25 | Verify downstream routers lost the suppressed transit prefix | Downstream routers | `show ip route <suppressed-transit-prefix>` | Suppressed transit prefix is no longer installed as an OSPF route |
| 26 | Verify loopbacks and passive prefixes remain reachable | Downstream routers | `show ip route <loopback-or-service-prefix>` | Required prefixes remain in the routing table |
| 27 | Verify OSPF neighbors did not drop | Candidate router and neighbors | `show ip ospf neighbor` | Neighbor adjacencies remain `FULL` |
| 28 | Verify end-to-end traffic still works across the transit link | Downstream routers | `ping <remote-loopback-ip>` | Traffic to real routed destinations still succeeds |
| 29 | Verify nobody depends on the suppressed transit IP as a destination | Downstream routers | `ping <suppressed-transit-interface-ip>` | Ping may fail if the transit prefix is intentionally suppressed |
| 30 | Use debug only when validating LSA generation in a lab | Candidate router | `debug ip ospf lsa-generation` then `debug condition interface <interface-id>` | Logs show OSPF suppressing the connected prefix from the router LSA |
| 31 | Disable debug after testing | Candidate router | `undebug all` | Debugging is stopped |
# OSPFv2_Prefix_Suppression_Skeleton
! =========================================================
! Process-level OSPFv2 prefix suppression
! Suppresses eligible connected prefixes for the OSPF process.
! Loopbacks, secondary addresses, and passive interfaces are normally preserved.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 prefix-suppression
end
write memory
! =========================================================
! Interface-level OSPFv2 prefix suppression
! Suppresses the connected prefix only for this interface.
! =========================================================
configure terminal
interface <INTERFACE_ID>
 ip ospf prefix-suppression
end
write memory
! =========================================================
! Interface override when global prefix suppression is enabled
! Use when one interface prefix must remain advertised.
! =========================================================
configure terminal
interface <INTERFACE_ID>
 ip ospf prefix-suppression disable
end
write memory
! =========================================================
! Example
! Suppress transit link prefix on GigabitEthernet0/0 only.
! =========================================================
configure terminal
interface GigabitEthernet0/0
 ip ospf prefix-suppression
end
write memory
! =========================================================
! Example
! Enable global prefix suppression, but keep Loopback0 behavior untouched
! and exempt one service-facing interface.
! =========================================================
configure terminal
router ospf 1
 prefix-suppression
exit
interface GigabitEthernet0/1
 ip ospf prefix-suppression disable
end
write memory
# OSPFv2_Prefix_Suppression_Verification_Commands
show ip ospf
show ip ospf | include Prefix-suppression
show ip ospf interface brief
show ip ospf interface
show ip ospf interface <interface-id>
show running-config | section router ospf
show running-config interface <interface-id>
show ip ospf neighbor
show ip ospf database
show ip ospf database router
show ip ospf database router self-originate
show ip ospf database network
show ip route ospf
show ip route <suppressed-transit-prefix>
show ip route <loopback-or-service-prefix>
ping <remote-loopback-ip>
ping <suppressed-transit-interface-ip>
debug ip ospf lsa-generation
debug condition interface <interface-id>
show debugging
show logging | include OSPF|Suppressing|prefix
undebug all
# OSPFv2_Prefix_Suppression_Rollback
! =========================================================
! Remove process-level prefix suppression
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no prefix-suppression
end
write memory
! =========================================================
! Remove interface-level prefix suppression
! =========================================================
configure terminal
interface <INTERFACE_ID>
 no ip ospf prefix-suppression
end
write memory
! =========================================================
! Remove interface override for global suppression
! =========================================================
configure terminal
interface <INTERFACE_ID>
 no ip ospf prefix-suppression disable
end
write memory
! =========================================================
! Re-enable advertisement on a specific interface while global suppression remains enabled
! =========================================================
configure terminal
interface <INTERFACE_ID>
 ip ospf prefix-suppression disable
end
write memory
! =========================================================
! Stop all debugging
! =========================================================
undebug all
! =========================================================
! Lab-only OSPF reset
! This disrupts adjacencies.
! =========================================================
clear ip ospf process
# OSPFv2_Prefix_Suppression_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| Suppressed transit prefix still appears in downstream routing table | Prefix is exempt because interface-level override is configured | `show running-config interface <interface-id>` | Remove `ip ospf prefix-suppression disable` if the prefix should be suppressed |
| Suppressed transit prefix still appears | Suppression was configured on wrong interface | `show ip ospf interface brief`, `show running-config interface <interface-id>` | Apply `ip ospf prefix-suppression` to the actual OSPF-enabled transit interface |
| Suppressed transit prefix still appears | Prefix is associated with loopback, secondary address, or passive interface | `show ip ospf interface <interface-id>`, `show running-config interface <interface-id>` | This may be expected. Do not use global suppression alone for prefixes that are intentionally preserved |
| Suppression has no effect | Platform or image does not support the command | `router ospf <process-id>` then `?`, `interface <interface-id>` then `ip ospf ?` | Use a supported IOS XE image or document the limitation |
| OSPF neighbor drops after suppression | Separate issue, because prefix suppression should not kill adjacency | `show ip ospf neighbor`, `show ip ospf interface <interface-id>` | Check area, timers, MTU, authentication, passive state, and interface status |
| Remote loopback becomes unreachable | The loopback or service prefix was suppressed or no alternate route exists | `show ip route <remote-loopback>`, `show ip ospf database router` | Remove suppression from the required interface or advertise the prefix intentionally |
| Transit interface IP cannot be pinged from remote routers | Expected result when transit prefix is suppressed | `show ip route <transit-prefix>` | No fix needed if the design goal is to hide transit links |
| Management tool loses access to transit IP | Suppressed prefix was being used for management reachability | `show ip route <transit-prefix>` from management path | Do not suppress management-relevant prefixes, or manage via loopbacks |
| `show ip ospf` does not show prefix suppression | Only interface-level suppression was configured | `show ip ospf interface <interface-id>` | Verify at interface level, not only process level |
| `show ip ospf interface` does not show suppression | Only process-level suppression was configured or interface is exempt | `show ip ospf | include Prefix-suppression`, `show running-config interface <interface-id>` | Confirm whether global suppression or interface override is intended |
| LSDB still has topology links | Normal behavior | `show ip ospf database router self-originate` | Prefix suppression removes connected prefix advertisement, not the topology needed for SPF |
| Type 2 network LSA behavior looks odd on broadcast segment | DR/network LSA behavior differs from point-to-point router LSA behavior | `show ip ospf database network`, `show ip ospf interface <interface-id>` | Verify DR role and network type before assuming failure |
| Downstream route exists from another source | Static route, connected route, redistribution, or another OSPF router advertises same prefix | `show ip route <prefix>` | Remove or adjust the alternate advertisement if the prefix must disappear |
| Route remains after rollback | SPF or LSA refresh has not completed | `show ip ospf database router`, `show ip route <prefix>` | Wait for reconvergence or use `clear ip ospf process` only in lab |
| Debug output is too noisy | Debug was enabled without a condition | `show debugging` | Use `debug condition interface <interface-id>` and disable debug with `undebug all` |
| Older routers install unexpected /32 routes | Older software may interpret suppressed broadcast-prefix behavior differently | `show ip route <prefix>`, `show ip ospf database network` | Validate mixed-code behavior before using prefix suppression in production |
##### Source_Basis
# OSPFv2_Prefix_Suppression_Mental_Model
# OSPFv2_Prefix_Suppression_Configuration_Checklist
# OSPFv2_Prefix_Suppression_Skeleton
# OSPFv2_Prefix_Suppression_Verification_Commands
# OSPFv2_Prefix_Suppression_Rollback
# OSPFv2_Prefix_Suppression_Failure_Checks
# index of each title throughout note, not in table format


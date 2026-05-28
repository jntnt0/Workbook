OSPFv3_Prefix_Suppression.md

OSPFv3_Prefix_Suppression

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | OSPF topic map | Points OSPF and OSPFv3 topics to `All_combined_part3.md`, `Ios-xs_combined_epubs.md`, and ENARSI material |
| `Ios-xs_combined_epubs.md` | Chapter 21, Prefix Suppression Support for OSPFv3 | Main source for OSPFv3 prefix-suppression behavior and syntax |
| `Ios-xs_combined_epubs.md` | Prefix Suppression Support for OSPFv3 | Explains that OSPFv3 separates topology from addressing, making prefix suppression cleaner than OSPFv2 |
| `Ios-xs_combined_epubs.md` | Configure Prefix Suppression Support of the OSPFv3 Process | Provides process-level `prefix-suppression` under `router ospfv3 <process-id>` |
| `Ios-xs_combined_epubs.md` | Configure Prefix Suppression Support of the OSPFv3 Process with IPv6 address family | Provides address-family-level `prefix-suppression` under OSPFv3 AF mode |
| `Ios-xs_combined_epubs.md` | Configure Prefix Suppression Support on a per-interface basis | Provides interface-level `ipv6 ospf prefix-suppression [disable]` and `ospfv3 prefix-suppression [disable]` behavior |
| `Ios-xs_combined_epubs.md` | Troubleshoot IPv4 and IPv6 prefix suppression | Provides `debug ospfv3 lsa-generation`, `debug condition interface`, `show debugging`, and `show logging` workflow |
| Related lab | `ospfv3-prefix-suppression-final` | Main lab for suppressing OSPFv3 IPv4 and IPv6 connected-prefix advertisements while preserving OSPFv3 topology |
# OSPFv3_Prefix_Suppression_Mental_Model
| Concept | Operational Meaning |
|---|---|
| OSPFv3 prefix suppression | Suppresses advertisement of directly connected IPv4 or IPv6 prefixes from OSPFv3 LSAs |
| Main goal | Reduce prefix information in the LSDB, which can improve convergence and lower LSDB noise in large OSPFv3 deployments |
| OSPFv3 topology separation | OSPFv3 separates topology LSAs from address-prefix LSAs, so prefixes can be hidden without removing the topology relationship |
| Router LSA | OSPFv3 Router-LSAs describe topology, not attached IPv4 or IPv6 prefixes |
| Network LSA | OSPFv3 Network-LSAs describe multiaccess topology, not ordinary attached prefixes |
| Intra-Area-Prefix LSA | Carries IPv4 and IPv6 prefixes associated with routers or transit networks inside an area |
| Link LSA | Carries link-local and link-specific information. Prefix suppression does not act like a generic filter for information learned from neighbors |
| Local-only suppression | A router suppresses only prefixes locally configured on its own interfaces |
| Not neighbor filtering | Prefix suppression does not suppress or filter prefixes learned from other routers |
| Not adjacency suppression | OSPFv3 neighbors still form. The topology stays in OSPFv3; only selected connected prefix advertisement changes |
| Not route-map filtering | This is not redistribution policy, Type 3 filtering, or distribute-list behavior |
| Process-level suppression | `prefix-suppression` under `router ospfv3 <process-id>` suppresses eligible IPv4 and IPv6 connected prefixes for the process |
| Address-family suppression | `prefix-suppression` under an OSPFv3 address family suppresses prefixes for that AF scope where supported |
| Interface-level suppression | `ospfv3 prefix-suppression` or `ipv6 ospf prefix-suppression` suppresses the prefix for a specific interface |
| Interface precedence | Interface-level configuration takes precedence over process-level suppression |
| Disable override | `ospfv3 prefix-suppression disable` or `ipv6 ospf prefix-suppression disable` can exempt an interface from global suppression |
| Global exceptions | Prefixes associated with loopbacks, secondary IP addresses, and passive interfaces are not suppressed by global suppression because they commonly need to remain reachable |
| Best use case | Suppress transit-link prefixes that are not used as destination addresses |
| Bad use case | Do not suppress loopbacks, management networks, service networks, or user LANs that must remain reachable |
# OSPFv3_Prefix_Suppression_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify prefixes that should be suppressed | Design step | `<transit-prefix-list>` | Only infrastructure or transit prefixes are selected |
| 2 | Identify prefixes that must remain advertised | Design step | `<loopbacks>, <management>, <services>, <LANs>` | Critical reachable prefixes are protected from suppression |
| 3 | Confirm OSPFv3 baseline neighbor state | All involved routers | `show ospfv3 neighbor` | Required neighbors are `FULL` before suppression |
| 4 | Confirm OSPFv3 interface participation | All involved routers | `show ospfv3 interface brief` | Intended interfaces appear in the correct process, AF, and area |
| 5 | Confirm current OSPFv3 process state | Candidate router | `show ospfv3` | Current OSPFv3 process and AF state are known |
| 6 | Confirm current OSPFv3 interface state | Candidate router | `show ospfv3 interface <interface-id>` | Area, AF, cost, neighbor count, and current prefix behavior are visible |
| 7 | Confirm current OSPFv3 LSDB before suppression | Candidate and downstream routers | `show ospfv3 database` | Baseline LSDB is known before changing prefix advertisement |
| 8 | Confirm current intra-area prefix LSAs | Candidate and downstream routers | `show ospfv3 database prefix` | Current advertised prefixes are visible before suppression |
| 9 | Confirm current route-table visibility of target prefix | Downstream routers | `show ipv6 route <target-ipv6-prefix>` or `show ip route <target-ipv4-prefix>` | Target prefix is currently reachable before suppression |
| 10 | Enter global configuration mode for process-level suppression | Candidate router | `configure terminal` | Router enters global configuration mode |
| 11 | Enter OSPFv3 process | Candidate router | `router ospfv3 <process-id>` | OSPFv3 router configuration mode opens |
| 12 | Enable process-level prefix suppression | Candidate router | `prefix-suppression` | OSPFv3 suppresses eligible connected IPv4 and IPv6 prefixes for the process |
| 13 | Exit configuration mode | Candidate router | `end` | Router returns to privileged EXEC mode |
| 14 | Save configuration | Candidate router | `write memory` | Configuration is saved |
| 15 | Verify process-level suppression state | Candidate router | `show ospfv3` | OSPFv3 process output reflects prefix suppression behavior |
| 16 | Enter OSPFv3 address-family mode if suppression should be AF-scoped | Candidate router | `router ospfv3 <process-id>` then `address-family <ipv4-or-ipv6> unicast` | Router enters the selected AF configuration mode |
| 17 | Enable address-family prefix suppression | Candidate router | `prefix-suppression` | OSPFv3 suppresses eligible prefixes for the selected address family where supported |
| 18 | Exit AF configuration mode | Candidate router | `exit-address-family` | Router returns to OSPFv3 router configuration mode |
| 19 | Save AF-level suppression configuration | Candidate router | `end` then `write memory` | Configuration is saved |
| 20 | Enable per-interface prefix suppression when only one interface should be affected | Candidate router | `interface <interface-id>` then `ospfv3 prefix-suppression` | OSPFv3 suppresses the connected prefix for that interface |
| 21 | Use legacy IPv6 OSPFv3 interface syntax if the lab image expects it | Candidate router | `interface <interface-id>` then `ipv6 ospf prefix-suppression` | OSPFv3 suppresses the IPv6 prefix for that interface |
| 22 | Exempt an interface from process-level suppression | Candidate router | `interface <interface-id>` then `ospfv3 prefix-suppression disable` | Interface prefix remains advertised even when global suppression is enabled |
| 23 | Exempt an interface using legacy IPv6 syntax if required | Candidate router | `interface <interface-id>` then `ipv6 ospf prefix-suppression disable` | Interface IPv6 prefix remains advertised |
| 24 | Save interface-level suppression configuration | Candidate router | `end` then `write memory` | Configuration is saved |
| 25 | Verify running configuration | Candidate router | `show running-config | section router ospfv3` and `show running-config interface <interface-id>` | Process, AF, or interface suppression commands are present |
| 26 | Verify OSPFv3 neighbors did not drop | Candidate and neighbor routers | `show ospfv3 neighbor` | Neighbors remain `FULL` |
| 27 | Verify OSPFv3 interface status after suppression | Candidate router | `show ospfv3 interface <interface-id>` | Interface remains active in OSPFv3 |
| 28 | Verify suppressed prefix was removed from prefix LSAs | Candidate and downstream routers | `show ospfv3 database prefix` | Suppressed connected prefix is absent from the expected Intra-Area-Prefix LSA view |
| 29 | Verify topology LSAs remain present | Candidate and downstream routers | `show ospfv3 database router` and `show ospfv3 database network` | Topology information remains intact |
| 30 | Verify downstream route disappearance for suppressed IPv6 prefix | Downstream routers | `show ipv6 route <suppressed-ipv6-prefix>` | Suppressed IPv6 prefix is no longer installed through OSPFv3 |
| 31 | Verify downstream route disappearance for suppressed IPv4 AF prefix | Downstream routers | `show ip route <suppressed-ipv4-prefix>` | Suppressed IPv4 prefix is no longer installed through OSPFv3 |
| 32 | Verify required loopbacks and service prefixes remain reachable | Downstream routers | `show ipv6 route <loopback-or-service-prefix>` or `show ip route <loopback-or-service-prefix>` | Required prefixes remain advertised |
| 33 | Verify end-to-end forwarding still works through the topology | Test routers | `ping <remote-loopback>` | Traffic to real routed destinations still succeeds |
| 34 | Verify suppressed transit address is no longer a routed destination | Downstream routers | `ping <suppressed-transit-interface-address>` | Ping may fail if the transit prefix is intentionally suppressed |
| 35 | Use LSA generation debug only in a lab | Candidate router | `debug ospfv3 lsa-generation` | OSPFv3 LSA generation behavior is visible |
| 36 | Limit debug to the tested interface | Candidate router | `debug condition interface <interface-id>` | Debug output is scoped to the relevant interface |
| 37 | Verify debug state | Candidate router | `show debugging` | Active debug and condition are visible |
| 38 | Check logs for OSPFv3 prefix behavior or instability | Candidate router | `show logging | include OSPFv3|OSPF|prefix|LSA|ADJCHG` | No unexpected neighbor churn or LSA errors are present |
| 39 | Stop debug after validation | Candidate router | `undebug all` | Debugging is stopped |
# OSPFv3_Prefix_Suppression_Skeleton
! =========================================================
! Process-level OSPFv3 prefix suppression
! Suppresses eligible IPv4 and IPv6 connected prefixes
! for the OSPFv3 process.
! =========================================================
configure terminal
router ospfv3 <PROCESS_ID>
 prefix-suppression
end
write memory
! =========================================================
! Address-family-level OSPFv3 prefix suppression
! Use where the platform supports AF-scoped suppression.
! =========================================================
configure terminal
router ospfv3 <PROCESS_ID>
 address-family <ipv4-or-ipv6> unicast
  prefix-suppression
 exit-address-family
end
write memory
! =========================================================
! Interface-level OSPFv3 prefix suppression
! Suppresses the connected prefix for one interface.
! =========================================================
configure terminal
interface <INTERFACE_ID>
 ospfv3 prefix-suppression
end
write memory
! =========================================================
! Legacy IPv6 OSPFv3 interface syntax
! Use if the image expects the older command form.
! =========================================================
configure terminal
interface <INTERFACE_ID>
 ipv6 ospf prefix-suppression
end
write memory
! =========================================================
! Interface override when global suppression is enabled
! Keeps the selected interface prefix advertised.
! =========================================================
configure terminal
interface <INTERFACE_ID>
 ospfv3 prefix-suppression disable
end
write memory
! =========================================================
! Legacy IPv6 override syntax
! =========================================================
configure terminal
interface <INTERFACE_ID>
 ipv6 ospf prefix-suppression disable
end
write memory
! =========================================================
! Example process-level suppression
! =========================================================
configure terminal
router ospfv3 15
 prefix-suppression
 address-family ipv6 unicast
  router-id 0.0.0.6
 exit-address-family
end
write memory
! =========================================================
! Example interface-level suppression on dual-stack OSPFv3 interface
! =========================================================
configure terminal
interface Ethernet1/0/1
 ip address 10.0.0.1 255.255.255.0
 ipv6 address 2001:201::201/64
 ipv6 enable
 ospfv3 prefix-suppression
 ospfv3 1 ipv4 area 0
 ospfv3 1 ipv6 area 0
 no shutdown
end
write memory
# OSPFv3_Prefix_Suppression_Verification_Commands
show running-config | section router ospfv3
show running-config interface <interface-id>
show ospfv3
show ospfv3 interface brief
show ospfv3 interface <interface-id>
show ospfv3 neighbor
show ospfv3 neighbor detail
show ospfv3 database
show ospfv3 database router
show ospfv3 database network
show ospfv3 database prefix
show ospfv3 database link
show ipv6 route ospf
show ipv6 route <suppressed-ipv6-prefix>
show ipv6 route <required-ipv6-prefix>
show ip route ospfv3
show ip route <suppressed-ipv4-prefix>
show ip route <required-ipv4-prefix>
ping <remote-loopback>
ping <suppressed-transit-interface-address>
traceroute <remote-loopback>
debug ospfv3 lsa-generation
debug condition interface <interface-id>
show debugging
show logging | include OSPFv3|OSPF|prefix|LSA|ADJCHG
undebug all
# OSPFv3_Prefix_Suppression_Rollback
! =========================================================
! Remove process-level OSPFv3 prefix suppression
! =========================================================
configure terminal
router ospfv3 <PROCESS_ID>
 no prefix-suppression
end
write memory
! =========================================================
! Remove address-family-level prefix suppression
! =========================================================
configure terminal
router ospfv3 <PROCESS_ID>
 address-family <ipv4-or-ipv6> unicast
  no prefix-suppression
 exit-address-family
end
write memory
! =========================================================
! Remove interface-level OSPFv3 prefix suppression
! =========================================================
configure terminal
interface <INTERFACE_ID>
 no ospfv3 prefix-suppression
end
write memory
! =========================================================
! Remove legacy IPv6 OSPFv3 interface suppression
! =========================================================
configure terminal
interface <INTERFACE_ID>
 no ipv6 ospf prefix-suppression
end
write memory
! =========================================================
! Remove interface disable override
! =========================================================
configure terminal
interface <INTERFACE_ID>
 no ospfv3 prefix-suppression disable
end
write memory
! =========================================================
! Remove legacy IPv6 disable override
! =========================================================
configure terminal
interface <INTERFACE_ID>
 no ipv6 ospf prefix-suppression disable
end
write memory
! =========================================================
! Re-enable advertisement on one interface while global suppression remains active
! =========================================================
configure terminal
interface <INTERFACE_ID>
 ospfv3 prefix-suppression disable
end
write memory
! =========================================================
! Stop all debugging
! =========================================================
undebug all
! =========================================================
! Lab-only OSPFv3 reset
! This disrupts adjacencies.
! =========================================================
clear ospfv3 process
# OSPFv3_Prefix_Suppression_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| Suppressed prefix still appears downstream | Prefix is loopback, secondary, or passive-interface prefix exempted from global suppression | `show running-config interface <interface-id>`, `show ospfv3 interface <interface-id>` | Use interface-level design intentionally or do not expect global suppression to remove exempt prefixes |
| Suppressed prefix still appears downstream | Suppression was configured on the wrong interface | `show running-config interface <interface-id>`, `show ospfv3 interface brief` | Apply `ospfv3 prefix-suppression` to the actual OSPFv3-enabled interface |
| Suppressed prefix still appears downstream | Interface override disables suppression | `show running-config interface <interface-id>` | Remove `ospfv3 prefix-suppression disable` if the prefix should be suppressed |
| Suppressed IPv6 prefix still appears | Legacy command form is required by the image | `interface <interface-id>` then `?` | Use `ipv6 ospf prefix-suppression` if `ospfv3 prefix-suppression` is not accepted |
| Suppressed IPv4 AF prefix still appears | Address-family or interface AF behavior was not applied as expected | `show running-config | section router ospfv3`, `show ospfv3 interface brief` | Confirm OSPFv3 IPv4 AF is enabled and apply supported process or interface suppression |
| OSPFv3 neighbor drops after suppression | Separate OSPFv3 issue, because prefix suppression should not kill adjacency | `show ospfv3 neighbor`, `show ospfv3 interface <interface-id>` | Check interface state, area, timers, MTU, authentication, and IPv6 link-local reachability |
| Topology LSAs still exist | Normal behavior | `show ospfv3 database router`, `show ospfv3 database network` | No fix needed. Prefix suppression removes prefix advertisement, not topology |
| Intra-Area-Prefix LSA still carries other prefixes | Other local prefixes are not eligible for suppression or belong to another interface | `show ospfv3 database prefix` | Check interface scope, loopbacks, passive interfaces, secondary addresses, and other routers |
| Downstream route exists from another source | Static, connected, redistribution, another OSPFv3 router, or another protocol still advertises the prefix | `show ipv6 route <prefix>` or `show ip route <prefix>` | Remove or adjust the alternate advertisement if the prefix must disappear |
| Management access to transit IP breaks | Suppressed transit prefix was used for management reachability | `show ipv6 route <transit-prefix>` or `show ip route <transit-prefix>` from management path | Manage through loopbacks or do not suppress management-relevant prefixes |
| Ping to suppressed transit interface fails | Expected result when the transit prefix is intentionally hidden | `show ipv6 route <suppressed-prefix>` or `show ip route <suppressed-prefix>` | No fix needed if only the transit address disappeared and forwarding still works |
| Remote loopback becomes unreachable | Required prefix was suppressed or not exempted | `show ospfv3 database prefix`, `show ipv6 route <loopback>` or `show ip route <loopback>` | Remove interface suppression or add an override so the required prefix is advertised |
| `show ospfv3` does not clearly show interface suppression | Suppression is configured at interface level | `show running-config interface <interface-id>`, `show ospfv3 interface <interface-id>` | Verify interface config instead of process-only output |
| Debug output is too noisy | Debug was enabled without a condition | `show debugging` | Use `debug condition interface <interface-id>` and stop with `undebug all` |
| Command is rejected | Platform or image does not support OSPFv3 prefix suppression | `router ospfv3 <process-id>` then `?`, `interface <interface-id>` then `ospfv3 ?` | Use a supported IOS XE image or document the limitation |
| AF-level command is rejected | Image supports only process-level or interface-level suppression | `router ospfv3 <process-id>` then `address-family <af> unicast` then `?` | Use process-level or interface-level suppression instead |
| Suppression works for IPv6 but not IPv4 | IPv4 AF is not enabled or not using OSPFv3 AF | `show ip protocols`, `show ospfv3 interface brief` | Enable `ospfv3 <process-id> ipv4 area <area-id>` and verify IPv4 AF support |
| Suppression works for IPv4 but not IPv6 | IPv6 AF is not enabled or interface lacks IPv6 OSPFv3 participation | `show ipv6 route ospf`, `show ospfv3 interface brief` | Enable `ospfv3 <process-id> ipv6 area <area-id>` and verify IPv6 interface state |
| Routes remain after rollback longer than expected | OSPFv3 LSAs have not refreshed yet | `show ospfv3 database prefix`, `show ipv6 route <prefix>` or `show ip route <prefix>` | Wait for reconvergence or clear OSPFv3 only in a lab window |
| Forwarding still fails after unsuppressing prefix | Return path or unrelated data-plane problem | `show ipv6 route <source-prefix>` or `show ip route <source-prefix>` on return side | Verify reverse route, ACLs, and interface state |
##### Source_Basis
# OSPFv3_Prefix_Suppression_Mental_Model
# OSPFv3_Prefix_Suppression_Configuration_Checklist
# OSPFv3_Prefix_Suppression_Skeleton
# OSPFv3_Prefix_Suppression_Verification_Commands
# OSPFv3_Prefix_Suppression_Rollback
# OSPFv3_Prefix_Suppression_Failure_Checks
# index of each title throughout note, not in table format


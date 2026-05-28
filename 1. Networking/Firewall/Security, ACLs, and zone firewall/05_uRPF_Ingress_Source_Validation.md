
uRPF_Ingress_Source_Validation.md
# uRPF_Ingress_Source_Validation
# uRPF_Ingress_Source_Validation_Mental_Model
| Concept | Operational Meaning |
|---|---|
| uRPF | Unicast Reverse Path Forwarding validates the source IP address of an ingress packet before forwarding it |
| Ingress source validation | The router checks whether the packet source is reachable according to the routing or CEF table |
| CEF dependency | uRPF depends on Cisco Express Forwarding because the source lookup is performed against the forwarding table |
| Strict mode | Source must be reachable through the same interface where the packet arrived |
| Loose mode | Source only needs to be reachable through some interface in the routing table |
| Default route behavior | By default, uRPF does not treat a default route as enough source reachability unless `allow-default` is configured |
| Asymmetric routing risk | Strict mode can drop valid traffic if return traffic would use a different interface |
| Edge-facing use case | Strict mode fits simple single-homed access or customer edges where source prefixes should only arrive on one interface |
| Uplink or asymmetric use case | Loose mode fits uplinks or paths where valid traffic may return through a different interface |
| ACL exception option | An ACL can be attached to permit specific traffic that fails the uRPF check, commonly for known asymmetric exceptions |
| `allow-self-ping` | Allows the router to ping its own interface when uRPF would otherwise drop that traffic |
| Not a firewall | uRPF does not inspect sessions or applications; it only validates source reachability |
| Lab purpose | Prove valid-source traffic passes, spoofed-source traffic fails, and strict versus loose mode behavior matches the routing table |
# uRPF_Ingress_Source_Validation_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the ingress interface where source validation should occur | R1 | `show ip interface brief` | Target interface is up/up and has the expected IP address |
| 2 | Confirm the interface forwarding role | R1 | `show ip route`<br>`show cdp neighbors`<br>`show lldp neighbors` | Interface is confirmed as host-facing, customer-facing, WAN-facing, or uplink-facing |
| 3 | Confirm CEF is enabled | R1 | `show ip cef` | CEF table is present and populated |
| 4 | Enable CEF if missing | R1 | `configure terminal`<br>`ip cef`<br>`end` | CEF is enabled globally |
| 5 | Confirm route to valid source prefix | R1 | `show ip route <valid_source_ip>`<br>`show ip cef <valid_source_ip>` | Router has a route and CEF entry for the valid source |
| 6 | Confirm reverse-path interface for valid source | R1 | `show ip cef <valid_source_ip>` | CEF points back out the same interface for strict mode, or any reachable interface for loose mode |
| 7 | Confirm spoofed source should not be valid | R1 | `show ip route <spoofed_source_ip>`<br>`show ip cef <spoofed_source_ip>` | Spoofed source is missing from the table or resolves through an invalid interface |
| 8 | Enter global configuration mode | R1 | `configure terminal` | CLI enters global configuration mode |
| 9 | Enter the target ingress interface | R1 | `interface <interface>` | CLI enters interface configuration mode |
| 10 | Configure strict uRPF for symmetric edge/source-specific paths | R1 | `ip verify unicast source reachable-via rx` | Packets pass only when the source is reachable through the receiving interface |
| 11 | Configure strict uRPF with default-route acceptance if required | R1 | `ip verify unicast source reachable-via rx allow-default` | Packets may pass strict validation when the reverse path resolves through the receiving interface by default route |
| 12 | Configure loose uRPF for asymmetric or uplink paths instead of strict mode | R1 | `ip verify unicast source reachable-via any` | Packets pass if the source is reachable through any route other than default unless `allow-default` is used |
| 13 | Configure loose uRPF with default-route acceptance if required | R1 | `ip verify unicast source reachable-via any allow-default` | Packets may pass loose validation when only a default route exists |
| 14 | Allow self-ping only when the lab requires it | R1 | `ip verify unicast source reachable-via rx allow-self-ping` | Router can ping its own interface when testing requires it |
| 15 | Configure drop-rate notification threshold if required | R1 | `ip verify notification threshold <pps>` | uRPF drop-rate notification threshold is set |
| 16 | Enable uRPF SNMP drop-rate trap if required | R1 | `snmp trap ip verify drop-rate` | Device can generate uRPF drop-rate notifications |
| 17 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 18 | Verify uRPF is enabled on the interface | R1 | `show cef interface <interface>` | Output shows CEF switching enabled and unicast RPF check enabled |
| 19 | Test valid source traffic | Valid Host | `ping <destination_ip>`<br>`traceroute <destination_ip>` | Traffic from expected source succeeds |
| 20 | Test spoofed source traffic | Test Host | Generate traffic with an invalid or unexpected source IP | Traffic with spoofed source fails |
| 21 | Confirm routing decision explains the result | R1 | `show ip route <test_source_ip>`<br>`show ip cef <test_source_ip>` | Source lookup matches the pass or drop behavior |
| 22 | Save configuration | R1 | `write memory` | uRPF configuration is saved |
# uRPF_Ingress_Source_Validation_Strict_Mode_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm topology is symmetric for the source prefix | R1 | `show ip route <source_ip>`<br>`show ip cef <source_ip>` | Reverse path points out the same interface where packets arrive |
| 2 | Enter the ingress interface | R1 | `configure terminal`<br>`interface <interface>` | CLI enters target interface |
| 3 | Apply strict uRPF | R1 | `ip verify unicast source reachable-via rx` | Source must be reachable through the receiving interface |
| 4 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 5 | Verify interface uRPF state | R1 | `show cef interface <interface>` | uRPF is enabled on the interface |
| 6 | Test valid source on correct interface | Host | `ping <destination_ip>` | Valid traffic succeeds |
| 7 | Test source arriving on wrong interface | Test Host | Send traffic using a source that routes back through another interface | Traffic fails strict uRPF |
# uRPF_Ingress_Source_Validation_Loose_Mode_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the path may be asymmetric | R1 | `show ip route <source_ip>`<br>`show ip cef <source_ip>` | Source may be reachable through a different interface |
| 2 | Enter the ingress interface | R1 | `configure terminal`<br>`interface <interface>` | CLI enters target interface |
| 3 | Apply loose uRPF | R1 | `ip verify unicast source reachable-via any` | Source only needs to be reachable through the routing table |
| 4 | Add default-route acceptance only if the design requires it | R1 | `ip verify unicast source reachable-via any allow-default` | Source validation can pass when only default route reachability exists |
| 5 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 6 | Verify interface uRPF state | R1 | `show cef interface <interface>` | uRPF is enabled on the interface |
| 7 | Test reachable source | Host | `ping <destination_ip>` | Traffic succeeds when source is reachable |
| 8 | Test unreachable or bogus source | Test Host | Send traffic using an unrouted source IP | Traffic fails loose uRPF |
# uRPF_Ingress_Source_Validation_ACL_Exception_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify known asymmetric source exceptions | R1 | `show ip route <exception_source_ip>`<br>`show ip cef <exception_source_ip>` | Exception source is known and justified |
| 2 | Enter global configuration mode | R1 | `configure terminal` | CLI enters global configuration mode |
| 3 | Create uRPF exception ACL | R1 | `ip access-list extended URPF_EXCEPTION` | CLI enters ACL configuration mode |
| 4 | Permit known traffic that may fail the uRPF check but should be forwarded | R1 | `permit ip <exception_source> <exception_wildcard> <destination> <destination_wildcard>` | Known asymmetric exception is permitted |
| 5 | Deny everything else from using the exception path | R1 | `deny ip any any` | Other failed uRPF traffic is not excepted |
| 6 | Exit ACL configuration mode | R1 | `exit` | CLI returns to global configuration mode |
| 7 | Enter the protected ingress interface | R1 | `interface <interface>` | CLI enters target interface |
| 8 | Apply uRPF with exception ACL | R1 | `ip verify unicast source reachable-via rx URPF_EXCEPTION` | Strict uRPF is enabled with explicit exceptions |
| 9 | Use loose mode with exception ACL if required | R1 | `ip verify unicast source reachable-via any URPF_EXCEPTION` | Loose uRPF is enabled with explicit exceptions |
| 10 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 11 | Verify ACL and uRPF state | R1 | `show access-lists URPF_EXCEPTION`<br>`show cef interface <interface>` | Exception ACL exists and uRPF is enabled |
| 12 | Test exception traffic | Exception Host | `ping <destination_ip>` | Approved exception traffic succeeds |
| 13 | Test non-exception spoofed traffic | Test Host | Send traffic from an unapproved source | Non-exception failed traffic is dropped |
# uRPF_Ingress_Source_Validation_Skeleton
configure terminal
ip cef
interface <interface>
 ip verify unicast source reachable-via rx
 no shutdown
exit
end
write memory
# uRPF_Ingress_Source_Validation_Strict_Mode_Skeleton
configure terminal
ip cef
interface <interface>
 ip verify unicast source reachable-via rx
 no shutdown
exit
end
write memory
# uRPF_Ingress_Source_Validation_Strict_Mode_Allow_Default_Skeleton
configure terminal
ip cef
interface <interface>
 ip verify unicast source reachable-via rx allow-default
 no shutdown
exit
end
write memory
# uRPF_Ingress_Source_Validation_Loose_Mode_Skeleton
configure terminal
ip cef
interface <interface>
 ip verify unicast source reachable-via any
 no shutdown
exit
end
write memory
# uRPF_Ingress_Source_Validation_Loose_Mode_Allow_Default_Skeleton
configure terminal
ip cef
interface <interface>
 ip verify unicast source reachable-via any allow-default
 no shutdown
exit
end
write memory
# uRPF_Ingress_Source_Validation_ACL_Exception_Skeleton
configure terminal
ip cef
ip access-list extended URPF_EXCEPTION
 permit ip <exception_source> <exception_wildcard> <destination> <destination_wildcard>
 deny ip any any
exit
interface <interface>
 ip verify unicast source reachable-via rx URPF_EXCEPTION
 no shutdown
exit
end
write memory
# uRPF_Ingress_Source_Validation_Verification_Commands
| Check | Device | Command | Expected Result |
|---|---|---|---|
| Interface state | R1 | `show ip interface brief` | Protected interface is up/up |
| CEF global table | R1 | `show ip cef` | CEF table is populated |
| CEF interface state | R1 | `show cef interface <interface>` | CEF switching is enabled on the interface |
| uRPF interface state | R1 | `show cef interface <interface>` | Output shows unicast RPF check is enabled |
| Reverse route lookup | R1 | `show ip route <source_ip>` | Source route is present or absent as expected |
| Reverse CEF lookup | R1 | `show ip cef <source_ip>` | Source resolves through expected interface or no route exists |
| Strict mode path check | R1 | `show ip cef <source_ip>` | For strict mode, source resolves out the same interface where traffic arrives |
| Loose mode path check | R1 | `show ip route <source_ip>` | For loose mode, source is reachable through any non-default route unless `allow-default` is configured |
| Interface running config | R1 | `show running-config interface <interface>` | `ip verify unicast source reachable-via` command is present |
| Exception ACL check | R1 | `show access-lists URPF_EXCEPTION` | Exception ACL exists and counters increment only for exception traffic |
| Valid source test | Valid Host | `ping <destination_ip>` | Valid traffic succeeds |
| Spoofed source test | Test Host | Generate traffic using invalid source IP | Spoofed traffic fails |
| Logging or telemetry check | R1 | `show logging` | Related drop or notification messages appear if supported and configured |
# uRPF_Ingress_Source_Validation_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter the protected interface | R1 | `configure terminal`<br>`interface <interface>` | CLI enters interface configuration mode |
| 2 | Remove uRPF from the interface | R1 | `no ip verify unicast source reachable-via` | uRPF is removed from the interface |
| 3 | Remove notification threshold if configured | R1 | `no ip verify notification threshold` | uRPF notification threshold is removed |
| 4 | Remove SNMP uRPF drop-rate trap if configured and no longer needed | R1 | `no snmp trap ip verify drop-rate` | uRPF SNMP drop-rate trap is disabled |
| 5 | Exit interface configuration mode | R1 | `exit` | CLI returns to global configuration mode |
| 6 | Remove exception ACL if not reused | R1 | `no ip access-list extended URPF_EXCEPTION` | Exception ACL is deleted |
| 7 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 8 | Confirm uRPF removal | R1 | `show cef interface <interface>` | Output no longer shows unicast RPF check enabled |
| 9 | Confirm interface config cleanup | R1 | `show running-config interface <interface>` | `ip verify unicast source reachable-via` is absent |
| 10 | Save rollback | R1 | `write memory` | Rollback is saved |
# uRPF_Ingress_Source_Validation_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| uRPF command is accepted but traffic behaves unexpectedly | CEF route for source does not match the intended path | R1 | `show ip cef <source_ip>` | Correct routing or use the correct uRPF mode |
| Valid traffic drops in strict mode | Return path points to a different interface | R1 | `show ip route <source_ip>`<br>`show ip cef <source_ip>` | Use loose mode or fix routing symmetry |
| Valid traffic drops when only default route exists | `allow-default` is missing | R1 | `show ip route <source_ip>` | Add `allow-default` only if default-route validation is intended |
| Spoofed traffic still passes | uRPF is not applied on the ingress interface | R1 | `show running-config interface <interface>`<br>`show cef interface <interface>` | Apply uRPF to the interface where traffic enters |
| Spoofed traffic still passes in loose mode | Source is reachable through some route | R1 | `show ip route <spoofed_source_ip>` | Use strict mode where symmetry is guaranteed or add filtering for bogus sources |
| Spoofed traffic still passes because of default route | Loose or strict uRPF is configured with `allow-default` | R1 | `show running-config interface <interface>` | Remove `allow-default` if default-route validation is too permissive |
| Exception traffic fails | Exception ACL does not permit the failed uRPF traffic | R1 | `show access-lists URPF_EXCEPTION` | Correct ACL source, destination, wildcard, or protocol |
| Too much traffic bypasses uRPF failure behavior | Exception ACL is too broad | R1 | `show access-lists URPF_EXCEPTION` | Narrow the exception ACL |
| Router cannot ping its own interface | uRPF drops self-ping behavior by default | R1 | `show running-config interface <interface>` | Add `allow-self-ping` only if required for the lab |
| No uRPF evidence in interface output | Checking wrong interface or unsupported platform behavior | R1 | `show cef interface <interface>`<br>`show running-config interface <interface>` | Verify the actual ingress interface and platform support |
| Valid source has no route | Missing connected, static, or dynamic route to source prefix | R1 | `show ip route <source_ip>` | Add or repair the route to the valid source prefix |
| Strict mode works in one direction but fails in another | Asymmetric routing exists between paths | R1 | `traceroute <source_ip>`<br>`show ip cef <source_ip>` | Use loose mode on asymmetric uplinks |
| Lab cannot prove spoofed-source failure | Test host cannot actually generate spoofed source traffic | Test Host | `ip addr`<br>`tcpdump` | Configure a real alternate source IP or use traffic generation that permits spoofing |
| uRPF removed but traffic still drops | ACL, ZBFW, CoPP, VACL, or another feature is blocking traffic | R1 | `show access-lists`<br>`show policy-map control-plane`<br>`show running-config interface <interface>` | Remove or fix the remaining enforcement feature |
# uRPF_Ingress_Source_Validation_Related_Labs
- urpf-final
# uRPF_Ingress_Source_Validation_Index
uRPF_Ingress_Source_Validation.md
# uRPF_Ingress_Source_Validation
# uRPF_Ingress_Source_Validation_Mental_Model
# uRPF_Ingress_Source_Validation_Configuration_Checklist
# uRPF_Ingress_Source_Validation_Strict_Mode_Checklist
# uRPF_Ingress_Source_Validation_Loose_Mode_Checklist
# uRPF_Ingress_Source_Validation_ACL_Exception_Checklist
# uRPF_Ingress_Source_Validation_Skeleton
# uRPF_Ingress_Source_Validation_Strict_Mode_Skeleton
# uRPF_Ingress_Source_Validation_Strict_Mode_Allow_Default_Skeleton
# uRPF_Ingress_Source_Validation_Loose_Mode_Skeleton
# uRPF_Ingress_Source_Validation_Loose_Mode_Allow_Default_Skeleton
# uRPF_Ingress_Source_Validation_ACL_Exception_Skeleton
# uRPF_Ingress_Source_Validation_Verification_Commands
# uRPF_Ingress_Source_Validation_Rollback
# uRPF_Ingress_Source_Validation_Failure_Checks
# uRPF_Ingress_Source_Validation_Related_Labs
# uRPF_Ingress_Source_Validation_Index


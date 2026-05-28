EIGRP_Default_Route_Injection_Default_Network.md

EIGRP_Default_Route_Injection_Default_Network

# Source_Basis
| Source                  | Relevant Section                                                                   | What It Supports                                                                                                                      |
| ----------------------- | ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| `_KB_INDEX.md`          | Book 3: EIGRP for IP, Chapter 8: Default Routes                                    | Points EIGRP default-route behavior to `All_combined_part3.md`                                                                        |
| `All_combined_part3.md` | Book 3, Chapter 8: Default Routes                                                  | Supports IOS default routing, default candidates, gateway of last resort, and EIGRP default candidate behavior                        |
| `All_combined_part3.md` | `ip default-network` behavior table                                                | Supports how `ip default-network <major-network>` marks a network as a default candidate and can inject it into EIGRP                 |
| `All_combined_part3.md` | Monitoring Default Candidates                                                      | Supports verification with `show ip route` and `show ip eigrp topology <network>`                                                     |
| `All_combined_part3.md` | Default Routes and Default Candidates in EIGRP                                     | Supports the distinction between `0.0.0.0/0` default routes and EIGRP default candidate networks                                      |
| `All_combined_part3.md` | Default Routing toward the Internet Implemented with the `default-network` Command | Supports the operational model where a gateway router marks an external network as the default candidate for downstream EIGRP routers |
# EIGRP_Default_Route_Injection_Default_Network_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Default route | `0.0.0.0/0`; used when no more specific route matches |
| Default candidate | A route marked with `*` in the routing table that IOS can use to choose the gateway of last resort |
| Gateway of last resort | The next hop IOS selects from the best default candidate |
| `ip default-network` | Marks a network as a default candidate |
| EIGRP default candidate propagation | EIGRP can carry the default candidate marker to downstream routers |
| Default network method | Does not necessarily advertise `0.0.0.0/0`; it can advertise a candidate default network such as `D* 192.77.3.0/24` |
| Gateway router | The router with the real exit toward the ISP, firewall, or upstream network |
| Downstream router | A router that learns the candidate default through EIGRP and sets its gateway of last resort toward the gateway router |
| Connected default network | If the marked major network is connected on the gateway router, EIGRP can automatically advertise it with the default candidate flag |
| Nonconnected default network | If the marked network is not already in EIGRP, `ip default-network` alone may not inject it into the EIGRP topology |
| Classful behavior risk | `ip default-network` is old IOS behavior and is tied to major-network logic; avoid sloppy subnet choices |
| Modern preference | For clean modern labs, static default plus redistribution or EIGRP summary default is usually clearer, but this note is specifically for the `ip default-network` mechanism |
# EIGRP_Default_Route_Injection_Default_Network_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the gateway router has an upstream-facing interface | Gateway router | `show ip interface brief` | ISP/firewall-facing interface is `up/up` with the expected IPv4 address |
| 2 | Confirm the gateway router has inside EIGRP-facing interfaces | Gateway router | `show ip interface brief` | Internal EIGRP-facing interfaces are `up/up` |
| 3 | Confirm downstream routers have EIGRP-facing interfaces up | Downstream routers | `show ip interface brief` | EIGRP-facing transit interfaces are `up/up` |
| 4 | Confirm the upstream network is connected on the gateway router | Gateway router | `show ip route connected` | The external or upstream network appears as connected |
| 5 | Confirm baseline EIGRP neighbors are stable | All EIGRP routers | `show ip eigrp neighbors` | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| 6 | Confirm baseline EIGRP process state | All EIGRP routers | `show ip protocols` | Correct EIGRP AS, router ID, network statements, passive interfaces, and metric values are shown |
| 7 | Confirm downstream routers do not yet have a gateway of last resort | Downstream routers | `show ip route` | Output shows `Gateway of last resort is not set` before default injection |
| 8 | Enter global configuration mode on the gateway router | Gateway router | `configure terminal` | Router enters global configuration mode |
| 9 | Ensure classless forwarding behavior is enabled | Gateway router | `ip classless` | Router uses classless forwarding behavior |
| 10 | Enter the classic EIGRP process | Gateway router | `router eigrp <AS_NUMBER>` | Router enters EIGRP router configuration mode |
| 11 | Ensure the internal EIGRP network is advertised | Gateway router | `network <INSIDE_NETWORK> <WILDCARD_MASK>` | Gateway router participates in EIGRP on the inside-facing interface |
| 12 | Exit EIGRP router configuration mode | Gateway router | `exit` | Router returns to global configuration mode |
| 13 | Mark the upstream major network as the default candidate | Gateway router | `ip default-network <UPSTREAM_MAJOR_NETWORK>` | The upstream network is marked as a candidate default |
| 14 | Exit configuration mode | Gateway router | `end` | Router returns to privileged EXEC mode |
| 15 | Verify the gateway router marks the default candidate locally | Gateway router | `show ip route` | The marked network appears with `*` or the gateway of last resort is set toward that network |
| 16 | Verify the marked network appears in the EIGRP topology table | Gateway router | `show ip eigrp topology <UPSTREAM_MAJOR_NETWORK>` | Topology entry exists and shows the default candidate or exterior flag behavior |
| 17 | Verify downstream routers receive the candidate default | Downstream routers | `show ip route` | A learned EIGRP route appears with `D*` and `Gateway of last resort` points toward the EIGRP next hop |
| 18 | Verify downstream EIGRP topology state for the default candidate | Downstream routers | `show ip eigrp topology <UPSTREAM_MAJOR_NETWORK>` | Route is passive and has a successor toward the gateway router |
| 19 | Confirm the route is internal or external as expected | Downstream routers | `show ip route <UPSTREAM_MAJOR_NETWORK>` | Route appears with expected EIGRP code, next hop, metric, and outgoing interface |
| 20 | Test reachability to an upstream test address | Downstream routers | `ping <UPSTREAM_TEST_IP>` | Ping succeeds if upstream routing and return path exist |
| 21 | Trace the forwarding path toward an unknown or upstream destination | Downstream routers | `traceroute <UPSTREAM_TEST_IP>` | Traffic exits toward the gateway router |
| 22 | Optionally filter incoming default candidate markers | Receiving router | `router eigrp <AS_NUMBER>` then `default-information in <ACL>` | Only matched routes keep the default candidate marker |
| 23 | Optionally filter outgoing default candidate markers | Advertising router | `router eigrp <AS_NUMBER>` then `default-information out <ACL>` | Only matched routes are advertised with the default candidate marker |
| 24 | Verify default-information policy if configured | Affected routers | `show running-config | section router eigrp` | `default-information in` or `default-information out` appears only where intended |
| 25 | Verify final gateway of last resort selection | Downstream routers | `show ip route` | Gateway of last resort points toward the intended gateway router |
| 26 | Save the configuration | Changed routers | `write memory` | Configuration is saved |
# EIGRP_Default_Route_Injection_Default_Network_Skeleton
! Baseline checks
show ip interface brief
show ip route connected
show ip eigrp neighbors
show ip protocols
show ip route
! Gateway router configuration
configure terminal
 ip classless
 router eigrp <AS_NUMBER>
  network <INSIDE_NETWORK> <WILDCARD_MASK>
 exit
 ip default-network <UPSTREAM_MAJOR_NETWORK>
end
! Gateway verification
show ip route
show ip eigrp topology <UPSTREAM_MAJOR_NETWORK>
show ip protocols
! Downstream verification
show ip route
show ip route <UPSTREAM_MAJOR_NETWORK>
show ip eigrp topology <UPSTREAM_MAJOR_NETWORK>
ping <UPSTREAM_TEST_IP>
traceroute <UPSTREAM_TEST_IP>
! Optional default candidate filtering
configure terminal
 access-list <ACL_NUMBER> permit <DEFAULT_CANDIDATE_NETWORK> <WILDCARD_MASK>
 router eigrp <AS_NUMBER>
  default-information in <ACL_NUMBER>
  ! or
  default-information out <ACL_NUMBER>
end
! Final save
write memory
# EIGRP_Default_Route_Injection_Default_Network_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state before default injection | Gateway upstream and EIGRP inside interfaces are `up/up` |
| `show ip route connected` | Confirms the upstream default candidate network exists locally | Upstream network appears as connected on the gateway router |
| `show ip eigrp neighbors` | Confirms EIGRP adjacency health | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| `show ip protocols` | Confirms EIGRP process state | Correct AS, network statements, passive interfaces, K-values, and AD values |
| `show running-config | include default-network` | Confirms global default-network command | Shows `ip default-network <UPSTREAM_MAJOR_NETWORK>` |
| `show running-config | section router eigrp` | Confirms EIGRP configuration | Shows correct EIGRP process and optional `default-information` controls |
| `show ip route` | Confirms gateway of last resort and candidate default status | Downstream routers show `Gateway of last resort` and a `D*` candidate route |
| `show ip route <UPSTREAM_MAJOR_NETWORK>` | Confirms exact RIB entry for the default candidate network | Route points toward the expected EIGRP next hop |
| `show ip eigrp topology <UPSTREAM_MAJOR_NETWORK>` | Confirms EIGRP topology entry and candidate behavior | Route is passive, has a successor, and shows default candidate behavior |
| `show ip eigrp topology all-links` | Shows all known EIGRP paths for the candidate network | Expected alternate paths appear if present |
| `ping <UPSTREAM_TEST_IP>` | Confirms data-plane reachability toward upstream | Ping succeeds if upstream and return routing exist |
| `traceroute <UPSTREAM_TEST_IP>` | Confirms forwarding exits toward the gateway router | First EIGRP hop points toward the expected gateway path |
# EIGRP_Default_Route_Injection_Default_Network_Rollback
! Remove the default candidate marking from the gateway router
configure terminal
 no ip default-network <UPSTREAM_MAJOR_NETWORK>
end
! Remove optional default-information controls if configured
configure terminal
 router eigrp <AS_NUMBER>
  no default-information in
  no default-information out
end
! Remove optional ACL if it was created only for default-information control
configure terminal
 no access-list <ACL_NUMBER>
end
! Verify rollback on gateway router
show running-config | include default-network
show running-config | section router eigrp
show ip route
show ip eigrp topology <UPSTREAM_MAJOR_NETWORK>
! Verify rollback on downstream routers
show ip route
show ip route <UPSTREAM_MAJOR_NETWORK>
show ip eigrp topology <UPSTREAM_MAJOR_NETWORK>
! Expected result:
! The default candidate marker is removed.
! Downstream routers no longer use that EIGRP-learned network as the gateway of last resort.
write memory
# EIGRP_Default_Route_Injection_Default_Network_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| Downstream router says `Gateway of last resort is not set` | Default candidate marker was not propagated | `show ip route` and `show ip eigrp topology <UPSTREAM_MAJOR_NETWORK>` | Confirm `ip default-network <UPSTREAM_MAJOR_NETWORK>` exists on the gateway router |
| Gateway router does not mark the network as default candidate | Wrong network was used | `show running-config | include default-network` and `show ip route connected` | Use the correct upstream major network |
| Candidate route does not appear downstream | EIGRP is not advertising the marked network | `show ip protocols` and `show ip eigrp topology <UPSTREAM_MAJOR_NETWORK>` | Ensure the marked connected network is eligible for EIGRP propagation |
| Candidate route appears but traffic fails | Upstream return route is missing | `show ip route <DOWNSTREAM_SOURCE_PREFIX>` on upstream device | Add return routing upstream or NAT at the edge |
| Candidate route appears but wrong exit is selected | Multiple default candidates exist | `show ip route` | Compare AD and metrics; tune EIGRP metrics or remove unintended candidates |
| `D*` route appears but no `0.0.0.0/0` appears | Normal default-network behavior | `show ip route` | No fix needed; `ip default-network` marks a candidate network, not necessarily `0.0.0.0/0` |
| `ip default-network <subnet>` behaves strangely | Subnet form can create major-network summary behavior | `show ip route` | Prefer marking the correct major network or use static default redistribution instead |
| Downstream route is missing from topology table | Neighbor or network statement issue | `show ip eigrp neighbors` and `show ip protocols` | Fix EIGRP adjacency or network participation first |
| Candidate route is filtered | `default-information in/out` or route filtering is blocking candidate behavior | `show running-config | section router eigrp` | Remove or correct `default-information` and filtering policy |
| Candidate route is present but not selected as gateway of last resort | Better default candidate exists | `show ip route` | Remove the better candidate or tune metrics intentionally |
| Candidate network is internal when expected external | Network is connected and automatically carried by EIGRP | `show ip eigrp topology <UPSTREAM_MAJOR_NETWORK>` | Accept if intended, or use static default redistribution for cleaner external control |
| Default route design is confusing | `ip default-network` is old classful-style behavior | `show ip route` | Use `ip route 0.0.0.0 0.0.0.0 <NEXT_HOP>` plus EIGRP redistribution or summary default in modern designs |
# Index
# Source_Basis
# EIGRP_Default_Route_Injection_Default_Network_Mental_Model
# EIGRP_Default_Route_Injection_Default_Network_Configuration_Checklist
# EIGRP_Default_Route_Injection_Default_Network_Skeleton
# EIGRP_Default_Route_Injection_Default_Network_Verification_Commands
# EIGRP_Default_Route_Injection_Default_Network_Rollback
# EIGRP_Default_Route_Injection_Default_Network_Failure_Checks
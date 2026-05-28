EIGRP_Foundation_Classic_Mode.md

EIGRP_Foundation_Classic_Mode

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | `All_combined_part3.md`, Book 3: EIGRP for IP | Points EIGRP concepts, DUAL, feasible successors, metrics, summarization, filtering, defaults, redistribution, and authentication to the main EIGRP source |
| `All_combined_part3.md` | Book 3, Chapter 1: Initial IP EIGRP Configuration | Supports classic EIGRP startup syntax: `router eigrp <as-number>` and `network <major-network>` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | EIGRP Configuration Modes, Classic Configuration Mode | Supports modern classic-mode syntax: `router eigrp <as-number>` plus `network <ip-address> <wildcard-mask>` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | EIGRP Network Statement | Confirms the `network` command enables EIGRP on matching interfaces and adds their connected networks to the EIGRP topology table |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Confirming Interfaces, Verifying Neighbor Adjacencies, Displaying Installed EIGRP Routes | Supports verification commands: `show ip eigrp interfaces`, `show ip eigrp neighbors`, and `show ip route eigrp` |
# EIGRP_Foundation_Classic_Mode_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Classic EIGRP process | Started globally with `router eigrp <AS>` |
| AS number | Must match between routers that should become EIGRP neighbors |
| Network statement | Enables EIGRP on interfaces whose IP addresses match the statement |
| Wildcard mask | Controls how broadly or narrowly interfaces are matched |
| EIGRP-enabled interface | Sends hellos, discovers neighbors, and advertises its connected subnet |
| Passive interface | Advertises the connected network but suppresses EIGRP hellos and neighbor formation |
| Neighbor adjacency | Required before EIGRP route exchange happens |
| Topology table | EIGRP’s route-decision database, not the same thing as the routing table |
| Routing table | Only the best selected EIGRP routes appear here as `D` routes |
| Internal EIGRP route | Appears as `D`, default AD 90 |
| External EIGRP route | Appears as `D EX`, default AD 170 |
| Foundation failure point | Most basic EIGRP failures are wrong AS number, wrong network statement, passive interface, interface down, or mismatched authentication/K-values if configured |
# EIGRP_Foundation_Classic_Mode_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm all routed interfaces are up before configuring EIGRP | All routers | `show ip interface brief` | EIGRP-facing interfaces are `up/up` with correct IPv4 addresses |
| 2 | Confirm each router has directly connected routes for its local links | All routers | `show ip route connected` | Local LAN, loopback, and transit networks appear as connected routes |
| 3 | Enter global configuration mode | All routers | `configure terminal` | Router enters `(config)#` mode |
| 4 | Start the classic EIGRP process | All routers | `router eigrp <AS_NUMBER>` | Router enters `(config-router)#` mode |
| 5 | Set a stable EIGRP router ID | All routers | `eigrp router-id <ROUTER_ID>` | Router ID is manually defined and not dependent on interface selection |
| 6 | Disable automatic classful summarization if the command exists on the platform | All routers | `no auto-summary` | EIGRP does not summarize at classful network boundaries |
| 7 | Enable EIGRP on a specific transit interface using host wildcard matching | All routers | `network <INTERFACE_IP> 0.0.0.0` | Only the interface with that exact IP joins EIGRP |
| 8 | Enable EIGRP on a full subnet when appropriate | All routers | `network <SUBNET_ID> <WILDCARD_MASK>` | All local interfaces matching that subnet join EIGRP |
| 9 | Enable EIGRP for loopback advertisement if loopbacks are used | All routers | `network <LOOPBACK_IP> 0.0.0.0` | Loopback prefix is inserted into the EIGRP topology table |
| 10 | Prevent accidental neighbor formation on non-router-facing interfaces | Edge/access routers | `passive-interface default` | All EIGRP-enabled interfaces stop sending hellos by default |
| 11 | Re-enable EIGRP hellos only on router-facing links | Edge/access routers | `no passive-interface <TRANSIT_INTERFACE>` | Neighbor-facing interface can form EIGRP adjacency |
| 12 | Exit router configuration mode | All routers | `end` | Router returns to privileged EXEC mode |
| 13 | Verify EIGRP is enabled on the intended interfaces | All routers | `show ip eigrp interfaces` | Router-facing interfaces appear; passive interfaces may not appear |
| 14 | Verify EIGRP neighbor formation | All routers | `show ip eigrp neighbors` | Expected neighbors appear with stable uptime and `Q Cnt` of `0` |
| 15 | Verify EIGRP process details | All routers | `show ip protocols` | Correct AS, router ID, network statements, passive interfaces, and AD values are shown |
| 16 | Verify EIGRP learned routes entered the routing table | All routers | `show ip route eigrp` | Remote internal EIGRP routes appear as `D` |
| 17 | Verify EIGRP topology table has expected prefixes | All routers | `show ip eigrp topology` | Expected prefixes appear as passive `P` routes |
| 18 | Test end-to-end reachability across learned EIGRP routes | Edge routers/hosts | `ping <REMOTE_PREFIX_IP>` | Ping succeeds across the EIGRP domain |
| 19 | Trace the forwarding path if ping succeeds or fails | Edge routers/hosts | `traceroute <REMOTE_PREFIX_IP>` | Path follows expected EIGRP next hops |
| 20 | Save the working configuration | All routers | `write memory` | Configuration is saved to startup config |
# EIGRP_Foundation_Classic_Mode_Skeleton
! Baseline interface check
show ip interface brief
show ip route connected
! Classic EIGRP configuration
configure terminal
 router eigrp <AS_NUMBER>
  eigrp router-id <ROUTER_ID>
  no auto-summary
  network <TRANSIT_INTERFACE_IP_1> 0.0.0.0
  network <TRANSIT_INTERFACE_IP_2> 0.0.0.0
  network <LOOPBACK_IP> 0.0.0.0
  passive-interface default
  no passive-interface <ROUTER_FACING_INTERFACE_1>
  no passive-interface <ROUTER_FACING_INTERFACE_2>
 end
! Verification
show ip eigrp interfaces
show ip eigrp neighbors
show ip protocols
show ip eigrp topology
show ip route eigrp
ping <REMOTE_IP>
traceroute <REMOTE_IP>
! Save
write memory
# EIGRP_Foundation_Classic_Mode_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state and addressing | EIGRP-facing interfaces are `up/up` |
| `show ip route connected` | Confirms local connected networks exist before EIGRP advertises them | Transit, LAN, and loopback prefixes appear as connected |
| `show running-config | section router eigrp` | Confirms classic EIGRP configuration | Shows `router eigrp <AS>`, router ID, network statements, and passive-interface settings |
| `show ip eigrp interfaces` | Confirms which interfaces are actively running EIGRP | Expected router-facing interfaces appear |
| `show ip eigrp interfaces detail` | Confirms timers, split horizon, authentication status, and packet counters | Interface has expected hello/hold timers and stable counters |
| `show ip eigrp neighbors` | Confirms adjacency formation | Expected neighbor IPs appear with stable uptime and `Q Cnt` of `0` |
| `show ip protocols` | Confirms EIGRP process-level state | Correct AS, RID, K-values, network statements, passive interfaces, and AD values |
| `show ip eigrp topology` | Confirms EIGRP knows the expected prefixes | Prefixes appear in passive state with successors |
| `show ip route eigrp` | Confirms EIGRP routes entered the RIB | Remote internal routes appear as `D` |
| `show ip route <REMOTE_PREFIX>` | Confirms exact next-hop decision for a route | Route points to the expected EIGRP neighbor/interface |
| `ping <REMOTE_IP>` | Confirms reachability | Ping succeeds |
| `traceroute <REMOTE_IP>` | Confirms forwarding path | Hops follow expected EIGRP path |
# EIGRP_Foundation_Classic_Mode_Rollback
! Remove the entire classic EIGRP process
configure terminal
 no router eigrp <AS_NUMBER>
end
! Verify EIGRP is removed
show running-config | section router eigrp
show ip eigrp neighbors
show ip route eigrp
! Expected result:
! No EIGRP router process remains.
! No EIGRP neighbors remain.
! No EIGRP-learned routes remain in the routing table.
write memory
# EIGRP_Foundation_Classic_Mode_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| No EIGRP neighbors form | Wrong AS number | `show running-config | section router eigrp` | Use the same `router eigrp <AS>` on routers that should peer |
| No EIGRP neighbors form | Interface not matched by network statement | `show ip eigrp interfaces` | Add the correct `network <interface-ip> 0.0.0.0` or subnet wildcard |
| No EIGRP neighbors form | Interface is passive | `show ip protocols` | Use `no passive-interface <interface>` under `router eigrp <AS>` |
| No EIGRP neighbors form | Interface down or wrong IP addressing | `show ip interface brief` | Fix interface state, cabling, CML link, or IP addressing |
| Neighbor forms but routes are missing | Local network not advertised | `show ip eigrp topology` | Add the missing connected interface with a `network` statement |
| Neighbor forms but route not in RIB | Better route from another protocol exists | `show ip route <prefix>` | Compare administrative distance and metric |
| Routes appear but ping fails | Return route missing | `show ip route <source-prefix>` on remote router | Ensure both directions are advertised |
| Routes appear but forwarding path is wrong | Unexpected metric or interface selection | `show ip route <prefix>` and `show ip eigrp topology <prefix>` | Correct bandwidth/delay only if intentional |
| EIGRP routes flap | Link instability or neighbor reset | `show logging | include EIGRP|DUAL` | Fix interface/link stability first |
| EIGRP enabled on too many interfaces | Broad wildcard or classful network statement | `show ip eigrp interfaces` | Replace broad `network` statements with exact host wildcard entries |
| Remote prefixes appear as `D EX` unexpectedly | Redistribution is involved | `show ip route eigrp` and `show ip protocols` | Remove or correct redistribution if this is supposed to be pure internal EIGRP |
# Index
# Source_Basis
# EIGRP_Foundation_Classic_Mode_Mental_Model
# EIGRP_Foundation_Classic_Mode_Configuration_Checklist
# EIGRP_Foundation_Classic_Mode_Skeleton
# EIGRP_Foundation_Classic_Mode_Verification_Commands
# EIGRP_Foundation_Classic_Mode_Rollback
# EIGRP_Foundation_Classic_Mode_Failure_Checks

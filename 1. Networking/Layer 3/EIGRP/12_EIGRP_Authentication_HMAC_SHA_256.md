EIGRP_Authentication_HMAC_SHA_256.md

EIGRP_Authentication_HMAC_SHA_256

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 3: EIGRP for IP, Chapter 15: Secure EIGRP Operation | Points EIGRP security and authentication topics to `All_combined_part3.md` |
| `All_combined_part3.md` | Book 3, Chapter 15: Secure EIGRP Operation | Supports EIGRP authentication concepts, key validation, adjacency protection, and authentication troubleshooting |
| `All_combined_part3.md` | CCIE R&S Foundations EIGRP Authentication Task | Supports the distinction that classic EIGRP uses MD5 key-chain authentication, while HMAC-SHA-256 is configured in named mode |
| `All_combined_part3.md` | HMAC-SHA-256 EIGRP task examples | Supports syntax: `router eigrp <name>`, `address-family ipv4 unicast as <AS>`, `af-interface <interface>`, and `authentication mode hmac-sha-256 <password>` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | EIGRP Authentication | Supports troubleshooting mismatched authentication, missing authentication, key mismatch behavior, and verification with EIGRP interface detail commands |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Named EIGRP Interface Verification | Supports `show eigrp address-family ipv4 interfaces detail` for named-mode interface authentication verification |
# EIGRP_Authentication_HMAC_SHA_256_Mental_Model
| Concept | Operational Meaning |
|---|---|
| EIGRP authentication | Controls whether EIGRP packets from a neighbor are accepted on a link |
| Per-interface authentication | Authentication is applied on the EIGRP interface, not globally for every link automatically |
| HMAC-SHA-256 | Stronger EIGRP authentication method used with named EIGRP mode |
| Classic EIGRP limitation | Classic EIGRP normally uses MD5 key-chain authentication, not HMAC-SHA-256 |
| Named mode requirement | HMAC-SHA-256 requires named EIGRP address-family and `af-interface` configuration |
| Shared secret | Both neighbors on the link must use the same HMAC-SHA-256 password |
| Interface scope | Authentication must be configured on the exact interface where the EIGRP neighbor forms |
| AS consistency | The named-mode address-family AS number must match between neighbors |
| Process name | The named EIGRP process name is local and does not have to match between routers |
| Neighbor impact | Adjacency drops until both sides use matching authentication settings |
| Missing authentication | One side authenticates and the other does not, so EIGRP packets are rejected |
| Invalid authentication | Both sides authenticate, but the password or method does not match |
| Special character caveat | Passwords containing `?` may require escape-key handling in IOS CLI |
| Verification anchor | The truth source is EIGRP interface detail output showing `Authentication mode is HMAC-SHA-256` |
| Operational warning | Do not enable this on one side during production without a maintenance window because the adjacency will reset |
# EIGRP_Authentication_HMAC_SHA_256_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm EIGRP-facing interfaces are operational | Both routers | `show ip interface brief` | Neighbor-facing interfaces are `up/up` with correct IPv4 addresses |
| 2 | Confirm baseline EIGRP neighbor state before authentication | Both routers | `show ip eigrp neighbors` | Expected neighbor is present before the change |
| 3 | Confirm current EIGRP mode and process | Both routers | `show running-config | section router eigrp` | Current classic or named-mode EIGRP configuration is visible |
| 4 | Confirm current protocol details | Both routers | `show ip protocols` | AS number, router ID, network statements, passive interfaces, and metric weights are documented |
| 5 | Confirm which interface forms the neighbor relationship | Both routers | `show ip eigrp neighbors` | Neighbor appears on the expected local interface |
| 6 | Verify current authentication state | Both routers | `show ip eigrp interfaces detail <INTERFACE> | include Auth` | Authentication is either not set or shows the current method |
| 7 | Decide whether migration to named mode is required | Both routers | Planning step | If currently classic mode, migrate to named mode before HMAC-SHA-256 |
| 8 | Capture the existing classic EIGRP process before migration | Both routers | `show running-config | section router eigrp` | Existing AS, networks, passive settings, and redistribution settings are documented |
| 9 | Remove classic EIGRP only if migration is required | Both routers | `no router eigrp <AS_NUMBER>` | Classic EIGRP process is removed |
| 10 | Create the named EIGRP process | Both routers | `router eigrp <PROCESS_NAME>` | Router enters named EIGRP configuration mode |
| 11 | Create the IPv4 unicast address family and bind it to the real AS number | Both routers | `address-family ipv4 unicast autonomous-system <AS_NUMBER>` | Router enters address-family mode |
| 12 | Rebuild the EIGRP router ID if used previously | Both routers | `eigrp router-id <ROUTER_ID>` | Stable router ID is configured |
| 13 | Rebuild EIGRP network statements | Both routers | `network <INTERFACE_IP> 0.0.0.0` | Intended interfaces are added to the EIGRP address family |
| 14 | Rebuild passive-interface baseline if used | Both routers | `af-interface default` then `passive-interface` | All EIGRP-enabled interfaces are passive by default |
| 15 | Re-enable EIGRP neighbor formation on the routed link | Both routers | `af-interface <NEIGHBOR_INTERFACE>` then `no passive-interface` | Neighbor-facing interface can send and receive EIGRP hellos |
| 16 | Configure HMAC-SHA-256 authentication on the neighbor-facing interface | Both routers | `authentication mode hmac-sha-256 <SHARED_SECRET>` | EIGRP authentication is applied to the selected interface |
| 17 | Exit interface submode | Both routers | `exit-af-interface` | Router returns to address-family mode |
| 18 | Exit address-family mode | Both routers | `exit-address-family` | Router returns to named EIGRP process mode |
| 19 | Exit configuration mode | Both routers | `end` | Router returns to privileged EXEC mode |
| 20 | Verify the named-mode configuration hierarchy | Both routers | `show running-config | section router eigrp` | Address-family, `af-interface`, and HMAC-SHA-256 authentication appear under named EIGRP |
| 21 | Verify authentication state on the interface | Both routers | `show ip eigrp interfaces detail <INTERFACE> | include Auth` | Output shows `Authentication mode is HMAC-SHA-256` |
| 22 | Verify named-mode interface detail if supported | Both routers | `show eigrp address-family ipv4 interfaces detail <INTERFACE> | include Auth` | Output confirms HMAC-SHA-256 authentication |
| 23 | Verify neighbor adjacency reforms | Both routers | `show ip eigrp neighbors` | Neighbor returns with stable uptime and `Q Cnt` of `0` |
| 24 | Verify EIGRP routes return to the routing table | Both routers | `show ip route eigrp` | Expected `D` routes are installed |
| 25 | Verify topology table stability | Both routers | `show ip eigrp topology` | Expected prefixes are passive with valid successors |
| 26 | Check logs for authentication failures | Both routers | `show logging | include EIGRP|auth|Auth|NBRCHANGE` | No repeated missing or invalid authentication messages remain |
| 27 | Test end-to-end reachability | Edge router or host | `ping <REMOTE_IP>` | Ping succeeds |
| 28 | Verify forwarding path | Edge router or host | `traceroute <REMOTE_IP>` | Path follows expected EIGRP next hops |
| 29 | Save the configuration after both neighbors are stable | Both routers | `write memory` | Configuration is saved |
# EIGRP_Authentication_HMAC_SHA_256_Skeleton
! Baseline verification
show ip interface brief
show ip eigrp neighbors
show ip protocols
show running-config | section router eigrp
show ip eigrp interfaces detail <INTERFACE> | include Auth
! If currently classic mode and HMAC-SHA-256 is required, migrate to named mode
configure terminal
 no router eigrp <AS_NUMBER>
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   eigrp router-id <ROUTER_ID>
   network <NEIGHBOR_INTERFACE_IP> 0.0.0.0
   network <LOCAL_PREFIX_INTERFACE_IP> 0.0.0.0
   af-interface default
    passive-interface
   exit-af-interface
   af-interface <NEIGHBOR_INTERFACE>
    no passive-interface
    authentication mode hmac-sha-256 <SHARED_SECRET>
   exit-af-interface
  exit-address-family
 end
! If already using named mode, apply only the interface authentication
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   af-interface <NEIGHBOR_INTERFACE>
    authentication mode hmac-sha-256 <SHARED_SECRET>
   exit-af-interface
  exit-address-family
 end
! Verification
show running-config | section router eigrp
show ip eigrp interfaces detail <INTERFACE> | include Auth
show eigrp address-family ipv4 interfaces detail <INTERFACE> | include Auth
show ip eigrp neighbors
show ip protocols
show ip route eigrp
show ip eigrp topology
show logging | include EIGRP|auth|Auth|NBRCHANGE
ping <REMOTE_IP>
traceroute <REMOTE_IP>
! Save
write memory
# EIGRP_Authentication_HMAC_SHA_256_Verification_Commands
| Command | Purpose | Healthy Output |
|---|---|---|
| `show ip interface brief` | Confirms interface state before authentication | Neighbor-facing interfaces are `up/up` |
| `show ip eigrp neighbors` | Confirms adjacency state | Neighbor appears with stable uptime and `Q Cnt` of `0` |
| `show ip protocols` | Confirms EIGRP process state | Correct AS, router ID, network statements, passive interfaces, and K-values |
| `show running-config | section router eigrp` | Confirms named-mode hierarchy and authentication placement | Shows `router eigrp <PROCESS_NAME>`, address-family, `af-interface`, and `authentication mode hmac-sha-256` |
| `show ip eigrp interfaces detail <INTERFACE>` | Confirms interface-level EIGRP settings | Interface shows authentication mode as HMAC-SHA-256 |
| `show ip eigrp interfaces detail <INTERFACE> | include Auth` | Quickly checks authentication state | Output shows `Authentication mode is HMAC-SHA-256` |
| `show eigrp address-family ipv4 interfaces detail <INTERFACE>` | Verifies named-mode interface details where supported | Interface detail shows HMAC-SHA-256 authentication |
| `show ip route eigrp` | Confirms EIGRP routes returned after authentication | Expected internal EIGRP routes appear as `D` |
| `show ip eigrp topology` | Confirms EIGRP topology stability | Expected prefixes are passive with valid successors |
| `show logging | include EIGRP|auth|Auth|NBRCHANGE` | Checks authentication and neighbor reset events | No repeated missing authentication or invalid authentication messages |
| `debug eigrp packets` | Lab-only troubleshooting of rejected EIGRP packets | Shows missing or invalid authentication if mismatch exists |
| `undebug all` | Stops debugging | All debugging is disabled |
| `ping <REMOTE_IP>` | Confirms data-plane reachability | Ping succeeds |
| `traceroute <REMOTE_IP>` | Confirms forwarding path | Path follows expected EIGRP route |
# EIGRP_Authentication_HMAC_SHA_256_Rollback
! Remove HMAC-SHA-256 authentication from named-mode EIGRP interface
configure terminal
 router eigrp <PROCESS_NAME>
  address-family ipv4 unicast autonomous-system <AS_NUMBER>
   af-interface <NEIGHBOR_INTERFACE>
    no authentication mode
   exit-af-interface
  exit-address-family
 end
! Optional: if this was a migration lab and you must return to classic EIGRP
configure terminal
 no router eigrp <PROCESS_NAME>
 router eigrp <AS_NUMBER>
  eigrp router-id <ROUTER_ID>
  no auto-summary
  network <NEIGHBOR_INTERFACE_IP> 0.0.0.0
  network <LOCAL_PREFIX_INTERFACE_IP> 0.0.0.0
  passive-interface default
  no passive-interface <NEIGHBOR_INTERFACE>
 end
! Verify rollback
show running-config | section router eigrp
show ip eigrp interfaces detail <INTERFACE> | include Auth
show ip eigrp neighbors
show ip protocols
show ip route eigrp
show ip eigrp topology
ping <REMOTE_IP>
traceroute <REMOTE_IP>
! Expected result:
! HMAC-SHA-256 authentication is removed.
! Neighbor adjacency reforms only if both sides use matching authentication state.
! EIGRP routes return after adjacency is stable.
write memory
# EIGRP_Authentication_HMAC_SHA_256_Failure_Checks
| Symptom | Likely Cause | Check Command | Fix |
|---|---|---|---|
| EIGRP neighbor drops immediately after authentication | Only one side has authentication configured | `show ip eigrp interfaces detail <INTERFACE> | include Auth` | Configure the same HMAC-SHA-256 authentication on both neighbors |
| EIGRP neighbor does not form | Shared secret mismatch | `show running-config | section router eigrp` and `debug eigrp packets` | Correct the password on both sides |
| EIGRP neighbor does not form | Authentication applied to wrong interface | `show ip eigrp neighbors` and `show ip eigrp interfaces detail <INTERFACE>` | Apply authentication under the actual neighbor-facing `af-interface` |
| EIGRP neighbor does not form | One side is classic mode and the other is named mode with HMAC-SHA-256 | `show running-config | section router eigrp` | Use named mode on both sides for HMAC-SHA-256 |
| HMAC-SHA-256 command is unavailable | Router is in classic EIGRP mode | `router eigrp <AS_NUMBER>` then `?` | Migrate to named EIGRP mode and configure authentication under `af-interface` |
| HMAC-SHA-256 command still unavailable | Platform or image does not support the feature | `authentication mode ?` under `af-interface` | Use supported EIGRP authentication method for that platform |
| Authentication appears configured but neighbor still fails | AS number mismatch under address-family | `show ip protocols` | Match `address-family ipv4 unicast autonomous-system <AS_NUMBER>` across neighbors |
| Authentication appears configured but no hello exchange occurs | Interface is passive | `show ip protocols` | Configure `no passive-interface` under the specific `af-interface` |
| Authentication configured but EIGRP is not active on the link | Missing network statement | `show ip eigrp interfaces` | Add `network <INTERFACE_IP> 0.0.0.0` under the address family |
| Password containing `?` cannot be typed normally | IOS interprets `?` as help | CLI behavior during command entry | Use IOS escape handling, such as pressing `Esc` then `q` before entering `?` |
| Routes do not return after neighbor reforms | Migration missed old network statements or passive settings | `show ip protocols` and `show running-config | section router eigrp` | Rebuild missing named-mode network and `af-interface` settings |
| Neighbor forms but routes are missing | Local route advertisement was not rebuilt after classic-to-named migration | `show ip eigrp topology` and `show ip route connected` | Add missing network statements or redistribution under named mode |
| Logs show missing authentication | Peer has no authentication configured | `show logging | include auth|Auth|EIGRP` | Configure HMAC-SHA-256 on the peer interface |
| Logs show invalid authentication | Peer password or authentication method is wrong | `debug eigrp packets` in lab only | Match method and shared secret exactly |
| Debug output overwhelms the router | Debugging left enabled | `show debugging` | Run `undebug all` |
| Ping fails even though EIGRP neighbor is up | Data-plane or return-route problem | `show ip route <REMOTE_PREFIX>` and remote `show ip route <SOURCE_PREFIX>` | Fix routing, ACL, NAT, or interface issue separately |
| Authentication configured on all links by mistake | `af-interface default` used incorrectly | `show running-config | section router eigrp` | Remove default authentication and apply it only under intended neighbor-facing interfaces |
# Index
# Source_Basis
# EIGRP_Authentication_HMAC_SHA_256_Mental_Model
# EIGRP_Authentication_HMAC_SHA_256_Configuration_Checklist
# EIGRP_Authentication_HMAC_SHA_256_Skeleton
# EIGRP_Authentication_HMAC_SHA_256_Verification_Commands
# EIGRP_Authentication_HMAC_SHA_256_Rollback
# EIGRP_Authentication_HMAC_SHA_256_Failure_Checks

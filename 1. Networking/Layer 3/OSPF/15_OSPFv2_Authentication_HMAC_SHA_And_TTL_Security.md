I used the local project source first, then Cisco IOS XE docs for the missing HMAC-SHA and TTL-security syntax: ip ospf authentication key-chain <name>, cryptographic-algorithm hmac-sha-256, ttl-security all-interfaces, and ip ospf ttl-security.  

OSPFv2_Authentication_HMAC_SHA_And_TTL_Security.md

OSPFv2_Authentication_HMAC_SHA_And_TTL_Security

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | OSPF topic map | Points enterprise OSPF to `All_combined_part3.md` Book 6 and IOS XE OSPF material as primary project sources |
| `All_combined_part3.md` | Book 6, Chapter 8 OSPF authentication tasks | Supports OSPFv2 area/interface authentication model, neighbor mismatch behavior, and OSPF authentication verification |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 6, OSPF Authentication | Supports OSPF authentication concepts, plaintext versus MD5 behavior, area versus interface authentication, and `show ip ospf interface` verification |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 8, Mismatched Authentication Information | Supports troubleshooting mismatched authentication type, key ID, key string, and neighbor adjacency failures |
| Cisco IOS XE IP Routing Configuration Guide | OSPFv2 Cryptographic Authentication | Provides HMAC-SHA key-chain syntax: `key chain`, `key`, `key-string`, `cryptographic-algorithm hmac-sha-256`, and `ip ospf authentication key-chain <name>` |
| Cisco IOS XE IP Routing Configuration Guide | OSPF TTL Security Check | Provides TTL security syntax: `ttl-security all-interfaces [hops <hop-count>]`, `ip ospf ttl-security [hops <hop-count> | disable]`, and verification with `show ip ospf interface`, `show ip ospf neighbor`, and `show ip ospf traffic` |
| Related labs | `ospf-hmac-sha-authentication-final`, `ospf-ttl-security-check-final` | Main labs for OSPFv2 HMAC-SHA authentication and OSPF TTL security check |
# OSPFv2_Authentication_HMAC_SHA_And_TTL_Security_Mental_Model
| Concept | Operational Meaning |
|---|---|
| OSPF authentication | Prevents unauthorized routers from forming OSPF adjacencies or injecting invalid OSPF packets |
| Null authentication | No authentication. OSPF packets are accepted without a shared key |
| Plaintext authentication | Weak legacy authentication. Password can be exposed on the wire |
| MD5 authentication | Legacy cryptographic authentication using `ip ospf message-digest-key <key-id> md5 <password>` |
| HMAC-SHA authentication | Stronger OSPFv2 cryptographic authentication using a key chain and HMAC-SHA algorithm |
| RFC 5709 model | OSPFv2 HMAC-SHA extends classic OSPFv2 cryptographic authentication beyond MD5 |
| Key chain | Container holding one or more authentication keys, key strings, algorithms, and lifetimes |
| Key ID | Numeric identifier for a key. OSPFv2 key IDs are in the 1 to 255 range |
| Security Association | Combination of key ID, key string, cryptographic algorithm, and lifetime |
| Key string | Shared secret. Must match between OSPF neighbors |
| Cryptographic algorithm | Defines the hash algorithm, such as `hmac-sha-256` |
| Send lifetime | Time range during which a key is used to send authenticated packets |
| Accept lifetime | Time range during which a key is accepted for received packets |
| Interface attachment | HMAC-SHA authentication is attached under the OSPF-enabled interface with `ip ospf authentication key-chain <name>` |
| Key-chain precedence | If OSPFv2 uses a key chain, older MD5 keys configured with `ip ospf message-digest-key` are ignored |
| TTL security check | Lightweight OSPF neighbor protection that sends OSPF packets with TTL 255 and drops received packets below the allowed TTL threshold |
| Direct neighbor protection | With default TTL security, directly connected OSPF neighbors should be received with TTL 255 |
| Hop-count logic | `hops <value>` defines the maximum number of hops an OSPF packet may have crossed before being accepted |
| Global TTL security | `ttl-security all-interfaces` applies TTL security to normal OSPF interfaces in the process |
| Interface TTL security | `ip ospf ttl-security` applies TTL security on one interface |
| Interface disable override | `ip ospf ttl-security disable` disables TTL security on an interface when global TTL security is enabled |
| Safe transition | `ip ospf ttl-security hops 254` is a loose transition setting that allows almost any received TTL while setting outgoing TTL to 255 |
| Scope warning | `ttl-security all-interfaces` does not cover virtual links or sham links. Those require separate TTL-security configuration under the virtual-link or sham-link command |
| Failure mode | Authentication mismatch or TTL-security mismatch can leave neighbors down, stuck in INIT, or repeatedly flapping |
# OSPFv2_Authentication_HMAC_SHA_And_TTL_Security_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify OSPF neighbors that need protection | Design step | `<router-a> <interface> to <router-b> <interface>` | Neighbor-facing OSPF links are known |
| 2 | Confirm OSPF baseline adjacency before adding protection | Both neighbors | `show ip ospf neighbor` | Neighbor is currently `FULL` |
| 3 | Confirm OSPF interface participation | Both neighbors | `show ip ospf interface brief` | Intended interfaces are in the correct area |
| 4 | Confirm current authentication state | Both neighbors | `show ip ospf interface <interface-id> | include authentication|Cryptographic|key` | Current authentication type is known |
| 5 | Confirm current TTL-security state | Both neighbors | `show ip ospf interface <interface-id> | include TTL|ttl|security` | Current TTL-security behavior is known |
| 6 | Confirm no stale MD5 key will confuse review | Both neighbors | `show running-config interface <interface-id> | include ospf.*message-digest|authentication` | Existing MD5 or plaintext commands are identified |
| 7 | Enter global configuration mode | Both neighbors | `configure terminal` | Router enters global configuration mode |
| 8 | Create the OSPF key chain | Both neighbors | `key chain <KEY_CHAIN_NAME>` | Key-chain configuration mode opens |
| 9 | Create matching key ID | Both neighbors | `key <KEY_ID>` | Key-chain key configuration mode opens |
| 10 | Configure matching key string | Both neighbors | `key-string <SHARED_SECRET>` | Shared secret is configured |
| 11 | Configure HMAC-SHA-256 algorithm | Both neighbors | `cryptographic-algorithm hmac-sha-256` | Key uses HMAC-SHA-256 |
| 12 | Configure send lifetime if key rollover is required | Both neighbors | `send-lifetime local <hh:mm:ss> <day> <month> <year> infinite` | Key is valid for sending during the intended lifetime |
| 13 | Configure accept lifetime if key rollover is required | Both neighbors | `accept-lifetime local <hh:mm:ss> <day> <month> <year> infinite` | Key is valid for received packets during the intended lifetime |
| 14 | Exit key-chain configuration | Both neighbors | `exit` until global configuration mode | Router returns to global configuration mode |
| 15 | Enter the OSPF neighbor-facing interface | Both neighbors | `interface <interface-id>` | Interface configuration mode opens |
| 16 | Attach HMAC-SHA key chain to OSPF interface | Both neighbors | `ip ospf authentication key-chain <KEY_CHAIN_NAME>` | Interface uses the key chain for OSPFv2 cryptographic authentication |
| 17 | Remove old plaintext authentication if present | Both neighbors | `no ip ospf authentication-key <password>` | Plaintext key is removed |
| 18 | Remove old interface MD5 mode if replacing MD5 | Both neighbors | `no ip ospf authentication message-digest` | Interface MD5 mode is removed if no longer used |
| 19 | Remove old MD5 key if replacing MD5 | Both neighbors | `no ip ospf message-digest-key <key-id> md5 <password>` | Legacy MD5 key is removed |
| 20 | Keep the OSPF interface active | Both neighbors | `no shutdown` | Interface remains up |
| 21 | Exit configuration mode | Both neighbors | `end` | Router returns to privileged EXEC mode |
| 22 | Save configuration | Both neighbors | `write memory` | Configuration is saved |
| 23 | Verify key-chain contents | Both neighbors | `show key chain <KEY_CHAIN_NAME>` | Key ID, key string presence, and lifetimes are visible |
| 24 | Verify interface authentication | Both neighbors | `show ip ospf interface <interface-id>` | Output shows cryptographic authentication and HMAC-SHA key-chain sending SA |
| 25 | Verify OSPF neighbor recovered | Both neighbors | `show ip ospf neighbor` | Neighbor returns to `FULL` |
| 26 | Enable TTL security globally if all normal OSPF interfaces should use it | All OSPF routers | `configure terminal` then `router ospf <process-id>` then `ttl-security all-interfaces` | TTL security applies to all normal OSPF interfaces |
| 27 | Enable TTL security globally with loose transition behavior | All OSPF routers during migration | `ttl-security all-interfaces hops 254` | Outgoing OSPF packets use TTL 255, while received packets are loosely accepted during transition |
| 28 | Enable TTL security on one interface only | Both neighbors | `interface <interface-id>` then `ip ospf ttl-security` | Interface enforces TTL security with default one-hop behavior |
| 29 | Enable TTL security on one interface with custom hop tolerance | Both neighbors | `ip ospf ttl-security hops <hop-count>` | Interface accepts OSPF packets within configured hop threshold |
| 30 | Disable TTL security on one interface when global TTL security is enabled | Selected router | `interface <interface-id>` then `ip ospf ttl-security disable` | Interface is exempt from global TTL security |
| 31 | Exit configuration mode | All changed routers | `end` | Router returns to privileged EXEC mode |
| 32 | Save configuration | All changed routers | `write memory` | Configuration is saved |
| 33 | Verify TTL-security interface state | All changed routers | `show ip ospf interface <interface-id>` | Interface shows TTL-security behavior |
| 34 | Verify OSPF neighbor state after TTL security | Both neighbors | `show ip ospf neighbor` | Neighbor remains `FULL` |
| 35 | Check TTL-security failures | Both neighbors | `show ip ospf traffic` | TTL-security failure counter is zero or not increasing |
| 36 | Check adjacency logs if neighbor changed state | Both neighbors | `show logging | include OSPF|AUTH|TTL|ADJCHG` | Logs confirm whether authentication or TTL caused the issue |
| 37 | Use debug only during lab troubleshooting | Lab routers | `debug ip ospf adj` | Debug shows authentication or TTL-related adjacency drops |
| 38 | Disable debug after validation | Lab routers | `undebug all` | Debugging is stopped |
| 39 | Confirm protected OSPF routes still install | Downstream routers | `show ip route ospf` | Expected OSPF routes remain installed |
| 40 | Confirm end-to-end reachability | Test routers | `ping <remote-loopback-ip>` | Traffic still works across the OSPF domain |
# OSPFv2_Authentication_HMAC_SHA_And_TTL_Security_Skeleton
! =========================================================
! OSPFv2 HMAC-SHA-256 authentication with key chain
! Configure matching key chain and interface attachment on both neighbors.
! =========================================================
configure terminal
key chain <KEY_CHAIN_NAME>
 key <KEY_ID>
  key-string <SHARED_SECRET>
  cryptographic-algorithm hmac-sha-256
  send-lifetime local <HH:MM:SS> <DAY> <MONTH> <YEAR> infinite
  accept-lifetime local <HH:MM:SS> <DAY> <MONTH> <YEAR> infinite
 exit
exit
interface <OSPF_INTERFACE>
 ip ospf authentication key-chain <KEY_CHAIN_NAME>
 no shutdown
exit
end
write memory
! =========================================================
! Minimal OSPFv2 HMAC-SHA-256 lab pattern
! Use when key lifetime testing is not required.
! =========================================================
configure terminal
key chain OSPF_HMAC
 key 1
  key-string <SHARED_SECRET>
  cryptographic-algorithm hmac-sha-256
 exit
exit
interface <OSPF_INTERFACE>
 ip ospf authentication key-chain OSPF_HMAC
exit
end
write memory
! =========================================================
! Replace legacy MD5 with HMAC-SHA key-chain authentication
! Confirm platform support before using in mixed-code labs.
! =========================================================
configure terminal
key chain OSPF_HMAC
 key 1
  key-string <SHARED_SECRET>
  cryptographic-algorithm hmac-sha-256
 exit
exit
interface <OSPF_INTERFACE>
 no ip ospf authentication message-digest
 no ip ospf message-digest-key <OLD_KEY_ID> md5 <OLD_MD5_SECRET>
 ip ospf authentication key-chain OSPF_HMAC
exit
end
write memory
! =========================================================
! TTL security on all normal OSPF interfaces
! Does not apply to virtual links or sham links.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 ttl-security all-interfaces
end
write memory
! =========================================================
! TTL security on one OSPF interface
! Default behavior expects directly connected neighbor packets.
! =========================================================
configure terminal
interface <OSPF_INTERFACE>
 ip ospf ttl-security
end
write memory
! =========================================================
! TTL security transition mode
! Start loose, configure both sides, then tighten.
! =========================================================
! Temporary loose setting:
configure terminal
interface <OSPF_INTERFACE>
 ip ospf ttl-security hops 254
end
write memory
! Final strict setting:
configure terminal
interface <OSPF_INTERFACE>
 ip ospf ttl-security
end
write memory
! =========================================================
! Exempt one interface from global TTL security
! Useful when ttl-security all-interfaces is configured.
! =========================================================
configure terminal
interface <OSPF_INTERFACE>
 ip ospf ttl-security disable
end
write memory
# OSPFv2_Authentication_HMAC_SHA_And_TTL_Security_Verification_Commands
show running-config | section key chain
show running-config interface <interface-id>
show running-config | section router ospf
show key chain
show key chain <KEY_CHAIN_NAME>
show ip ospf
show ip ospf interface brief
show ip ospf interface <interface-id>
show ip ospf interface <interface-id> | include authentication|Cryptographic|Sending SA|TTL|security
show ip ospf neighbor
show ip ospf neighbor detail
show ip ospf traffic
show ip route ospf
show logging | include OSPF|AUTH|authentication|TTL|ADJCHG
debug ip ospf adj
show debugging
undebug all
ping <neighbor-interface-ip>
ping <remote-loopback-ip>
traceroute <remote-loopback-ip>
# OSPFv2_Authentication_HMAC_SHA_And_TTL_Security_Rollback
! =========================================================
! Remove HMAC-SHA key-chain authentication from one interface
! =========================================================
configure terminal
interface <OSPF_INTERFACE>
 no ip ospf authentication key-chain <KEY_CHAIN_NAME>
end
write memory
! =========================================================
! Remove key chain after detaching it from all OSPF interfaces
! =========================================================
configure terminal
no key chain <KEY_CHAIN_NAME>
end
write memory
! =========================================================
! Remove one key from a key chain
! =========================================================
configure terminal
key chain <KEY_CHAIN_NAME>
 no key <KEY_ID>
end
write memory
! =========================================================
! Remove global TTL security from OSPF process
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no ttl-security all-interfaces
end
write memory
! =========================================================
! Remove interface TTL security
! =========================================================
configure terminal
interface <OSPF_INTERFACE>
 no ip ospf ttl-security
end
write memory
! =========================================================
! Remove interface TTL-security disable override
! =========================================================
configure terminal
interface <OSPF_INTERFACE>
 no ip ospf ttl-security disable
end
write memory
! =========================================================
! Restore legacy MD5 only if the lab explicitly requires it
! =========================================================
configure terminal
interface <OSPF_INTERFACE>
 ip ospf authentication message-digest
 ip ospf message-digest-key <KEY_ID> md5 <MD5_SECRET>
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
# OSPFv2_Authentication_HMAC_SHA_And_TTL_Security_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| Neighbor drops immediately after HMAC-SHA is applied | Key-chain name mismatch | `show running-config interface <interface-id>` | Use the same key-chain name attachment on both sides or attach the correct local key chain |
| Neighbor drops after authentication | Key ID mismatch | `show key chain <KEY_CHAIN_NAME>` | Use matching key IDs on both sides |
| Neighbor drops after authentication | Key string mismatch | `show running-config | section key chain` | Reconfigure the same shared secret on both routers |
| Neighbor drops after authentication | Algorithm mismatch | `show key chain <KEY_CHAIN_NAME>` | Use matching `cryptographic-algorithm hmac-sha-256` on both sides |
| Neighbor drops after authentication | Key lifetime invalid or not active | `show key chain <KEY_CHAIN_NAME>` | Correct `send-lifetime` and `accept-lifetime` or remove lifetime constraints in lab |
| Neighbor stays down with old MD5 config visible | Key-chain authentication causes old MD5 interface keys to be ignored | `show running-config interface <interface-id>` | Remove old MD5 commands and verify key-chain authentication is intended |
| Neighbor fails after replacing plaintext | Plaintext command remains on one side | `show running-config interface <interface-id>` | Remove plaintext authentication and configure HMAC-SHA key chain on both sides |
| `show ip ospf` says area has no authentication | Authentication is interface-based | `show ip ospf interface <interface-id>` | Verify at the interface level |
| HMAC-SHA command is rejected | Platform or software does not support OSPFv2 HMAC-SHA | `key chain <name>` then `cryptographic-algorithm ?` | Use a supported IOS XE image or fall back to MD5 only for older lab images |
| Neighbor stuck in INIT after TTL security | TTL security enabled on one side only | `show ip ospf neighbor`, `show ip ospf interface <interface-id>` | Enable TTL security on both neighbors or temporarily use `ip ospf ttl-security hops 254` during transition |
| Neighbor drops after global TTL security | One interface cannot meet the TTL threshold | `show ip ospf traffic`, `show ip ospf interface <interface-id>` | Use interface-specific `ip ospf ttl-security hops <count>` or `ip ospf ttl-security disable` |
| TTL-security failure counter increases | Received OSPF packets arrive below the allowed TTL threshold | `show ip ospf traffic` | Confirm direct adjacency or increase allowed hops only if the design requires it |
| TTL security does not affect virtual link | Global TTL security does not apply to virtual links | `show running-config | section router ospf` | Configure TTL security separately on the virtual-link command if required |
| TTL security does not affect sham link | Global TTL security does not apply to sham links | `show running-config | section router ospf` | Configure TTL security separately on the sham-link command if required |
| Neighbor still down after fixing auth and TTL | Another OSPF mismatch exists | `show ip ospf interface <interface-id>`, `show ip ospf neighbor detail` | Check area ID, timers, network type, MTU, passive interface, and subnet mask |
| Debug shows mismatched authentication type | One side uses null, plaintext, MD5, or HMAC-SHA while the other uses a different type | `debug ip ospf adj` | Make both sides use the same authentication method |
| Debug shows packet dropped due to TTL | TTL-security threshold rejects received packet | `debug ip ospf adj`, `show ip ospf traffic` | Fix TTL-security configuration or neighbor topology |
| OSPF routes disappear after protection | Neighbor adjacency failed | `show ip ospf neighbor`, `show ip route ospf` | Restore `FULL` adjacency first |
| Ping to remote prefix fails but neighbor is FULL | Not an authentication or TTL-security issue | `show ip route <remote-prefix>`, `traceroute <remote-prefix>` | Troubleshoot routing, return path, ACLs, or redistribution |
| Key rollover causes temporary outage | Send and accept lifetimes do not overlap | `show key chain <KEY_CHAIN_NAME>` | Configure overlapping accept and send lifetimes before changing active keys |
| Wrong router forms adjacency before security is applied | Authentication and TTL security were not enabled on the exposed segment | `show ip ospf neighbor` | Apply HMAC-SHA and TTL security to all neighbor-facing interfaces on that segment |
##### Source_Basis
# OSPFv2_Authentication_HMAC_SHA_And_TTL_Security_Mental_Model
# OSPFv2_Authentication_HMAC_SHA_And_TTL_Security_Configuration_Checklist
# OSPFv2_Authentication_HMAC_SHA_And_TTL_Security_Skeleton
# OSPFv2_Authentication_HMAC_SHA_And_TTL_Security_Verification_Commands
# OSPFv2_Authentication_HMAC_SHA_And_TTL_Security_Rollback
# OSPFv2_Authentication_HMAC_SHA_And_TTL_Security_Failure_Checks
# index of each title throughout note, not in table format


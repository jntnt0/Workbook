OSPFv3_Keychain_Authentication_And_ESP_Encryption.md

OSPFv3_Keychain_Authentication_And_ESP_Encryption

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | OSPF topic map | Points OSPF and OSPFv3 topics to `All_combined_part3.md`, `Ios-xs_combined_epubs.md`, and ENARSI material |
| `Ios-xs_combined_epubs.md` | OSPFv3 Authentication Trailer | Supports OSPFv3 key-chain authentication trailer model, interface-level authentication, area-level authentication, virtual-link authentication, sham-link authentication, `authentication mode deployment`, and `authentication mode normal` |
| `Ios-xs_combined_epubs.md` | Configuration examples for OSPFv3 Authentication Trailer | Provides syntax for `ospfv3 <process-id> ipv6 authentication key-chain <name>`, `area <area-id> authentication key-chain <name>`, `key chain`, `key`, `key-string`, and `cryptographic-algorithm hmac-sha-256` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 9, OSPFv3 Authentication | Supports OSPFv3 IPsec AH and ESP model, interface-level versus area-level precedence, manual SPI requirement, and `show ospfv3 interface` verification |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 9, OSPFv3 Interface Authentication and Encryption | Provides ESP syntax with `ospfv3 encryption ipsec spi <spi> esp <encryption-algorithm> <encryption-key> <authentication-algorithm> <authentication-key>` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 9, OSPFv3 Area Authentication and Encryption | Provides area-level ESP encryption syntax under `router ospfv3 <process-id>` |
| Related labs | `ospfv3-keychain-authentication-final`, `ospfv3-esp-encryption-final` | Main labs for OSPFv3 key-chain authentication trailer and OSPFv3 ESP encryption |
# OSPFv3_Keychain_Authentication_And_ESP_Encryption_Mental_Model
| Concept | Operational Meaning |
|---|---|
| OSPFv3 security | OSPFv3 can protect routing packets using the newer authentication trailer model or the older IPsec-based model |
| Authentication trailer | Non-IPsec OSPFv3 authentication method that appends an authentication trailer to OSPFv3 packets |
| Key-chain authentication | Uses IOS key chains to manage authentication keys, key IDs, algorithms, and optional lifetimes |
| HMAC-SHA-256 | Common modern authentication algorithm used with OSPFv3 key-chain authentication |
| Deployment mode | Transitional mode used during rollout so adjacency impact can be controlled while authentication is being introduced |
| Normal mode | Fully enforced authentication mode after all OSPFv3 speakers are configured consistently |
| Null authentication | Explicitly disables authentication for an interface or area, useful as an override in controlled labs |
| Interface-level authentication | Applied directly under the interface with `ospfv3 <process-id> <af> authentication key-chain <name>` |
| Area-level authentication | Applied under the OSPFv3 address family with `area <area-id> authentication key-chain <name>` |
| Precedence | Interface-level OSPFv3 security settings override area-level settings |
| Address-family awareness | OSPFv3 authentication trailer syntax can be scoped to IPv4 AF or IPv6 AF |
| ESP encryption | IPsec ESP protects OSPFv3 packets with encryption plus authentication |
| ESP protocol | ESP uses IP protocol 50 |
| SPI | Security Parameter Index manually identifies the OSPFv3 IPsec security association |
| Manual SA model | OSPFv3 IPsec protection does not use IKE to negotiate keys. SPI, algorithms, and keys must match manually |
| SPI uniqueness | IPsec peers should not reuse the same SPI values for different security associations |
| ESP area protection | Area-level ESP configuration requires all routers in that area to use matching security settings |
| ESP interface protection | Interface-level ESP applies only to that OSPFv3 interface and overrides area settings |
| Key mismatch failure | Wrong key string, key ID, algorithm, SPI, or encryption settings can prevent neighbors from reaching `FULL` |
| Verification target | `show ospfv3 interface` should show authentication or encryption state and secure socket status |
| Design choice | Use authentication trailer key-chain where supported. Use ESP encryption when packet confidentiality is specifically required or the lab demands IPsec-based OSPFv3 protection |
# OSPFv3_Keychain_Authentication_And_ESP_Encryption_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify whether the lab requires key-chain authentication or ESP encryption | Design step | `Security mode: key-chain authentication / ESP encryption` | Correct OSPFv3 security mechanism is selected |
| 2 | Identify protected OSPFv3 address family | Design step | `AF: ipv4 / ipv6` | Configuration is applied to the correct OSPFv3 AF |
| 3 | Identify protected interfaces or area | Design step | `Interface: <interface-id>` or `Area: <area-id>` | Scope is known before security is applied |
| 4 | Confirm OSPFv3 baseline adjacency before security changes | Both neighbors | `show ospfv3 neighbor` | Neighbor is currently `FULL` |
| 5 | Confirm OSPFv3 interface participation | Both neighbors | `show ospfv3 interface brief` | Interfaces appear under correct process, AF, and area |
| 6 | Confirm detailed OSPFv3 interface state | Both neighbors | `show ospfv3 interface <interface-id>` | Area, process ID, AF, network type, cost, and current security state are known |
| 7 | Confirm OSPFv3 process and AF configuration | Both neighbors | `show running-config | section router ospfv3` | Existing OSPFv3 AF configuration is known |
| 8 | Confirm interface-level OSPFv3 security config | Both neighbors | `show running-config interface <interface-id>` | Existing interface authentication or encryption commands are known |
| 9 | Enter global configuration mode for key-chain authentication | Both neighbors | `configure terminal` | Router enters global configuration mode |
| 10 | Create the key chain | Both neighbors | `key chain <KEY_CHAIN_NAME>` | Key-chain configuration mode opens |
| 11 | Create matching key ID | Both neighbors | `key <KEY_ID>` | Key-chain key configuration mode opens |
| 12 | Configure matching shared secret | Both neighbors | `key-string <SHARED_SECRET>` | Same key string exists on both neighbors |
| 13 | Configure HMAC-SHA-256 algorithm | Both neighbors | `cryptographic-algorithm hmac-sha-256` | Key uses HMAC-SHA-256 |
| 14 | Configure optional key send lifetime | Both neighbors | `send-lifetime local <hh:mm:ss> <day> <month> <year> infinite` | Key is valid for sending during intended window |
| 15 | Configure optional key accept lifetime | Both neighbors | `accept-lifetime local <hh:mm:ss> <day> <month> <year> infinite` | Key is valid for receiving during intended window |
| 16 | Apply key-chain authentication to one OSPFv3 interface | Both neighbors | `interface <interface-id>` then `ospfv3 <process-id> <ipv4-or-ipv6> authentication key-chain <KEY_CHAIN_NAME>` | Interface uses authentication trailer key-chain for selected AF |
| 17 | Apply key-chain authentication to an entire OSPFv3 area if required | All routers in area | `router ospfv3 <process-id>` then `address-family <ipv4-or-ipv6> unicast` then `area <area-id> authentication key-chain <KEY_CHAIN_NAME>` | Area uses authentication trailer key-chain |
| 18 | Use deployment mode during rollout if the lab requires staged migration | All routers in protected AF | `authentication mode deployment` | Authentication rollout can be staged |
| 19 | Move to normal mode after all routers are configured | All routers in protected AF | `authentication mode normal` | Authentication is fully enforced |
| 20 | Save key-chain authentication configuration | All changed routers | `end` then `write memory` | Configuration is saved |
| 21 | Verify key-chain contents | All changed routers | `show key chain <KEY_CHAIN_NAME>` | Key ID, algorithm, and lifetime information are visible |
| 22 | Verify OSPFv3 authentication trailer state | All changed routers | `show ospfv3` | Output shows authentication mode and key-chain information |
| 23 | Verify OSPFv3 interface authentication state | Both neighbors | `show ospfv3 interface <interface-id>` | Interface shows authentication behavior for the selected AF |
| 24 | Verify neighbor recovery after key-chain authentication | Both neighbors | `show ospfv3 neighbor` | Neighbor returns to `FULL` |
| 25 | Confirm SPI availability before ESP configuration | Both neighbors | `show crypto ipsec sa | include spi` | Existing SPI values are known so conflicts can be avoided |
| 26 | Enter global configuration mode for interface-level ESP | Both neighbors | `configure terminal` | Router enters global configuration mode |
| 27 | Enter protected OSPFv3 interface | Both neighbors | `interface <interface-id>` | Interface configuration mode opens |
| 28 | Configure ESP encryption and authentication on interface | Both neighbors | `ospfv3 encryption ipsec spi <SPI> esp <ENCRYPTION_ALGORITHM> <ENCRYPTION_KEY> <AUTH_ALGORITHM> <AUTH_KEY>` | OSPFv3 packets on the interface are encrypted and authenticated |
| 29 | Configure ESP null encryption if only ESP authentication behavior is required | Both neighbors | `ospfv3 encryption null` | Payload encryption is disabled while ESP authentication behavior is used where supported |
| 30 | Configure area-level ESP encryption if the whole area must be protected | All routers in area | `router ospfv3 <process-id>` then `area <area-id> encryption ipsec spi <SPI> esp <ENCRYPTION_ALGORITHM> <ENCRYPTION_KEY> <AUTH_ALGORITHM> <AUTH_KEY>` | OSPFv3 area uses ESP protection |
| 31 | Save ESP configuration | All changed routers | `end` then `write memory` | Configuration is saved |
| 32 | Verify ESP state on interface | Both neighbors | `show ospfv3 interface <interface-id>` | Output shows ESP encryption/authentication and secure socket status |
| 33 | Verify IPsec security association state | Both neighbors | `show crypto ipsec sa` | OSPFv3 IPsec SAs exist and counters increment |
| 34 | Verify neighbor state after ESP configuration | Both neighbors | `show ospfv3 neighbor` | Neighbor reaches `FULL` |
| 35 | Verify OSPFv3 LSDB remains synchronized | Both neighbors | `show ospfv3 database` | Expected LSAs remain present |
| 36 | Verify route installation | Both neighbors | `show ipv6 route ospf` or `show ip route ospfv3` | Expected OSPFv3 routes remain installed |
| 37 | Verify reachability across protected adjacency | Test router | `ping <remote-loopback>` | Remote destination responds |
| 38 | Check logs for security mismatch symptoms | Both neighbors | `show logging | include OSPFv3|OSPF|IPSEC|SPI|AUTH|ADJCHG` | No repeated authentication, SPI, or adjacency failures |
# OSPFv3_Keychain_Authentication_And_ESP_Encryption_Skeleton
! =========================================================
! OSPFv3 authentication trailer with key chain
! Interface-level model
! =========================================================
configure terminal
key chain <KEY_CHAIN_NAME>
 key <KEY_ID>
  key-string <SHARED_SECRET>
  cryptographic-algorithm hmac-sha-256
 exit
exit
interface <OSPFV3_INTERFACE>
 ospfv3 <PROCESS_ID> <ipv4-or-ipv6> authentication key-chain <KEY_CHAIN_NAME>
exit
router ospfv3 <PROCESS_ID>
 address-family <ipv4-or-ipv6> unicast
  authentication mode deployment
 exit-address-family
exit
end
write memory
! =========================================================
! Move authentication trailer from deployment to normal mode
! =========================================================
configure terminal
router ospfv3 <PROCESS_ID>
 address-family <ipv4-or-ipv6> unicast
  authentication mode normal
 exit-address-family
exit
end
write memory
! =========================================================
! OSPFv3 authentication trailer with key chain
! Area-level model
! =========================================================
configure terminal
key chain <KEY_CHAIN_NAME>
 key <KEY_ID>
  key-string <SHARED_SECRET>
  cryptographic-algorithm hmac-sha-256
 exit
exit
router ospfv3 <PROCESS_ID>
 address-family <ipv4-or-ipv6> unicast
  area <AREA_ID> authentication key-chain <KEY_CHAIN_NAME>
  authentication mode normal
 exit-address-family
exit
end
write memory
! =========================================================
! Explicit null authentication override
! Use only when intentionally disabling authentication
! on a specific interface or area.
! =========================================================
configure terminal
interface <OSPFV3_INTERFACE>
 ospfv3 <PROCESS_ID> <ipv4-or-ipv6> authentication null
exit
router ospfv3 <PROCESS_ID>
 address-family <ipv4-or-ipv6> unicast
  area <AREA_ID> authentication null
 exit-address-family
exit
end
write memory
! =========================================================
! OSPFv3 ESP encryption and authentication
! Interface-level model
! This is one long command even if it wraps in terminal output.
! =========================================================
configure terminal
interface <OSPFV3_INTERFACE>
 ospfv3 encryption ipsec spi <SPI> esp <ENCRYPTION_ALGORITHM> <ENCRYPTION_KEY> <AUTH_ALGORITHM> <AUTH_KEY>
exit
end
write memory
! =========================================================
! OSPFv3 ESP encryption and authentication
! Area-level model
! This applies to the area and must match on routers in that area.
! =========================================================
configure terminal
router ospfv3 <PROCESS_ID>
 area <AREA_ID> encryption ipsec spi <SPI> esp <ENCRYPTION_ALGORITHM> <ENCRYPTION_KEY> <AUTH_ALGORITHM> <AUTH_KEY>
end
write memory
! =========================================================
! Example key-chain authentication trailer for IPv6 AF
! =========================================================
configure terminal
key chain OSPF3_AUTH
 key 1
  key-string <SHARED_SECRET>
  cryptographic-algorithm hmac-sha-256
 exit
exit
interface GigabitEthernet0/0
 ospfv3 1 ipv6 authentication key-chain OSPF3_AUTH
exit
router ospfv3 1
 address-family ipv6 unicast
  authentication mode normal
 exit-address-family
exit
end
write memory
! =========================================================
! Example ESP encryption on one interface
! Values are placeholders. Use real non-predictable keys.
! =========================================================
configure terminal
interface GigabitEthernet0/0
 ospfv3 encryption ipsec spi 500 esp 3des <ENCRYPTION_KEY> sha1 <AUTH_KEY>
exit
end
write memory
# OSPFv3_Keychain_Authentication_And_ESP_Encryption_Verification_Commands
show running-config | section key chain
show running-config | section router ospfv3
show running-config interface <interface-id>
show key chain
show key chain <KEY_CHAIN_NAME>
show ospfv3
show ospfv3 interface brief
show ospfv3 interface <interface-id>
show ospfv3 neighbor
show ospfv3 neighbor detail
show ospfv3 database
show ipv6 route ospf
show ip route ospfv3
show crypto ipsec sa
show crypto ipsec sa | include spi
show logging | include OSPFv3|OSPF|AUTH|authentication|IPSEC|IPsec|SPI|ADJCHG
ping <remote-loopback>
traceroute <remote-loopback>
# OSPFv3_Keychain_Authentication_And_ESP_Encryption_Rollback
! =========================================================
! Remove interface-level key-chain authentication
! =========================================================
configure terminal
interface <OSPFV3_INTERFACE>
 no ospfv3 <PROCESS_ID> <ipv4-or-ipv6> authentication key-chain <KEY_CHAIN_NAME>
end
write memory
! =========================================================
! Remove area-level key-chain authentication
! =========================================================
configure terminal
router ospfv3 <PROCESS_ID>
 address-family <ipv4-or-ipv6> unicast
  no area <AREA_ID> authentication key-chain <KEY_CHAIN_NAME>
 exit-address-family
end
write memory
! =========================================================
! Remove authentication mode
! =========================================================
configure terminal
router ospfv3 <PROCESS_ID>
 address-family <ipv4-or-ipv6> unicast
  no authentication mode
 exit-address-family
end
write memory
! =========================================================
! Remove key chain after detaching it from OSPFv3
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
! Remove interface-level OSPFv3 ESP encryption
! =========================================================
configure terminal
interface <OSPFV3_INTERFACE>
 no ospfv3 encryption
end
write memory
! =========================================================
! Remove area-level OSPFv3 ESP encryption
! =========================================================
configure terminal
router ospfv3 <PROCESS_ID>
 no area <AREA_ID> encryption ipsec spi <SPI> esp <ENCRYPTION_ALGORITHM> <ENCRYPTION_KEY> <AUTH_ALGORITHM> <AUTH_KEY>
end
write memory
! =========================================================
! Explicitly disable authentication with null only when intended
! =========================================================
configure terminal
interface <OSPFV3_INTERFACE>
 ospfv3 <PROCESS_ID> <ipv4-or-ipv6> authentication null
end
write memory
! =========================================================
! Lab-only OSPFv3 reset
! This disrupts adjacencies.
! =========================================================
clear ospfv3 process
# OSPFv3_Keychain_Authentication_And_ESP_Encryption_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| Neighbor drops after key-chain authentication | Key-chain name mismatch | `show running-config interface <interface-id>` | Attach the correct local key chain on both sides |
| Neighbor drops after key-chain authentication | Key ID mismatch | `show key chain <KEY_CHAIN_NAME>` | Configure matching key IDs |
| Neighbor drops after key-chain authentication | Key string mismatch | `show running-config | section key chain` | Reconfigure the same shared secret on both routers |
| Neighbor drops after key-chain authentication | Algorithm mismatch | `show key chain <KEY_CHAIN_NAME>` | Use matching `cryptographic-algorithm hmac-sha-256` |
| Neighbor drops after key-chain authentication | Key lifetime is not active | `show key chain <KEY_CHAIN_NAME>` | Correct send and accept lifetimes or remove lifetime constraints in lab |
| Neighbor remains up during rollout but fails later | Authentication was left in deployment mode during transition and then moved to normal before all routers were ready | `show ospfv3` | Configure all routers consistently before changing to `authentication mode normal` |
| Authentication configured but not enforced as expected | Deployment mode is active | `show ospfv3` | Move to `authentication mode normal` after validation |
| Interface does not use area-level authentication | Interface-level authentication or null override is configured | `show running-config interface <interface-id>` | Remove interface override or configure intended interface-level authentication |
| Area authentication fails on one link only | Interface-level setting overrides area-level setting | `show ospfv3 interface <interface-id>` | Fix or remove the interface-level setting |
| `ospfv3 authentication key-chain` command is rejected | Platform or image does not support OSPFv3 authentication trailer | `interface <interface-id>` then `ospfv3 ?` | Use supported IOS XE image or use IPsec-based OSPFv3 security if the lab requires it |
| `cryptographic-algorithm hmac-sha-256` command is rejected | Key-chain algorithm support is missing on the image | `key chain <name>` then `key <id>` then `?` | Use supported image or supported algorithm |
| ESP neighbor fails to form | SPI mismatch | `show running-config interface <interface-id>`, `show crypto ipsec sa | include spi` | Configure matching SPI values for the OSPFv3 peers |
| ESP neighbor fails to form | Encryption algorithm mismatch | `show running-config interface <interface-id>` | Configure matching ESP encryption algorithm on both sides |
| ESP neighbor fails to form | Encryption key mismatch | `show running-config interface <interface-id>` | Reconfigure matching encryption key |
| ESP neighbor fails to form | Authentication algorithm mismatch | `show running-config interface <interface-id>` | Configure matching authentication algorithm |
| ESP neighbor fails to form | Authentication key mismatch | `show running-config interface <interface-id>` | Reconfigure matching authentication key |
| ESP secure socket shows errors | SPI, key, or algorithm mismatch | `show ospfv3 interface <interface-id>`, `show crypto ipsec sa` | Correct all ESP parameters on both peers |
| ESP command works on area but one interface behaves differently | Interface-level encryption overrides area-level encryption | `show running-config interface <interface-id>` | Remove interface-level override or configure it intentionally |
| `show crypto ipsec sa` shows conflicting SPI | SPI reused by another manual SA | `show crypto ipsec sa | include spi` | Choose unused SPI values |
| OSPFv3 route disappears after security change | Neighbor adjacency failed | `show ospfv3 neighbor`, `show ospfv3 interface <interface-id>` | Restore neighbor `FULL` state first |
| LSDB is missing after security change | Authentication or ESP mismatch prevented adjacency | `show ospfv3 database`, `show ospfv3 neighbor detail` | Fix security mismatch and recheck LSDB |
| Ping fails but neighbor is `FULL` | Not a security issue after adjacency is restored | `show ipv6 route ospf` or `show ip route ospfv3` | Check routing, return path, ACLs, and interface state |
| Key-chain authentication and ESP are both configured unexpectedly | Mixed security models were applied to the same scope | `show running-config interface <interface-id>`, `show running-config | section router ospfv3` | Use one intended protection model per lab scope |
| Null authentication leaves link unprotected | `authentication null` was left after testing | `show running-config interface <interface-id>` | Remove null override and apply key-chain authentication |
| Debug or logs show repeated adjacency resets | Security parameters are inconsistent or timers are flapping after repeated resets | `show logging | include OSPFv3|AUTH|IPSEC|ADJCHG` | Fix security first, then check OSPFv3 timers, area, MTU, and interface state |
| Area-wide ESP breaks multiple adjacencies | Not every router in the area has matching ESP settings | `show running-config | section router ospfv3` on all area routers | Configure matching area ESP on all routers or use interface-level ESP only where required |
| Key rollover causes outage | Send and accept lifetimes do not overlap | `show key chain <KEY_CHAIN_NAME>` | Configure overlapping key lifetimes before changing active keys |
##### Source_Basis
# OSPFv3_Keychain_Authentication_And_ESP_Encryption_Mental_Model
# OSPFv3_Keychain_Authentication_And_ESP_Encryption_Configuration_Checklist
# OSPFv3_Keychain_Authentication_And_ESP_Encryption_Skeleton
# OSPFv3_Keychain_Authentication_And_ESP_Encryption_Verification_Commands
# OSPFv3_Keychain_Authentication_And_ESP_Encryption_Rollback
# OSPFv3_Keychain_Authentication_And_ESP_Encryption_Failure_Checks
# index of each title throughout note, not in table format


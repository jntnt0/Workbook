OSPF_Virtual_Link_Authentication.md

OSPF_Virtual_Link_Authentication

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | OSPF topic map | Points enterprise OSPF to `All_combined_part3.md` Book 6 as the main project OSPF source |
| `All_combined_part3.md` | Book 6, Chapter 8, OSPF Authentication | Supports area authentication, MD5 message-digest authentication, interface keys, and authentication mismatch behavior |
| `All_combined_part3.md` | Book 6, Chapter 8, Virtual Link Authentication tasks | Supports `area <area-id> virtual-link <router-id> authentication message-digest`, authentication reset, and `authentication null` behavior |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 6, OSPF Authentication | Supports plaintext authentication, MD5 authentication, key ID, password matching, and verification concepts |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 7, Virtual Links | Supports virtual-link endpoint, transit-area, router-ID, and `show ip ospf virtual-links` verification |
| Related labs | `ospf-virtual-link-authentication-md5-final`, `ospf-virtual-link-authentication-plain-final` | Main labs for applying authentication directly to OSPF virtual links |
# OSPF_Virtual_Link_Authentication_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Virtual link | Logical OSPF backbone link built between ABRs across a normal transit area |
| Authentication location | Virtual-link authentication is configured under `router ospf`, not under the physical transit interface |
| Remote router ID | The virtual-link command references the remote endpoint router ID, not the remote interface IP |
| Transit area | The area number in the command is the transit area used to reach the remote endpoint |
| Area 0 logic | A virtual link behaves like a logical Area 0 point-to-point interface |
| Plaintext authentication | Simple password authentication. It is weak but useful for lab comparison |
| MD5 authentication | Message-digest authentication. The password is not sent in clear text and is preferred over plaintext |
| Key ID | MD5 uses a key ID. The key ID and password must match on both virtual-link endpoints |
| Matching requirement | Both virtual-link endpoints must use the same authentication type and key material |
| Authentication mismatch | A mismatch prevents the virtual-link adjacency from reaching `FULL` |
| Area authentication inheritance | If Area 0 authentication is enabled, the virtual link may need matching authentication unless overridden |
| `authentication null` | Explicitly disables authentication on a virtual link, often used to override area-level authentication |
| Physical transit interface auth | Transit-area physical interface authentication is separate from virtual-link authentication |
| Verification target | The authoritative command is `show ip ospf virtual-links`, because it shows the virtual-link state and authentication behavior indirectly through adjacency status |
# OSPF_Virtual_Link_Authentication_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the virtual-link endpoints | Design step | `Endpoint A: <router-name>, Endpoint B: <router-name>` | Correct ABRs are selected |
| 2 | Confirm the transit area | Design step | `Transit area: <area-id>` | Same transit area will be used on both endpoints |
| 3 | Confirm the remote router IDs | Both endpoints | `show ip ospf | include ID` | Each endpoint router ID is known |
| 4 | Confirm the virtual link is already configured or planned | Both endpoints | `show running-config | section router ospf` | Existing `area <transit-area> virtual-link <remote-router-id>` command is visible or ready to configure |
| 5 | Confirm the transit area is not stub or NSSA | Both endpoints | `show ip ospf` | Transit area is a normal OSPF area |
| 6 | Confirm transit-area reachability between endpoint router IDs | Both endpoints | `show ip route <remote-router-id>` | Each endpoint can reach the remote router ID through the transit area |
| 7 | Confirm virtual-link status before authentication | Both endpoints | `show ip ospf virtual-links` | Virtual link state is known before changes |
| 8 | Confirm OSPF routes before authentication change | Affected routers | `show ip route ospf` | Existing interarea reachability baseline is known |
| 9 | Enter configuration mode | Endpoint A | `configure terminal` | Router enters global configuration mode |
| 10 | Enter OSPF process | Endpoint A | `router ospf <process-id>` | OSPF router configuration mode opens |
| 11 | Configure plaintext virtual-link authentication if required | Endpoint A | `area <transit-area-id> virtual-link <endpoint-b-router-id> authentication` | Plaintext authentication is enabled on the virtual link |
| 12 | Configure plaintext virtual-link password if required | Endpoint A | `area <transit-area-id> virtual-link <endpoint-b-router-id> authentication-key <password>` | Plaintext key is applied to the virtual link |
| 13 | Configure MD5 virtual-link authentication if required | Endpoint A | `area <transit-area-id> virtual-link <endpoint-b-router-id> authentication message-digest` | MD5 authentication is enabled on the virtual link |
| 14 | Configure MD5 virtual-link key if required | Endpoint A | `area <transit-area-id> virtual-link <endpoint-b-router-id> message-digest-key <key-id> md5 <password>` | MD5 key ID and password are applied |
| 15 | Exit configuration mode | Endpoint A | `end` | Router returns to privileged EXEC mode |
| 16 | Enter configuration mode | Endpoint B | `configure terminal` | Router enters global configuration mode |
| 17 | Enter OSPF process | Endpoint B | `router ospf <process-id>` | OSPF router configuration mode opens |
| 18 | Configure matching plaintext virtual-link authentication if required | Endpoint B | `area <transit-area-id> virtual-link <endpoint-a-router-id> authentication` | Plaintext authentication matches endpoint A |
| 19 | Configure matching plaintext virtual-link password if required | Endpoint B | `area <transit-area-id> virtual-link <endpoint-a-router-id> authentication-key <password>` | Plaintext key matches endpoint A |
| 20 | Configure matching MD5 virtual-link authentication if required | Endpoint B | `area <transit-area-id> virtual-link <endpoint-a-router-id> authentication message-digest` | MD5 authentication matches endpoint A |
| 21 | Configure matching MD5 virtual-link key if required | Endpoint B | `area <transit-area-id> virtual-link <endpoint-a-router-id> message-digest-key <key-id> md5 <password>` | MD5 key ID and password match endpoint A |
| 22 | Exit configuration mode | Endpoint B | `end` | Router returns to privileged EXEC mode |
| 23 | Save configuration | Both endpoints | `write memory` | Configuration is saved |
| 24 | Verify virtual-link configuration | Both endpoints | `show running-config | section router ospf` | Virtual-link authentication commands are present on both endpoints |
| 25 | Verify virtual-link state | Both endpoints | `show ip ospf virtual-links` | Virtual link is `up` and adjacency state is `FULL` |
| 26 | Verify OSPF neighbor state | Both endpoints | `show ip ospf neighbor` | Virtual-link neighbor reaches `FULL` |
| 27 | Verify LSDB continuity | Both endpoints and affected routers | `show ip ospf database summary` | Expected Type 3 LSAs remain visible |
| 28 | Verify interarea routes remain installed | Affected routers | `show ip route ospf` | Expected `O IA` routes remain present |
| 29 | Verify end-to-end reachability | Affected routers | `ping <remote-area-loopback-ip>` | Remote area loopback responds |
| 30 | Verify forwarding path | Affected routers | `traceroute <remote-area-loopback-ip>` | Path still crosses the expected virtual-link repair path |
| 31 | Override authentication only when intentionally disabling it | Both endpoints | `area <transit-area-id> virtual-link <remote-router-id> authentication null` | Authentication is explicitly disabled on the virtual link |
# OSPF_Virtual_Link_Authentication_Skeleton
! =========================================================
! Plaintext virtual-link authentication
! Configure matching values on both virtual-link endpoints.
! =========================================================
! Endpoint A:
configure terminal
router ospf <PROCESS_ID>
 area <TRANSIT_AREA_ID> virtual-link <ENDPOINT_B_ROUTER_ID> authentication
 area <TRANSIT_AREA_ID> virtual-link <ENDPOINT_B_ROUTER_ID> authentication-key <PASSWORD>
end
write memory
! Endpoint B:
configure terminal
router ospf <PROCESS_ID>
 area <TRANSIT_AREA_ID> virtual-link <ENDPOINT_A_ROUTER_ID> authentication
 area <TRANSIT_AREA_ID> virtual-link <ENDPOINT_A_ROUTER_ID> authentication-key <PASSWORD>
end
write memory
! =========================================================
! MD5 virtual-link authentication
! Preferred over plaintext.
! Configure matching key ID and password on both endpoints.
! =========================================================
! Endpoint A:
configure terminal
router ospf <PROCESS_ID>
 area <TRANSIT_AREA_ID> virtual-link <ENDPOINT_B_ROUTER_ID> authentication message-digest
 area <TRANSIT_AREA_ID> virtual-link <ENDPOINT_B_ROUTER_ID> message-digest-key <KEY_ID> md5 <PASSWORD>
end
write memory
! Endpoint B:
configure terminal
router ospf <PROCESS_ID>
 area <TRANSIT_AREA_ID> virtual-link <ENDPOINT_A_ROUTER_ID> authentication message-digest
 area <TRANSIT_AREA_ID> virtual-link <ENDPOINT_A_ROUTER_ID> message-digest-key <KEY_ID> md5 <PASSWORD>
end
write memory
! =========================================================
! Disable authentication on a virtual link
! Useful when overriding area-level authentication in a lab.
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 area <TRANSIT_AREA_ID> virtual-link <REMOTE_ROUTER_ID> authentication null
end
write memory
! =========================================================
! Example MD5
! R2 RID 0.0.0.2
! R3 RID 0.0.0.3
! Transit area 1
! Key ID 1
! Password Micronics
! =========================================================
! On R2:
configure terminal
router ospf 1
 area 1 virtual-link 0.0.0.3 authentication message-digest
 area 1 virtual-link 0.0.0.3 message-digest-key 1 md5 Micronics
end
write memory
! On R3:
configure terminal
router ospf 1
 area 1 virtual-link 0.0.0.2 authentication message-digest
 area 1 virtual-link 0.0.0.2 message-digest-key 1 md5 Micronics
end
write memory
# OSPF_Virtual_Link_Authentication_Verification_Commands
show running-config | section router ospf
show ip ospf
show ip ospf interface brief
show ip ospf neighbor
show ip ospf neighbor detail
show ip ospf virtual-links
show ip ospf database
show ip ospf database router
show ip ospf database summary
show ip route ospf
show ip route <remote-router-id>
show ip route <remote-area-prefix>
ping <remote-router-id>
ping <remote-area-loopback-ip>
traceroute <remote-area-loopback-ip>
show logging | include OSPF|AUTH|authentication|virtual-link|mismatch|ADJCHG
# OSPF_Virtual_Link_Authentication_Rollback
! =========================================================
! Remove plaintext virtual-link authentication
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no area <TRANSIT_AREA_ID> virtual-link <REMOTE_ROUTER_ID> authentication
 no area <TRANSIT_AREA_ID> virtual-link <REMOTE_ROUTER_ID> authentication-key <PASSWORD>
end
write memory
! =========================================================
! Remove MD5 virtual-link authentication
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no area <TRANSIT_AREA_ID> virtual-link <REMOTE_ROUTER_ID> authentication message-digest
 no area <TRANSIT_AREA_ID> virtual-link <REMOTE_ROUTER_ID> message-digest-key <KEY_ID> md5 <PASSWORD>
end
write memory
! =========================================================
! Replace wrong MD5 key
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no area <TRANSIT_AREA_ID> virtual-link <REMOTE_ROUTER_ID> message-digest-key <WRONG_KEY_ID> md5 <WRONG_PASSWORD>
 area <TRANSIT_AREA_ID> virtual-link <REMOTE_ROUTER_ID> message-digest-key <CORRECT_KEY_ID> md5 <CORRECT_PASSWORD>
end
write memory
! =========================================================
! Disable authentication on the virtual link
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 area <TRANSIT_AREA_ID> virtual-link <REMOTE_ROUTER_ID> authentication null
end
write memory
! =========================================================
! Remove the whole virtual link
! =========================================================
configure terminal
router ospf <PROCESS_ID>
 no area <TRANSIT_AREA_ID> virtual-link <REMOTE_ROUTER_ID>
end
write memory
! =========================================================
! Lab-only OSPF reset
! This disrupts adjacencies.
! =========================================================
clear ip ospf process
# OSPF_Virtual_Link_Authentication_Failure_Checks
| Symptom | Likely Cause | Check | Corrective Action |
|---|---|---|---|
| Virtual link drops after authentication is enabled | One side has authentication and the other does not | `show running-config | section router ospf` | Configure matching virtual-link authentication on both endpoints |
| Virtual link stays down | Authentication type mismatch | `show running-config | section router ospf`, `show ip ospf virtual-links` | Match plaintext, MD5, or null authentication on both endpoints |
| Virtual link stays down | MD5 key ID mismatch | `show running-config | section router ospf` | Use the same `message-digest-key <key-id>` on both endpoints |
| Virtual link stays down | MD5 password mismatch | `show running-config | section router ospf`, `show logging | include AUTH` | Reconfigure matching MD5 password on both endpoints |
| Virtual link stays down | Plaintext authentication key mismatch | `show running-config | section router ospf` | Reconfigure matching `authentication-key <password>` on both endpoints |
| Virtual link authentication configured under physical interface only | Authentication was applied to transit interface, not the virtual link | `show running-config interface <transit-interface>`, `show running-config | section router ospf` | Configure authentication under `router ospf` with the virtual-link command |
| Physical transit adjacency works but virtual link fails | Virtual-link authentication mismatch | `show ip ospf neighbor`, `show ip ospf virtual-links` | Leave transit-area authentication alone and fix virtual-link authentication |
| Area 0 authentication breaks virtual link | Virtual link inherits or conflicts with backbone authentication expectations | `show running-config | section router ospf` | Configure matching virtual-link authentication or explicitly use `authentication null` if intended |
| Virtual link fails after changing Area 0 auth | Virtual-link auth was not updated with the backbone policy | `show ip ospf virtual-links`, `show logging | include AUTH` | Apply matching virtual-link authentication on both endpoints |
| Virtual link stays down despite matching auth | Wrong remote router ID | `show ip ospf | include ID`, `show running-config | section router ospf` | Use the remote endpoint router ID, not interface IP |
| Virtual link stays down despite matching auth | Wrong transit area ID | `show running-config | section router ospf`, `show ip ospf` | Use the transit area shared by both endpoints |
| Virtual link stays down despite matching auth | Transit area is stub or NSSA | `show ip ospf` | Convert transit area to normal area or use a different transit area |
| Virtual link stays down despite matching auth | Remote router ID not reachable through transit area | `show ip route <remote-router-id>` | Restore OSPF reachability through the transit area |
| Neighbor state flaps repeatedly | Authentication mismatch or unstable transit-area path | `show logging | include OSPF|AUTH|ADJCHG` | Fix authentication first, then check transit-area reachability |
| `show ip ospf neighbor` does not clearly show virtual-link auth | Expected behavior | `show ip ospf virtual-links` | Use virtual-link verification commands and running config |
| Interarea routes disappear | Virtual link adjacency failed | `show ip ospf virtual-links`, `show ip route ospf` | Restore virtual-link adjacency and authentication match |
| LSDB summaries disappear | Virtual link is not fully adjacent | `show ip ospf database summary` | Fix virtual-link state before troubleshooting Type 3 LSAs |
| Ping fails after auth change | Virtual link is down or return path missing | `show ip ospf virtual-links`, `show ip route <remote-prefix>` | Restore virtual link, then verify forward and reverse routes |
##### Source_Basis
# OSPF_Virtual_Link_Authentication_Mental_Model
# OSPF_Virtual_Link_Authentication_Configuration_Checklist
# OSPF_Virtual_Link_Authentication_Skeleton
# OSPF_Virtual_Link_Authentication_Verification_Commands
# OSPF_Virtual_Link_Authentication_Rollback
# OSPF_Virtual_Link_Authentication_Failure_Checks
# index of each title throughout note, not in table format


PPPoE_Verification_And_Failure_Isolation.md
# PPPoE_Verification_And_Failure_Isolation
# PPPoE_Verification_And_Failure_Isolation_Related_Labs
- pppoe-server-final
- pppoe-server-two-clients-final
- pppoe-server-client-authentication-final
# PPPoE_Verification_And_Failure_Isolation_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Layered verification | Verify PPPoE from the bottom up: physical link, discovery, session, PPP LCP, authentication, IPCP, then IP reachability |
| Physical link | PPPoE cannot start if the Ethernet interface is down, shutdown, or connected to the wrong segment |
| Discovery stage | Client sends PADI, server replies PADO, client sends PADR, server replies PADS |
| Session stage | After PADS, the server assigns a PPPoE Session ID and PPP runs inside the PPPoE session |
| Server attachment point | The access concentrator must have `pppoe enable group <name>` on the client-facing Ethernet interface |
| Client attachment point | The client physical interface must have `pppoe-client dial-pool-number <number>` |
| Dialer pool binding | The client Dialer `dialer pool <number>` must match the physical interface `pppoe-client dial-pool-number <number>` |
| Virtual-Template | The server-side template used to create a dynamic Virtual-Access interface per client |
| Virtual-Access | The proof that the server accepted a PPPoE client and created a per-session PPP interface |
| LCP Open | PPP link control is established inside the PPPoE session |
| Authentication pass | PAP or CHAP succeeded if authentication is configured |
| IPCP Open | IP control protocol succeeded and the client should have an IP address |
| Local address pool | Server-side pool used to assign client addresses through IPCP |
| Dialer up/up | Client-side proof that PPPoE and PPP negotiation succeeded far enough to create a usable logical interface |
| Ping test | Final reachability check, not the first troubleshooting step |
# PPPoE_Verification_And_Failure_Isolation_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm all routers have the expected physical interfaces | R1 / R2 / R3 | `show ip interface brief` | PPPoE-facing Ethernet interfaces are listed |
| 2 | Confirm the access concentrator Ethernet interface is up/up | R1 / PPPoE-AC | `show ip interface brief FastEthernet0/0` | FastEthernet0/0 is up/up |
| 3 | Confirm client Ethernet interfaces are up/up | R2 / R3 | `show ip interface brief FastEthernet0/0` | Client FastEthernet0/0 interfaces are up/up |
| 4 | Confirm the server loopback endpoint exists | R1 / PPPoE-AC | `show ip interface brief Loopback123` | Loopback123 is up/up with 123.1.1.1 |
| 5 | Confirm the server Virtual-Template exists | R1 / PPPoE-AC | `show running-config interface Virtual-Template123` | Template contains `ip unnumbered Loopback123` |
| 6 | Confirm the server template assigns client addresses | R1 / PPPoE-AC | `show running-config interface Virtual-Template123` | Template contains `peer default ip address pool tst-pool` |
| 7 | Confirm server MTU and MSS handling | R1 / PPPoE-AC | `show running-config interface Virtual-Template123` | Template contains `mtu 1492` and `ip tcp adjust-mss 1452` |
| 8 | Confirm the local address pool exists | R1 / PPPoE-AC | `show running-config | include ip local pool` | Output shows `ip local pool tst-pool 123.1.1.2 123.1.1.3` |
| 9 | Confirm the PPPoE server bba-group exists | R1 / PPPoE-AC | `show running-config | section bba-group` | Output shows `bba-group pppoe tst` |
| 10 | Confirm the bba-group points to the correct Virtual-Template | R1 / PPPoE-AC | `show running-config | section bba-group` | Output shows `virtual-template 123` |
| 11 | Confirm PPPoE service is attached to the client-facing interface | R1 / PPPoE-AC | `show running-config interface FastEthernet0/0` | Output shows `pppoe enable group tst` |
| 12 | Confirm the client Dialer exists | R2 / R3 | `show running-config interface Dialer123` | Dialer123 exists |
| 13 | Confirm the client Dialer uses PPP | R2 / R3 | `show running-config interface Dialer123` | Output shows `encapsulation ppp` |
| 14 | Confirm the client Dialer receives its address by IPCP | R2 / R3 | `show running-config interface Dialer123` | Output shows `ip address negotiated` |
| 15 | Confirm the client Dialer has PPPoE-safe MTU and MSS | R2 / R3 | `show running-config interface Dialer123` | Output shows `mtu 1492` and `ip tcp adjust-mss 1452` |
| 16 | Confirm the client Dialer pool number | R2 / R3 | `show running-config interface Dialer123` | Output shows `dialer pool 100` |
| 17 | Confirm the physical client PPPoE binding | R2 / R3 | `show running-config interface FastEthernet0/0` | Output shows `pppoe-client dial-pool-number 100` |
| 18 | Confirm client Dialer pool matches physical PPPoE binding | R2 / R3 | `show running-config interface Dialer123` and `show running-config interface FastEthernet0/0` | `dialer pool 100` matches `pppoe-client dial-pool-number 100` |
| 19 | Confirm PPPoE sessions exist on the access concentrator | R1 / PPPoE-AC | `show pppoe session` | One session appears for single-client labs, two sessions appear for the two-client lab |
| 20 | Confirm each client has a unique Session ID | R1 / PPPoE-AC | `show pppoe session` | Each client maps to a unique PPPoE SID |
| 21 | Confirm each client has a unique remote MAC address | R1 / PPPoE-AC | `show pppoe session` | R2 and R3 show different remote MAC addresses |
| 22 | Confirm each client maps to a Virtual-Access interface | R1 / PPPoE-AC | `show pppoe session` | Output shows VA entries for each session |
| 23 | Inspect the first Virtual-Access interface | R1 / PPPoE-AC | `show interface virtual-access <id>` | Interface is up/up and cloned from Virtual-Template123 |
| 24 | Confirm PPP LCP opened | R1 / PPPoE-AC | `show interface virtual-access <id>` | Output shows `LCP Open` |
| 25 | Confirm IPCP opened | R1 / PPPoE-AC | `show interface virtual-access <id>` | Output shows `Open: IPCP` |
| 26 | Confirm the client Dialer came up | R2 / R3 | `show ip interface brief Dialer123` | Dialer123 is up/up |
| 27 | Confirm the client received an address | R2 / R3 | `show ip interface brief Dialer123` | Dialer123 has an IPCP-learned address |
| 28 | Confirm server pool consumption | R1 / PPPoE-AC | `show ip local pool` | Client addresses are allocated from `tst-pool` |
| 29 | Test client-to-server PPP reachability | R2 / R3 | `ping 123.1.1.1` | Ping succeeds |
| 30 | If no PPPoE session exists, check PPPoE discovery events | R1 / PPPoE-AC | `debug pppoe events` | Debug shows discovery activity when clients attempt to connect |
| 31 | If the session exists but PPP fails, check PPP negotiation | R1 / PPPoE-AC | `debug ppp negotiation` | Debug shows LCP and IPCP negotiation state |
| 32 | If authentication is configured, check PPP authentication | R1 / PPPoE-AC | `debug ppp authentication` | Debug shows PAP or CHAP success or failure |
| 33 | Disable debugging after the test | R1 / PPPoE-AC | `undebug all` | All debugs are disabled |
# PPPoE_Verification_And_Failure_Isolation_Skeleton
! PHASE 1: PHYSICAL AND BASELINE
show ip interface brief
show running-config interface FastEthernet0/0
! PHASE 2: SERVER-SIDE PPPoE CONTROL PLANE
show running-config | section bba-group
show running-config interface Virtual-Template123
show running-config | include ip local pool
show running-config interface FastEthernet0/0
! PHASE 3: CLIENT-SIDE DIALER AND PHYSICAL BINDING
show running-config interface Dialer123
show running-config interface FastEthernet0/0
show ip interface brief Dialer123
! PHASE 4: PPPoE SESSION STATE
show pppoe session
show interface virtual-access <id>
show ip local pool
! PHASE 5: FINAL REACHABILITY
ping 123.1.1.1
! PHASE 6: DEBUG ONLY IF NEEDED
debug pppoe events
debug pppoe errors
debug ppp negotiation
debug ppp authentication
undebug all
# PPPoE_Verification_And_Failure_Isolation_Verification_Commands
| Verification Goal | Device | Command | Good Output |
|---|---|---|---|
| Confirm physical interface inventory | R1 / R2 / R3 | `show ip interface brief` | Expected Ethernet, Loopback, Dialer, and Virtual-Access interfaces are visible as applicable |
| Confirm server access interface state | R1 / PPPoE-AC | `show ip interface brief FastEthernet0/0` | Interface is up/up |
| Confirm client access interface state | R2 / R3 | `show ip interface brief FastEthernet0/0` | Interface is up/up |
| Confirm server loopback endpoint | R1 / PPPoE-AC | `show ip interface brief Loopback123` | Loopback123 is up/up with 123.1.1.1 |
| Confirm bba-group configuration | R1 / PPPoE-AC | `show running-config | section bba-group` | Output shows `bba-group pppoe tst` and `virtual-template 123` |
| Confirm Virtual-Template inheritance source | R1 / PPPoE-AC | `show running-config interface Virtual-Template123` | Output shows the common PPP settings inherited by Virtual-Access interfaces |
| Confirm local pool exists | R1 / PPPoE-AC | `show running-config | include ip local pool` | Output shows `ip local pool tst-pool 123.1.1.2 123.1.1.3` |
| Confirm PPPoE server is enabled on Ethernet | R1 / PPPoE-AC | `show running-config interface FastEthernet0/0` | Output shows `pppoe enable group tst` |
| Confirm client Dialer PPP configuration | R2 / R3 | `show running-config interface Dialer123` | Output shows `encapsulation ppp` |
| Confirm client IPCP address negotiation | R2 / R3 | `show running-config interface Dialer123` | Output shows `ip address negotiated` |
| Confirm client Dialer pool | R2 / R3 | `show running-config interface Dialer123` | Output shows `dialer pool 100` |
| Confirm client physical binding | R2 / R3 | `show running-config interface FastEthernet0/0` | Output shows `pppoe-client dial-pool-number 100` |
| Confirm active PPPoE sessions | R1 / PPPoE-AC | `show pppoe session` | Active session entries appear |
| Confirm single-client lab result | R1 / PPPoE-AC | `show pppoe session` | One locally terminated PTA session appears |
| Confirm two-client lab result | R1 / PPPoE-AC | `show pppoe session` | Two locally terminated PTA sessions appear |
| Confirm unique Session IDs | R1 / PPPoE-AC | `show pppoe session` | Each client has a different PPPoE SID |
| Confirm Virtual-Access assignment | R1 / PPPoE-AC | `show pppoe session` | Each session maps to a VA interface |
| Confirm Virtual-Access clone source | R1 / PPPoE-AC | `show interface virtual-access <id>` | Output shows it is cloned from Virtual-Template123 |
| Confirm PPP LCP state | R1 / PPPoE-AC | `show interface virtual-access <id>` | Output shows `LCP Open` |
| Confirm PPP IPCP state | R1 / PPPoE-AC | `show interface virtual-access <id>` | Output shows `Open: IPCP` |
| Confirm client Dialer state | R2 / R3 | `show ip interface brief Dialer123` | Dialer123 is up/up |
| Confirm client negotiated address | R2 / R3 | `show ip interface brief Dialer123` | Dialer123 has 123.1.1.2 or 123.1.1.3 |
| Confirm address pool use | R1 / PPPoE-AC | `show ip local pool` | Pool shows allocated addresses |
| Confirm authenticated PPP state if authentication is enabled | R1 / PPPoE-AC | `debug ppp authentication` | PAP or CHAP succeeds |
| Confirm client-to-server reachability | R2 / R3 | `ping 123.1.1.1` | Ping succeeds |
| Confirm no debug remains enabled | R1 / PPPoE-AC | `show debugging` | No active PPP or PPPoE debugs remain |
# PPPoE_Verification_And_Failure_Isolation_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Disable all active debugging | R1 / R2 / R3 | `undebug all` | All debugging is disabled |
| 2 | Disable terminal debug display if previously enabled | R1 / R2 / R3 | `terminal no monitor` | Debug output stops displaying on remote terminal sessions |
| 3 | Confirm no debugs remain | R1 / R2 / R3 | `show debugging` | No PPP or PPPoE debugging remains active |
| 4 | Clear PPPoE sessions only if testing teardown and renegotiation | R2 / R3 | `clear pppoe all` | Client PPPoE session resets and attempts to reform |
| 5 | Recheck session state after clear | R1 / PPPoE-AC | `show pppoe session` | Session reforms if baseline configuration is still correct |
| 6 | Recheck client Dialer after clear | R2 / R3 | `show ip interface brief Dialer123` | Dialer123 returns to up/up with negotiated IP |
| 7 | Restore client interface if it was shut during testing | R2 / R3 | `configure terminal` then `interface FastEthernet0/0` then `no shutdown` | Client physical interface returns up/up |
| 8 | Restore server interface if it was shut during testing | R1 / PPPoE-AC | `configure terminal` then `interface FastEthernet0/0` then `no shutdown` | Server access interface returns up/up |
| 9 | Confirm the server still has PPPoE enabled | R1 / PPPoE-AC | `show running-config interface FastEthernet0/0` | `pppoe enable group tst` remains present |
| 10 | Save restored state if changes were intentionally kept | R1 / R2 / R3 | `write memory` | Restored working state is saved |
# PPPoE_Verification_And_Failure_Isolation_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| No PPPoE session appears on the server | Server access interface is down or shutdown | R1 / PPPoE-AC | `show ip interface brief FastEthernet0/0` | Apply `no shutdown` and fix the physical or switching path |
| No PPPoE session appears on the server | PPPoE server not enabled on the Ethernet interface | R1 / PPPoE-AC | `show running-config interface FastEthernet0/0` | Add `pppoe enable group tst` |
| No PPPoE session appears on the server | Client physical interface is down or shutdown | R2 / R3 | `show ip interface brief FastEthernet0/0` | Apply `no shutdown` and fix the physical or switching path |
| No PPPoE session appears on the server | Client physical interface is not bound to a dialer pool | R2 / R3 | `show running-config interface FastEthernet0/0` | Add `pppoe-client dial-pool-number 100` |
| Client Dialer stays down/down | Dialer pool mismatch | R2 / R3 | `show running-config interface Dialer123` and `show running-config interface FastEthernet0/0` | Match `dialer pool 100` with `pppoe-client dial-pool-number 100` |
| Client Dialer exists but cannot negotiate PPP | Missing `encapsulation ppp` | R2 / R3 | `show running-config interface Dialer123` | Add `encapsulation ppp` |
| PPPoE session appears but no usable Virtual-Access exists | bba-group is not bound to the Virtual-Template | R1 / PPPoE-AC | `show running-config | section bba-group` | Add `virtual-template 123` under `bba-group pppoe tst` |
| Virtual-Access exists but is not using the expected server IP | Virtual-Template is not unnumbered to Loopback123 | R1 / PPPoE-AC | `show running-config interface Virtual-Template123` | Add `ip unnumbered Loopback123` |
| LCP does not open | PPP negotiation problem | R1 / PPPoE-AC | `debug ppp negotiation` | Fix PPP settings on the Virtual-Template or client Dialer |
| Authentication fails after PPPoE session forms | PAP or CHAP credentials are wrong | R1 / R2 / R3 | `debug ppp authentication` | Correct the PAP sent username, CHAP hostname, or local username database |
| IPCP does not open | Server template lacks address pool reference | R1 / PPPoE-AC | `show running-config interface Virtual-Template123` | Add `peer default ip address pool tst-pool` |
| IPCP does not open | Local pool missing or pool name mismatch | R1 / PPPoE-AC | `show running-config | include ip local pool` | Create `ip local pool tst-pool 123.1.1.2 123.1.1.3` |
| First client works but second client fails | Local pool has too few addresses | R1 / PPPoE-AC | `show ip local pool` | Expand the local pool to one address per simultaneous client |
| Two clients do not appear separately | Duplicate client MAC addresses | R2 / R3 | `show interface FastEthernet0/0` | Use unique MAC addresses per PPPoE client |
| Session exists but client has no IP address | Client Dialer lacks `ip address negotiated` | R2 / R3 | `show running-config interface Dialer123` | Add `ip address negotiated` |
| Session exists and Dialer has IP but ping fails | Wrong target or broken PPP endpoint assumption | R2 / R3 | `show ip interface brief Dialer123` and `ping 123.1.1.1` | Confirm server loopback is 123.1.1.1 and Dialer has a 123.1.1.x address |
| Ping works but large TCP traffic fails | MTU or MSS not adjusted | R1 / R2 / R3 | `show running-config interface Virtual-Template123` and `show running-config interface Dialer123` | Add `mtu 1492` and `ip tcp adjust-mss 1452` |
| Debug output floods the terminal | Debugs left running | R1 / R2 / R3 | `show debugging` | Run `undebug all` |
| Troubleshooting starts with ping and gives no clue | Wrong isolation order | R1 / R2 / R3 | `show pppoe session` then `show interface virtual-access <id>` then `show ip interface brief Dialer123` | Work bottom up: physical, discovery, session, PPP, IPCP, then ping |
# Index
PPPoE_Verification_And_Failure_Isolation.md
# PPPoE_Verification_And_Failure_Isolation
# PPPoE_Verification_And_Failure_Isolation_Related_Labs
# PPPoE_Verification_And_Failure_Isolation_Mental_Model
# PPPoE_Verification_And_Failure_Isolation_Configuration_Checklist
# PPPoE_Verification_And_Failure_Isolation_Skeleton
# PPPoE_Verification_And_Failure_Isolation_Verification_Commands
# PPPoE_Verification_And_Failure_Isolation_Rollback
# PPPoE_Verification_And_Failure_Isolation_Failure_Checks
# Index

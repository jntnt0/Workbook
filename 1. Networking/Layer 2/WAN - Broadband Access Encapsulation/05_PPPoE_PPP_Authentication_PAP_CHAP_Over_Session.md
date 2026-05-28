PPPoE_PPP_Authentication_PAP_CHAP_Over_Session.md
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session_Related_Labs
- pppoe-server-client-authentication-final
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session_Mental_Model
| Concept | Operational Meaning |
|---|---|
| PPPoE discovery is not authentication | PADI, PADO, PADR, and PADS only build the PPPoE session. They do not prove user identity |
| PPP session stage | After PADS assigns a Session ID, normal PPP runs inside the PPPoE session |
| LCP authentication negotiation | PAP or CHAP is negotiated during PPP LCP, after PPPoE discovery succeeds |
| PAP | The client sends a username and password to the authenticator |
| PAP authenticator | The side with `ppp authentication pap`, usually the PPPoE access concentrator Virtual-Template |
| PAP sender | The side with `ppp pap sent-username <name> password <password>`, usually the PPPoE client Dialer |
| CHAP | The authenticator sends a challenge. The peer responds using a hash based on the shared secret |
| CHAP authenticator | The side with `ppp authentication chap`, usually the PPPoE access concentrator Virtual-Template |
| CHAP username lookup | Each side must have a username entry matching the peer name it receives in CHAP |
| Virtual-Template placement | Server-side PPP authentication belongs under the Virtual-Template so every cloned Virtual-Access session inherits it |
| Dialer placement | Client-side PAP or CHAP identity belongs under the Dialer because the Dialer is the PPP client interface |
| Username database | Local `username <peer> password <secret>` entries are used to validate PPP credentials |
| Session failure boundary | If `show pppoe session` shows a session but IPCP does not open, troubleshoot PPP authentication before IP reachability |
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the PPPoE server baseline exists | R1 / PPPoE-AC | `show running-config | section bba-group` | `bba-group pppoe tst` references `virtual-template 123` |
| 2 | Confirm the server Virtual-Template exists | R1 / PPPoE-AC | `show running-config interface Virtual-Template123` | Template contains `ip unnumbered Loopback123` and `peer default ip address pool tst-pool` |
| 3 | Confirm the client Dialer exists | R2 / PPPoE-Client | `show running-config interface Dialer123` | Dialer contains `encapsulation ppp`, `ip address negotiated`, and `dialer pool 100` |
| 4 | Confirm the physical PPPoE client binding exists | R2 / PPPoE-Client | `show running-config interface FastEthernet0/0` | Interface contains `pppoe-client dial-pool-number 100` |
| 5 | Confirm the unauthenticated PPPoE session can form before adding authentication | R1 / PPPoE-AC | `show pppoe session` | Session appears before authentication is enforced |
| 6 | Choose PAP or CHAP for the lab | R1 / R2 | `show running-config interface Virtual-Template123` | Decide whether the Virtual-Template will use PAP or CHAP |
| 7 | Enter global configuration mode on the access concentrator | R1 / PPPoE-AC | `configure terminal` | R1 enters configuration mode |
| 8 | Add the client credential to the server local username database for PAP | R1 / PPPoE-AC | `username R2 password Cisco` | R1 can validate PAP username `R2` with password `Cisco` |
| 9 | Enter the server Virtual-Template | R1 / PPPoE-AC | `interface Virtual-Template123` | Server PPP template is selected |
| 10 | Require PAP authentication from PPPoE clients | R1 / PPPoE-AC | `ppp authentication pap` | All cloned Virtual-Access interfaces require PAP during PPP LCP |
| 11 | Exit server configuration mode | R1 / PPPoE-AC | `end` | R1 returns to privileged EXEC mode |
| 12 | Enter global configuration mode on the PPPoE client | R2 / PPPoE-Client | `configure terminal` | R2 enters configuration mode |
| 13 | Enter the client Dialer interface | R2 / PPPoE-Client | `interface Dialer123` | Client PPP interface is selected |
| 14 | Configure the PAP identity the client sends to the server | R2 / PPPoE-Client | `ppp pap sent-username R2 password Cisco` | R2 sends username `R2` and password `Cisco` when R1 requests PAP |
| 15 | Exit client configuration mode | R2 / PPPoE-Client | `end` | R2 returns to privileged EXEC mode |
| 16 | Clear or flap the PPPoE session so authentication renegotiates | R2 / PPPoE-Client | `clear pppoe all` | PPPoE session tears down and reforms with PAP |
| 17 | Verify PAP authentication succeeds | R1 / PPPoE-AC | `debug ppp authentication` | Debug shows PAP authentication success or AUTH-ACK |
| 18 | Confirm the PPPoE session is active after PAP | R1 / PPPoE-AC | `show pppoe session` | Session appears in locally terminated PTA state |
| 19 | Confirm PPP and IPCP opened after PAP | R1 / PPPoE-AC | `show interface virtual-access <id>` | Output shows `LCP Open` and `Open: IPCP` |
| 20 | Confirm client Dialer received an IP address after PAP | R2 / PPPoE-Client | `show ip interface brief Dialer123` | Dialer123 is up/up with a negotiated address |
| 21 | Test reachability after PAP | R2 / PPPoE-Client | `ping 123.1.1.1` | Ping succeeds across the authenticated PPPoE session |
| 22 | Remove PAP before testing CHAP if switching methods | R1 / PPPoE-AC | `interface Virtual-Template123` then `no ppp authentication pap` | PAP is removed from the server template |
| 23 | Remove PAP sent credentials before testing CHAP if switching methods | R2 / PPPoE-Client | `interface Dialer123` then `no ppp pap sent-username R2 password Cisco` | Client no longer sends PAP credentials |
| 24 | Add the client username to the server database for CHAP | R1 / PPPoE-AC | `username R2 password Cisco` | R1 can validate CHAP responses from R2 |
| 25 | Add the server username to the client database for CHAP | R2 / PPPoE-Client | `username R1 password Cisco` | R2 can respond to CHAP challenges from R1 |
| 26 | Enter the server Virtual-Template | R1 / PPPoE-AC | `interface Virtual-Template123` | Server PPP template is selected |
| 27 | Require CHAP authentication from PPPoE clients | R1 / PPPoE-AC | `ppp authentication chap` | All cloned Virtual-Access interfaces require CHAP during PPP LCP |
| 28 | Enter the client Dialer interface | R2 / PPPoE-Client | `interface Dialer123` | Client PPP interface is selected |
| 29 | Set the CHAP hostname only if the lab requires a specific client identity | R2 / PPPoE-Client | `ppp chap hostname R2` | R2 uses `R2` as its CHAP response name |
| 30 | Clear or flap the PPPoE session so CHAP renegotiates | R2 / PPPoE-Client | `clear pppoe all` | PPPoE session tears down and reforms with CHAP |
| 31 | Verify CHAP authentication succeeds | R1 / PPPoE-AC | `debug ppp authentication` | Debug shows CHAP challenge, response, and success |
| 32 | Confirm the PPPoE session is active after CHAP | R1 / PPPoE-AC | `show pppoe session` | Session appears in locally terminated PTA state |
| 33 | Confirm PPP and IPCP opened after CHAP | R1 / PPPoE-AC | `show interface virtual-access <id>` | Output shows `LCP Open` and `Open: IPCP` |
| 34 | Confirm client Dialer received an IP address after CHAP | R2 / PPPoE-Client | `show ip interface brief Dialer123` | Dialer123 is up/up with a negotiated address |
| 35 | Test reachability after CHAP | R2 / PPPoE-Client | `ping 123.1.1.1` | Ping succeeds across the authenticated PPPoE session |
| 36 | Disable debugging after validation | R1 / PPPoE-AC | `undebug all` | PPP authentication debugging is disabled |
| 37 | Save the working configuration | R1 / R2 | `write memory` | Authentication configuration is saved |
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session_PAP_Skeleton
! R1 PPPoE ACCESS CONCENTRATOR WITH PAP
configure terminal
!
username R2 password Cisco
!
interface Virtual-Template123
 ppp authentication pap
 end
!
write memory
! R2 PPPoE CLIENT WITH PAP
configure terminal
!
interface Dialer123
 ppp pap sent-username R2 password Cisco
 end
!
write memory
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session_CHAP_Skeleton
! R1 PPPoE ACCESS CONCENTRATOR WITH CHAP
configure terminal
!
username R2 password Cisco
!
interface Virtual-Template123
 ppp authentication chap
 end
!
write memory
! R2 PPPoE CLIENT WITH CHAP
configure terminal
!
username R1 password Cisco
!
interface Dialer123
 ppp chap hostname R2
 end
!
write memory
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session_Full_Baseline_With_PAP
! R1 PPPoE ACCESS CONCENTRATOR
configure terminal
!
interface Loopback123
 ip address 123.1.1.1 255.255.255.0
 exit
!
interface Virtual-Template123
 ip unnumbered Loopback123
 peer default ip address pool tst-pool
 mtu 1492
 ip tcp adjust-mss 1452
 ppp authentication pap
 exit
!
username R2 password Cisco
!
ip local pool tst-pool 123.1.1.2 123.1.1.3
!
bba-group pppoe tst
 virtual-template 123
 exit
!
interface FastEthernet0/0
 mac-address 0000.1111.1111
 pppoe enable group tst
 no shutdown
 end
!
write memory
! R2 PPPoE CLIENT
configure terminal
!
interface Dialer123
 encapsulation ppp
 ip address negotiated
 mtu 1492
 ip tcp adjust-mss 1452
 dialer pool 100
 ppp pap sent-username R2 password Cisco
 exit
!
interface FastEthernet0/0
 mac-address 0000.2222.2222
 pppoe-client dial-pool-number 100
 no shutdown
 end
!
write memory
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session_Full_Baseline_With_CHAP
! R1 PPPoE ACCESS CONCENTRATOR
configure terminal
!
interface Loopback123
 ip address 123.1.1.1 255.255.255.0
 exit
!
interface Virtual-Template123
 ip unnumbered Loopback123
 peer default ip address pool tst-pool
 mtu 1492
 ip tcp adjust-mss 1452
 ppp authentication chap
 exit
!
username R2 password Cisco
!
ip local pool tst-pool 123.1.1.2 123.1.1.3
!
bba-group pppoe tst
 virtual-template 123
 exit
!
interface FastEthernet0/0
 mac-address 0000.1111.1111
 pppoe enable group tst
 no shutdown
 end
!
write memory
! R2 PPPoE CLIENT
configure terminal
!
username R1 password Cisco
!
interface Dialer123
 encapsulation ppp
 ip address negotiated
 mtu 1492
 ip tcp adjust-mss 1452
 dialer pool 100
 ppp chap hostname R2
 exit
!
interface FastEthernet0/0
 mac-address 0000.2222.2222
 pppoe-client dial-pool-number 100
 no shutdown
 end
!
write memory
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session_Verification_Commands
| Verification Goal | Device | Command | Good Output |
|---|---|---|---|
| Confirm server Virtual-Template authentication method | R1 / PPPoE-AC | `show running-config interface Virtual-Template123` | Output shows `ppp authentication pap` or `ppp authentication chap` |
| Confirm PAP client sent credentials | R2 / PPPoE-Client | `show running-config interface Dialer123` | Output shows `ppp pap sent-username R2 password Cisco` |
| Confirm CHAP server username database | R1 / PPPoE-AC | `show running-config | include username R2` | Output shows `username R2 password Cisco` |
| Confirm CHAP client username database | R2 / PPPoE-Client | `show running-config | include username R1` | Output shows `username R1 password Cisco` |
| Confirm CHAP client name override if used | R2 / PPPoE-Client | `show running-config interface Dialer123` | Output shows `ppp chap hostname R2` |
| Confirm PPPoE session exists | R1 / PPPoE-AC | `show pppoe session` | Session appears in locally terminated PTA state |
| Confirm server-side Virtual-Access clone | R1 / PPPoE-AC | `show interface virtual-access <id>` | Output shows the interface cloned from Virtual-Template123 |
| Confirm PPP LCP opened | R1 / PPPoE-AC | `show interface virtual-access <id>` | Output shows `LCP Open` |
| Confirm IPCP opened | R1 / PPPoE-AC | `show interface virtual-access <id>` | Output shows `Open: IPCP` |
| Confirm PAP success | R1 / PPPoE-AC | `debug ppp authentication` | Output shows PAP AUTH-REQ followed by AUTH-ACK or login PASS |
| Confirm CHAP success | R1 / PPPoE-AC | `debug ppp authentication` | Output shows CHAP challenge, response, and success |
| Confirm PAP failure counters | R1 / PPPoE-AC | `show ppp statistics | include Counter|PAP` | PAP failures do not increase after correct credentials are applied |
| Confirm Dialer state | R2 / PPPoE-Client | `show ip interface brief Dialer123` | Dialer123 is up/up |
| Confirm client negotiated address | R2 / PPPoE-Client | `show ip interface brief Dialer123` | Dialer123 has an IPCP-learned address |
| Confirm authenticated path | R2 / PPPoE-Client | `ping 123.1.1.1` | Ping succeeds |
| Disable debugging | R1 / PPPoE-AC | `undebug all` | Debugging is disabled |
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter global configuration mode on the access concentrator | R1 / PPPoE-AC | `configure terminal` | R1 enters configuration mode |
| 2 | Enter the server Virtual-Template | R1 / PPPoE-AC | `interface Virtual-Template123` | Server PPP template is selected |
| 3 | Remove PAP authentication if configured | R1 / PPPoE-AC | `no ppp authentication pap` | Server no longer requires PAP |
| 4 | Remove CHAP authentication if configured | R1 / PPPoE-AC | `no ppp authentication chap` | Server no longer requires CHAP |
| 5 | Exit the Virtual-Template | R1 / PPPoE-AC | `exit` | R1 returns to global configuration mode |
| 6 | Remove the server username entry for R2 if only used for this lab | R1 / PPPoE-AC | `no username R2` | R1 local credential for R2 is removed |
| 7 | Exit server configuration mode | R1 / PPPoE-AC | `end` | R1 returns to privileged EXEC mode |
| 8 | Enter global configuration mode on the client | R2 / PPPoE-Client | `configure terminal` | R2 enters configuration mode |
| 9 | Enter the client Dialer | R2 / PPPoE-Client | `interface Dialer123` | Client PPP interface is selected |
| 10 | Remove PAP sent credentials if configured | R2 / PPPoE-Client | `no ppp pap sent-username R2 password Cisco` | Client no longer sends PAP credentials |
| 11 | Remove CHAP hostname override if configured | R2 / PPPoE-Client | `no ppp chap hostname R2` | Client returns to default CHAP hostname behavior |
| 12 | Exit the Dialer | R2 / PPPoE-Client | `exit` | R2 returns to global configuration mode |
| 13 | Remove the client username entry for R1 if only used for this lab | R2 / PPPoE-Client | `no username R1` | R2 local credential for R1 is removed |
| 14 | Exit client configuration mode | R2 / PPPoE-Client | `end` | R2 returns to privileged EXEC mode |
| 15 | Clear existing PPPoE sessions | R2 / PPPoE-Client | `clear pppoe all` | Existing PPPoE session is torn down |
| 16 | Confirm authentication has been removed from the template | R1 / PPPoE-AC | `show running-config interface Virtual-Template123` | No `ppp authentication pap` or `ppp authentication chap` remains |
| 17 | Confirm PPPoE session can reform without authentication if baseline remains | R1 / PPPoE-AC | `show pppoe session` | Session reforms without PPP authentication requirement |
| 18 | Save rollback state | R1 / R2 | `write memory` | Rollback configuration is saved |
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| `show pppoe session` shows no session | Discovery or physical binding is broken, not PPP authentication | R1 / R2 | `show running-config interface FastEthernet0/0` | Fix `pppoe enable group tst` on R1 or `pppoe-client dial-pool-number 100` on R2 |
| PPPoE session appears but Dialer has no IP | PPP authentication failed before IPCP opened | R1 | `debug ppp authentication` | Fix PAP or CHAP credentials |
| PAP debug shows AUTH-NAK | Server does not have a matching username and password for the client | R1 | `show running-config | include username R2` | Add `username R2 password Cisco` |
| PAP debug shows failed login | Client sends wrong PAP username or password | R2 | `show running-config interface Dialer123` | Correct `ppp pap sent-username R2 password Cisco` |
| PAP never starts | Server Virtual-Template does not require PAP | R1 | `show running-config interface Virtual-Template123` | Add `ppp authentication pap` |
| PAP is configured on the wrong interface | Authentication configured on Ethernet instead of the Virtual-Template | R1 | `show running-config interface FastEthernet0/0` | Move PPP authentication to `interface Virtual-Template123` |
| CHAP debug shows failure after response | Server username does not match the client's CHAP name | R1 | `debug ppp authentication` | Add or correct `username R2 password Cisco` |
| CHAP debug shows client cannot respond correctly | Client does not have a username entry for the server challenge name | R2 | `show running-config | include username R1` | Add `username R1 password Cisco` |
| CHAP uses unexpected client name | Client hostname does not match the username expected on the server | R2 | `show running-config interface Dialer123` | Add `ppp chap hostname R2` or change the server username entry |
| CHAP never starts | Server Virtual-Template does not require CHAP | R1 | `show running-config interface Virtual-Template123` | Add `ppp authentication chap` |
| PAP works but CHAP fails | PAP sent-username logic was reused incorrectly for CHAP | R1 / R2 | `debug ppp authentication` | Use CHAP username database matching instead of PAP sent credentials |
| CHAP works on one client but not another | Server lacks a unique username entry for each client identity | R1 | `show running-config | include username` | Add one username entry per client CHAP name |
| Two-client authentication fails for second client | Duplicate CHAP hostname on both clients | R2 / R3 | `show running-config interface Dialer123` | Use unique CHAP names such as R2 and R3 |
| Authentication succeeds but large TCP traffic fails | MTU or MSS issue, not authentication | R1 / R2 | `show running-config interface Virtual-Template123` and `show running-config interface Dialer123` | Keep `mtu 1492` and `ip tcp adjust-mss 1452` |
| Ping fails even though authentication succeeds | IPCP did not assign an address or address pool is broken | R1 / R2 | `show ip local pool` and `show ip interface brief Dialer123` | Fix `peer default ip address pool tst-pool` and `ip local pool tst-pool` |
| Debug output floods the console | PPP authentication debugging left enabled | R1 | `show debugging` | Run `undebug all` |
# Index
PPPoE_PPP_Authentication_PAP_CHAP_Over_Session.md
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session_Related_Labs
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session_Mental_Model
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session_Configuration_Checklist
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session_PAP_Skeleton
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session_CHAP_Skeleton
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session_Full_Baseline_With_PAP
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session_Full_Baseline_With_CHAP
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session_Verification_Commands
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session_Rollback
# PPPoE_PPP_Authentication_PAP_CHAP_Over_Session_Failure_Checks
# Index

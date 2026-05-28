PPPoE_Client_Dialer_Profile_And_Physical_Binding.md
# PPPoE_Client_Dialer_Profile_And_Physical_Binding
# PPPoE_Client_Dialer_Profile_And_Physical_Binding_Related_Labs
- pppoe-server-final
- pppoe-server-two-clients-final
- pppoe-server-client-authentication-final
# PPPoE_Client_Dialer_Profile_And_Physical_Binding_Mental_Model
| Concept | Operational Meaning |
|---|---|
| PPPoE client | The router that initiates PPPoE discovery toward the access concentrator |
| Dialer interface | The logical PPP interface where PPP encapsulation, IPCP address negotiation, MTU, MSS, and dialer pool selection are configured |
| Physical Ethernet interface | The client-facing transport interface that sends PPPoE discovery frames and binds the PPPoE client instance to a dialer pool |
| Dialer profile | Another name for the Dialer interface model. It holds the PPP client behavior but is not directly tied to one physical interface |
| Dialer pool | The glue between the Dialer interface and the PPPoE client instance on the Ethernet interface |
| Pool number match | The Dialer `dialer pool <number>` must match the physical interface `pppoe-client dial-pool-number <number>` |
| PPP encapsulation | Dialer interfaces need `encapsulation ppp` so PPP can run after PPPoE discovery succeeds |
| IPCP negotiated address | `ip address negotiated` tells the client to accept its IP address from the PPPoE server through PPP IPCP |
| PPPoE discovery | The physical Ethernet interface sends PADI, receives PADO, sends PADR, and receives PADS |
| PPPoE session stage | After PADS, the Dialer carries PPP over the PPPoE session |
| Virtual-Access binding | IOS dynamically binds a Virtual-Access interface to the Dialer profile when the PPPoE session comes up |
| MTU 1492 | PPPoE plus PPP overhead consumes 8 bytes, so the client Dialer should not use a 1500-byte MTU |
| MSS 1452 | TCP MSS should be clamped so TCP payloads do not exceed the PPPoE-safe MTU |
# PPPoE_Client_Dialer_Profile_And_Physical_Binding_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the client physical interface exists | PPPoE-Client | `show ip interface brief` | Client Ethernet interface is present |
| 2 | Confirm the PPPoE server side is expected to be reachable on the same Ethernet segment | PPPoE-Client | `show cdp neighbors` | Neighbor visibility confirms the local lab link if CDP is enabled |
| 3 | Enter global configuration mode | PPPoE-Client | `configure terminal` | Device enters configuration mode |
| 4 | Create the client Dialer interface | PPPoE-Client | `interface Dialer123` | Dialer123 is created |
| 5 | Set the Dialer interface to PPP encapsulation | PPPoE-Client | `encapsulation ppp` | Dialer123 uses PPP instead of the default encapsulation |
| 6 | Configure the Dialer to receive its address through IPCP | PPPoE-Client | `ip address negotiated` | Client expects the server to assign the Dialer IP address |
| 7 | Set PPPoE-safe MTU on the Dialer | PPPoE-Client | `mtu 1492` | Dialer123 accounts for PPPoE plus PPP overhead |
| 8 | Clamp TCP MSS for PPPoE transport | PPPoE-Client | `ip tcp adjust-mss 1452` | TCP sessions avoid oversized segments across PPPoE |
| 9 | Bind the Dialer to dialer pool 100 | PPPoE-Client | `dialer pool 100` | Dialer123 can select a PPPoE client instance from pool 100 |
| 10 | Move to the physical Ethernet interface facing the access concentrator | PPPoE-Client | `interface FastEthernet0/0` | Client physical PPPoE transport interface is selected |
| 11 | Set the lab MAC address if required by the topology | PPPoE-Client | `mac-address 0000.2222.2222` | Client uses the expected lab MAC address |
| 12 | Bind the physical PPPoE client instance to the same dialer pool | PPPoE-Client | `pppoe-client dial-pool-number 100` | Ethernet interface offers PPPoE client services to Dialer123 |
| 13 | Bring up the physical Ethernet interface | PPPoE-Client | `no shutdown` | Ethernet interface begins PPPoE discovery |
| 14 | Exit to privileged EXEC mode | PPPoE-Client | `end` | Device exits configuration mode |
| 15 | Watch for the Dialer binding message | PPPoE-Client | `terminal monitor` | Console may show `Interface Vi1 bound to profile Di123` |
| 16 | Confirm the client Dialer came up | PPPoE-Client | `show ip interface brief Dialer123` | Dialer123 is up/up |
| 17 | Confirm the client received an IPCP address | PPPoE-Client | `show ip interface brief Dialer123` | Dialer123 shows an address assigned by IPCP |
| 18 | Confirm the server sees the client session | PPPoE-AC | `show pppoe session` | Server shows a PPPoE session for the client MAC address |
| 19 | Confirm server-side Virtual-Access was created | PPPoE-AC | `show interface virtual-access <id>` | Virtual-Access is up/up and cloned from the server Virtual-Template |
| 20 | Test reachability to the access concentrator PPP endpoint | PPPoE-Client | `ping 123.1.1.1` | Ping succeeds across the PPPoE session |
| 21 | Save the working client configuration | PPPoE-Client | `write memory` | Client PPPoE configuration is saved |
# PPPoE_Client_Dialer_Profile_And_Physical_Binding_Skeleton
! PPPoE CLIENT
configure terminal
!
interface Dialer123
 encapsulation ppp
 ip address negotiated
 mtu 1492
 ip tcp adjust-mss 1452
 dialer pool 100
 exit
!
interface FastEthernet0/0
 mac-address 0000.2222.2222
 pppoe-client dial-pool-number 100
 no shutdown
 end
!
write memory
# PPPoE_Client_Dialer_Profile_And_Physical_Binding_Second_Client_Skeleton
! SECOND PPPoE CLIENT
configure terminal
!
interface Dialer123
 encapsulation ppp
 ip address negotiated
 mtu 1492
 ip tcp adjust-mss 1452
 dialer pool 100
 exit
!
interface FastEthernet0/0
 mac-address 0000.3333.3333
 pppoe-client dial-pool-number 100
 no shutdown
 end
!
write memory
# PPPoE_Client_Dialer_Profile_And_Physical_Binding_Verification_Commands
| Verification Goal | Device | Command | Good Output |
|---|---|---|---|
| Confirm client physical interface exists | PPPoE-Client | `show ip interface brief` | FastEthernet0/0 or equivalent Ethernet interface is listed |
| Confirm client physical interface is up | PPPoE-Client | `show ip interface brief FastEthernet0/0` | Interface is up/up |
| Confirm Dialer interface exists | PPPoE-Client | `show ip interface brief Dialer123` | Dialer123 is listed |
| Confirm Dialer is up after PPPoE session establishment | PPPoE-Client | `show ip interface brief Dialer123` | Dialer123 is up/up |
| Confirm IPCP assigned an address | PPPoE-Client | `show ip interface brief Dialer123` | Dialer123 has an address such as 123.1.1.2 |
| Confirm client Dialer configuration | PPPoE-Client | `show running-config interface Dialer123` | Output shows `encapsulation ppp`, `ip address negotiated`, `mtu 1492`, `ip tcp adjust-mss 1452`, and `dialer pool 100` |
| Confirm physical PPPoE binding | PPPoE-Client | `show running-config interface FastEthernet0/0` | Output shows `pppoe-client dial-pool-number 100` |
| Confirm Dialer pool match | PPPoE-Client | `show running-config interface Dialer123` and `show running-config interface FastEthernet0/0` | Dialer pool number matches the PPPoE client dial-pool number |
| Confirm server sees the session | PPPoE-AC | `show pppoe session` | Server shows the client MAC address and a PPPoE Session ID |
| Confirm server-side Virtual-Access interface | PPPoE-AC | `show interface virtual-access <id>` | Virtual-Access is up/up |
| Confirm PPP opened on server-side Virtual-Access | PPPoE-AC | `show interface virtual-access <id>` | Output shows `Encapsulation PPP, LCP Open` |
| Confirm IPCP opened on server-side Virtual-Access | PPPoE-AC | `show interface virtual-access <id>` | Output shows `Open: IPCP` |
| Confirm client reachability to server PPP endpoint | PPPoE-Client | `ping 123.1.1.1` | Ping succeeds |
| Confirm second client session when used | PPPoE-AC | `show pppoe session` | Server shows two locally terminated PTA sessions |
# PPPoE_Client_Dialer_Profile_And_Physical_Binding_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter global configuration mode | PPPoE-Client | `configure terminal` | Device enters configuration mode |
| 2 | Enter the physical Ethernet interface | PPPoE-Client | `interface FastEthernet0/0` | Client physical PPPoE transport interface is selected |
| 3 | Remove PPPoE client binding | PPPoE-Client | `no pppoe-client dial-pool-number 100` | Physical interface stops initiating PPPoE sessions for dialer pool 100 |
| 4 | Remove lab MAC address if it was only used for this lab | PPPoE-Client | `no mac-address 0000.2222.2222` | Interface returns to default hardware MAC behavior |
| 5 | Exit interface configuration mode | PPPoE-Client | `exit` | Device returns to global configuration mode |
| 6 | Remove the Dialer interface | PPPoE-Client | `no interface Dialer123` | Client logical PPP interface is removed |
| 7 | Return to privileged EXEC mode | PPPoE-Client | `end` | Device exits configuration mode |
| 8 | Confirm Dialer is removed | PPPoE-Client | `show ip interface brief Dialer123` | Dialer123 is no longer present |
| 9 | Confirm physical interface no longer has PPPoE client binding | PPPoE-Client | `show running-config interface FastEthernet0/0` | `pppoe-client dial-pool-number 100` is absent |
| 10 | Confirm server session clears | PPPoE-AC | `show pppoe session` | Removed client session is gone |
| 11 | Save rollback state | PPPoE-Client | `write memory` | Rollback configuration is saved |
# PPPoE_Client_Dialer_Profile_And_Physical_Binding_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Dialer123 never comes up | Physical Ethernet interface is shutdown or down/down | PPPoE-Client | `show ip interface brief FastEthernet0/0` | Apply `no shutdown` and fix the physical or switching path |
| Dialer123 stays down/down | Dialer pool number mismatch | PPPoE-Client | `show running-config interface Dialer123` and `show running-config interface FastEthernet0/0` | Make `dialer pool 100` match `pppoe-client dial-pool-number 100` |
| PPPoE discovery never starts | Missing PPPoE client command under the physical interface | PPPoE-Client | `show running-config interface FastEthernet0/0` | Add `pppoe-client dial-pool-number 100` |
| Dialer exists but PPP cannot negotiate | Missing PPP encapsulation on Dialer | PPPoE-Client | `show running-config interface Dialer123` | Add `encapsulation ppp` |
| Dialer comes up but has no IP address | Missing negotiated IP command | PPPoE-Client | `show running-config interface Dialer123` | Add `ip address negotiated` |
| Dialer gets no IP address | Server pool or IPCP is broken | PPPoE-AC | `show ip local pool` | Fix or expand the server local pool |
| Server shows no session for the client | Client is on wrong Ethernet segment or server is not listening | PPPoE-Client / PPPoE-AC | `show ip interface brief` and `show running-config interface FastEthernet0/0` | Fix physical topology or server `pppoe enable group` |
| Server sees one client but not the second | Duplicate or wrong lab MAC address | PPPoE-Client | `show interface FastEthernet0/0` | Use unique MACs such as `0000.2222.2222` and `0000.3333.3333` |
| PPPoE session exists but ping fails | Dialer has no IPCP address or wrong negotiated state | PPPoE-Client | `show ip interface brief Dialer123` | Fix server address pool and client `ip address negotiated` |
| Large TCP transfers fail while ping works | MTU or MSS missing on Dialer | PPPoE-Client | `show running-config interface Dialer123` | Add `mtu 1492` and `ip tcp adjust-mss 1452` |
| Session forms but authentication lab fails | Client authentication settings are missing or wrong | PPPoE-Client | `show running-config interface Dialer123` | Add the required PAP or CHAP client settings for the authentication note |
| Server shows Virtual-Access up but client appears unstable | PPPoE session forms but PPP control protocols are not stable | PPPoE-AC | `show interface virtual-access <id>` | Confirm `LCP Open` and `Open: IPCP`; fix PPP settings |
| Client configuration looks correct but no session appears | Server bba-group or Virtual-Template is broken | PPPoE-AC | `show running-config | section bba-group` | Fix server-side bba-group to Virtual-Template binding |
# Index
PPPoE_Client_Dialer_Profile_And_Physical_Binding.md
# PPPoE_Client_Dialer_Profile_And_Physical_Binding
# PPPoE_Client_Dialer_Profile_And_Physical_Binding_Related_Labs
# PPPoE_Client_Dialer_Profile_And_Physical_Binding_Mental_Model
# PPPoE_Client_Dialer_Profile_And_Physical_Binding_Configuration_Checklist
# PPPoE_Client_Dialer_Profile_And_Physical_Binding_Skeleton
# PPPoE_Client_Dialer_Profile_And_Physical_Binding_Second_Client_Skeleton
# PPPoE_Client_Dialer_Profile_And_Physical_Binding_Verification_Commands
# PPPoE_Client_Dialer_Profile_And_Physical_Binding_Rollback
# PPPoE_Client_Dialer_Profile_And_Physical_Binding_Failure_Checks
# Index

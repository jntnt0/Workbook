PPPoE_Discovery_And_Session_Formation.md
# PPPoE_Discovery_And_Session_Formation
# PPPoE_Discovery_And_Session_Formation_Related_Labs
- pppoe-server-final
- pppoe-server-two-clients-final
- pppoe-server-client-authentication-final
# PPPoE_Discovery_And_Session_Formation_Mental_Model
| Concept | Operational Meaning |
|---|---|
| PPPoE access concentrator | The server-side router that listens for PPPoE clients on an Ethernet interface |
| PPPoE client | The router that starts discovery and requests a PPPoE session through its Ethernet interface |
| Discovery stage | The client finds and selects an access concentrator before normal PPP begins |
| PADI | Client broadcast asking for available PPPoE access concentrators |
| PADO | Access concentrator unicast offer back to the client |
| PADR | Client unicast request to the selected access concentrator |
| PADS | Access concentrator confirmation that assigns the PPPoE Session ID |
| PPP session stage | After PADS, normal PPP runs inside Ethernet plus PPPoE encapsulation |
| Session ID | The identifier used by the access concentrator to separate each client PPPoE session |
| Virtual-Template | Server-side template used to spawn per-client Virtual-Access interfaces |
| Virtual-Access | Dynamic interface created per PPPoE client session |
| Dialer interface | Client-side logical PPP interface used to carry the negotiated IP session |
| Dialer pool | Binding mechanism between the client Dialer interface and the physical Ethernet PPPoE client instance |
| MTU 1492 | PPPoE plus PPP consumes 8 bytes, reducing usable Ethernet payload from 1500 to 1492 |
| MSS 1452 | TCP MSS adjusted to avoid fragmentation across the PPPoE path |
# PPPoE_Discovery_And_Session_Formation_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm Ethernet path exists between PPPoE server and client | PPPoE-AC / PPPoE-Client | `show ip interface brief` | Physical Ethernet interfaces are present and can be brought up/up |
| 2 | Create the server loopback used as the unnumbered PPP endpoint | PPPoE-AC | `interface Loopback123` | Loopback interface is created |
| 3 | Assign the server PPP endpoint address | PPPoE-AC | `ip address 123.1.1.1 255.255.255.0` | Server has the address clients will ping after PPPoE comes up |
| 4 | Create the server Virtual-Template | PPPoE-AC | `interface Virtual-Template123` | Template exists for dynamic Virtual-Access interfaces |
| 5 | Borrow the loopback address for PPPoE client sessions | PPPoE-AC | `ip unnumbered Loopback123` | Virtual-Access interfaces inherit 123.1.1.1 as the server-side PPP endpoint |
| 6 | Assign client addresses from a local pool | PPPoE-AC | `peer default ip address pool tst-pool` | PPP IPCP will hand client addresses from the named local pool |
| 7 | Set PPPoE-safe MTU on the template | PPPoE-AC | `mtu 1492` | PPPoE sessions avoid exceeding standard Ethernet payload after PPPoE overhead |
| 8 | Clamp TCP MSS for PPPoE | PPPoE-AC | `ip tcp adjust-mss 1452` | TCP sessions avoid oversized segments across PPPoE |
| 9 | Create the local address pool for clients | PPPoE-AC | `ip local pool tst-pool 123.1.1.2 123.1.1.3` | Client addresses are available without DHCP |
| 10 | Create the PPPoE server instance | PPPoE-AC | `bba-group pppoe tst` | Server PPPoE bba-group exists |
| 11 | Bind the bba-group to the Virtual-Template | PPPoE-AC | `virtual-template 123` | New PPPoE clients clone Virtual-Access interfaces from Virtual-Template123 |
| 12 | Enter the Ethernet interface facing clients | PPPoE-AC | `interface FastEthernet0/0` | Server is positioned on the client-facing Ethernet segment |
| 13 | Enable PPPoE server discovery on the Ethernet interface | PPPoE-AC | `pppoe enable group tst` | Interface listens for client PADI discovery frames |
| 14 | Bring up the server Ethernet interface | PPPoE-AC | `no shutdown` | Server Ethernet interface is enabled |
| 15 | Create the client Dialer interface | PPPoE-Client | `interface Dialer123` | Client logical PPP interface exists |
| 16 | Set PPP encapsulation on the Dialer | PPPoE-Client | `encapsulation ppp` | Dialer is ready for PPP negotiation after PPPoE discovery |
| 17 | Negotiate the client IP address through PPP IPCP | PPPoE-Client | `ip address negotiated` | Client expects address assignment from the access concentrator |
| 18 | Set PPPoE-safe MTU on the client Dialer | PPPoE-Client | `mtu 1492` | Client PPP interface accounts for PPPoE overhead |
| 19 | Clamp client TCP MSS for PPPoE | PPPoE-Client | `ip tcp adjust-mss 1452` | TCP sessions avoid fragmentation across the PPPoE session |
| 20 | Bind the Dialer to a dialer pool | PPPoE-Client | `dialer pool 100` | Dialer can use a matching PPPoE client instance from pool 100 |
| 21 | Enter the client Ethernet interface facing the access concentrator | PPPoE-Client | `interface FastEthernet0/0` | Client is positioned on the PPPoE discovery segment |
| 22 | Bind the physical interface to the Dialer pool | PPPoE-Client | `pppoe-client dial-pool-number 100` | Ethernet interface starts PPPoE discovery for Dialer123 |
| 23 | Bring up the client Ethernet interface | PPPoE-Client | `no shutdown` | Client sends PADI and attempts PPPoE session establishment |
| 24 | Confirm server sees a PPPoE session | PPPoE-AC | `show pppoe session` | Session appears in locally terminated PTA state |
| 25 | Confirm Virtual-Access was created | PPPoE-AC | `show interface virtual-access <id>` | Virtual-Access is up/up, cloned from Virtual-Template123, with LCP and IPCP open |
| 26 | Confirm client Dialer has a negotiated IP | PPPoE-Client | `show ip interface brief Dialer123` | Dialer123 is up/up with an IPCP-learned address |
| 27 | Test client-to-server PPP reachability | PPPoE-Client | `ping 123.1.1.1` | Ping succeeds across the PPPoE session |
# PPPoE_Discovery_And_Session_Formation_Skeleton
! PPPoE ACCESS CONCENTRATOR / SERVER
interface Loopback123
 ip address 123.1.1.1 255.255.255.0
!
interface Virtual-Template123
 ip unnumbered Loopback123
 peer default ip address pool tst-pool
 mtu 1492
 ip tcp adjust-mss 1452
!
ip local pool tst-pool 123.1.1.2 123.1.1.3
!
bba-group pppoe tst
 virtual-template 123
!
interface FastEthernet0/0
 pppoe enable group tst
 no shutdown
! PPPoE CLIENT
interface Dialer123
 encapsulation ppp
 ip address negotiated
 mtu 1492
 ip tcp adjust-mss 1452
 dialer pool 100
!
interface FastEthernet0/0
 pppoe-client dial-pool-number 100
 no shutdown
# PPPoE_Discovery_And_Session_Formation_Verification_Commands
| Verification Goal | Device | Command | Good Output |
|---|---|---|---|
| Confirm physical link state | PPPoE-AC / PPPoE-Client | `show ip interface brief` | Ethernet interfaces are up/up |
| Confirm PPPoE server sessions | PPPoE-AC | `show pppoe session` | Session count increments and client MAC appears |
| Confirm Session ID assignment | PPPoE-AC | `show pppoe session` | Output shows a PPPoE SID for each client |
| Confirm Virtual-Access creation | PPPoE-AC | `show interface virtual-access <id>` | Interface is up/up and cloned from Virtual-Template123 |
| Confirm PPP control state | PPPoE-AC | `show interface virtual-access <id>` | `Encapsulation PPP, LCP Open` appears |
| Confirm IPCP opened | PPPoE-AC | `show interface virtual-access <id>` | `Open: IPCP` appears |
| Confirm client Dialer state | PPPoE-Client | `show ip interface brief Dialer123` | Dialer123 is up/up |
| Confirm client negotiated address | PPPoE-Client | `show ip interface brief Dialer123` | Dialer123 has an IPCP-assigned IP address |
| Confirm local pool usage | PPPoE-AC | `show ip local pool` | Client address is allocated from the pool |
| Confirm routed reachability over PPPoE | PPPoE-Client | `ping 123.1.1.1` | Ping succeeds |
| Confirm client-to-server path | PPPoE-Client | `traceroute 123.1.1.1` | First hop is the PPPoE access concentrator |
# PPPoE_Discovery_And_Session_Formation_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Remove PPPoE client binding from Ethernet | PPPoE-Client | `interface FastEthernet0/0` | Client physical interface selected |
| 2 | Disable client PPPoE discovery | PPPoE-Client | `no pppoe-client dial-pool-number 100` | Client stops initiating PPPoE discovery |
| 3 | Remove client Dialer interface | PPPoE-Client | `no interface Dialer123` | Logical PPP interface is removed |
| 4 | Remove PPPoE server from client-facing Ethernet | PPPoE-AC | `interface FastEthernet0/0` | Server physical interface selected |
| 5 | Disable PPPoE server discovery | PPPoE-AC | `no pppoe enable group tst` | Server stops answering PADI frames on the interface |
| 6 | Remove PPPoE bba-group | PPPoE-AC | `no bba-group pppoe tst` | PPPoE server instance is removed |
| 7 | Remove Virtual-Template | PPPoE-AC | `no interface Virtual-Template123` | Template for Virtual-Access sessions is removed |
| 8 | Remove local address pool | PPPoE-AC | `no ip local pool tst-pool 123.1.1.2 123.1.1.3` | Client pool is removed |
| 9 | Remove loopback if only used for this lab | PPPoE-AC | `no interface Loopback123` | PPP endpoint loopback is removed |
| 10 | Confirm sessions are gone | PPPoE-AC | `show pppoe session` | No active PPPoE sessions remain |
# PPPoE_Discovery_And_Session_Formation_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| No PPPoE session appears | Server interface is not listening for PPPoE | PPPoE-AC | `show running-config interface FastEthernet0/0` | Add `pppoe enable group tst` |
| No PPPoE session appears | Client physical interface is not bound to dialer pool | PPPoE-Client | `show running-config interface FastEthernet0/0` | Add `pppoe-client dial-pool-number 100` |
| Dialer stays down/down | Dialer pool mismatch | PPPoE-Client | `show running-config interface Dialer123` | Make `dialer pool 100` match the physical interface dial pool |
| Server receives discovery but no usable session forms | bba-group is not tied to Virtual-Template | PPPoE-AC | `show running-config | section bba-group` | Add `virtual-template 123` under the bba-group |
| Virtual-Access interface is not created | Virtual-Template missing or wrong number | PPPoE-AC | `show running-config interface Virtual-Template123` | Create the correct Virtual-Template and bind it under bba-group |
| Client gets no IP address | Local pool missing or exhausted | PPPoE-AC | `show ip local pool` | Create or expand `ip local pool tst-pool` |
| Client gets no IP address | Virtual-Template missing peer address pool | PPPoE-AC | `show running-config interface Virtual-Template123` | Add `peer default ip address pool tst-pool` |
| Session forms but large TCP traffic fails | MTU or MSS not adjusted | PPPoE-AC / PPPoE-Client | `show running-config interface Virtual-Template123` / `show running-config interface Dialer123` | Use `mtu 1492` and `ip tcp adjust-mss 1452` |
| PPPoE session exists but ping fails | Dialer has no negotiated address | PPPoE-Client | `show ip interface brief Dialer123` | Fix IPCP address assignment from the server pool |
| PPPoE session exists but PPP not open | PPP negotiation failed after discovery | PPPoE-AC | `show interface virtual-access <id>` | Check for `LCP Open` and `Open: IPCP`; fix PPP settings |
| Multiple clients fail after first client | Address pool too small | PPPoE-AC | `show ip local pool` | Increase the local pool range |
| Authentication lab fails after discovery | PPPoE discovery succeeded but PPP authentication failed | PPPoE-AC / PPPoE-Client | `debug ppp authentication` | Fix PAP or CHAP settings under the PPP interfaces |
# Index
PPPoE_Discovery_And_Session_Formation.md
# PPPoE_Discovery_And_Session_Formation
# PPPoE_Discovery_And_Session_Formation_Related_Labs
# PPPoE_Discovery_And_Session_Formation_Mental_Model
# PPPoE_Discovery_And_Session_Formation_Configuration_Checklist
# PPPoE_Discovery_And_Session_Formation_Skeleton
# PPPoE_Discovery_And_Session_Formation_Verification_Commands
# PPPoE_Discovery_And_Session_Formation_Rollback
# PPPoE_Discovery_And_Session_Formation_Failure_Checks
# Index

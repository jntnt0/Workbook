PPPoE_Server_Access_Concentrator_BBA_Group_And_Virtual_Template.md
# PPPoE_Server_Access_Concentrator_BBA_Group_And_Virtual_Template
# PPPoE_Server_Access_Concentrator_BBA_Group_And_Virtual_Template_Related_Labs
- pppoe-server-final
- pppoe-server-two-clients-final
- pppoe-server-client-authentication-final
# PPPoE_Server_Access_Concentrator_BBA_Group_And_Virtual_Template_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Access concentrator | The PPPoE server-side router that terminates client PPPoE sessions |
| BBA group | The IOS PPPoE server instance that listens for PPPoE clients and points them to a Virtual-Template |
| Physical Ethernet attachment | PPPoE server functionality does not run globally by itself. The bba-group must be enabled on the client-facing Ethernet interface or subinterface |
| Virtual-Template | The reusable server-side interface template copied into dynamic Virtual-Access interfaces |
| Virtual-Access | A per-client dynamic PPP interface created when a PPPoE client session is accepted |
| IP unnumbered | Lets each Virtual-Access interface borrow the loopback address instead of creating overlapping interface IPs |
| Peer default IP address pool | The server-side PPP IPCP mechanism used to assign client IP addresses without DHCP |
| Local pool | The address range consumed by PPPoE clients as they connect |
| MTU 1492 | PPPoE plus PPP overhead consumes 8 bytes, so the virtual PPP path should not use the full 1500-byte Ethernet MTU |
| TCP MSS 1452 | MSS clamp prevents TCP sessions from sending oversized payloads across the PPPoE path |
| Server inheritance model | Configure common client behavior on the Virtual-Template. Each connected client inherits it through its Virtual-Access interface |
# PPPoE_Server_Access_Concentrator_BBA_Group_And_Virtual_Template_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the server-side client-facing interface exists | PPPoE-AC | `show ip interface brief` | The Ethernet interface facing PPPoE clients is present |
| 2 | Enter global configuration mode | PPPoE-AC | `configure terminal` | Device enters configuration mode |
| 3 | Create the loopback used as the stable server-side PPP endpoint | PPPoE-AC | `interface Loopback123` | Loopback123 is created |
| 4 | Assign the server-side PPP endpoint address | PPPoE-AC | `ip address 123.1.1.1 255.255.255.0` | Loopback123 owns 123.1.1.1/24 |
| 5 | Return to global configuration mode | PPPoE-AC | `exit` | Device returns to global configuration mode |
| 6 | Create the PPPoE server Virtual-Template | PPPoE-AC | `interface Virtual-Template123` | Virtual-Template123 is created |
| 7 | Borrow the loopback IP address for cloned client sessions | PPPoE-AC | `ip unnumbered Loopback123` | Virtual-Access interfaces inherit 123.1.1.1 as the server endpoint |
| 8 | Point PPP clients to the local address pool | PPPoE-AC | `peer default ip address pool tst-pool` | IPCP will assign client addresses from `tst-pool` |
| 9 | Set PPPoE-safe MTU on the template | PPPoE-AC | `mtu 1492` | Cloned Virtual-Access interfaces use MTU 1492 |
| 10 | Clamp TCP MSS for client sessions | PPPoE-AC | `ip tcp adjust-mss 1452` | TCP SYN and SYN-ACK packets are adjusted to MSS 1452 |
| 11 | Return to global configuration mode | PPPoE-AC | `exit` | Device returns to global configuration mode |
| 12 | Create the local pool for PPPoE client IPCP assignment | PPPoE-AC | `ip local pool tst-pool 123.1.1.2 123.1.1.3` | Pool exists with two usable client addresses |
| 13 | Create the PPPoE server bba-group | PPPoE-AC | `bba-group pppoe tst` | PPPoE server instance `tst` is created |
| 14 | Bind the bba-group to the Virtual-Template | PPPoE-AC | `virtual-template 123` | New PPPoE sessions clone Virtual-Access interfaces from Virtual-Template123 |
| 15 | Exit bba-group configuration mode | PPPoE-AC | `exit` | Device returns to global configuration mode |
| 16 | Enter the client-facing Ethernet interface | PPPoE-AC | `interface FastEthernet0/0` | Server is positioned on the PPPoE access segment |
| 17 | Set the lab MAC address if required by the topology | PPPoE-AC | `mac-address 0000.1111.1111` | Server Ethernet interface uses the expected lab MAC address |
| 18 | Enable the PPPoE server instance on the Ethernet interface | PPPoE-AC | `pppoe enable group tst` | Interface listens for PPPoE discovery frames and uses bba-group `tst` |
| 19 | Bring up the server access interface | PPPoE-AC | `no shutdown` | Ethernet interface is administratively enabled |
| 20 | Exit interface configuration mode | PPPoE-AC | `end` | Device returns to privileged EXEC mode |
| 21 | Save the server configuration | PPPoE-AC | `write memory` | PPPoE server configuration is saved |
| 22 | Confirm the bba-group is present | PPPoE-AC | `show running-config | section bba-group` | Output shows `bba-group pppoe tst` with `virtual-template 123` |
| 23 | Confirm the Virtual-Template inheritance source | PPPoE-AC | `show running-config interface Virtual-Template123` | Template shows `ip unnumbered`, peer pool, MTU, and MSS settings |
| 24 | Confirm the server interface is listening for PPPoE clients | PPPoE-AC | `show running-config interface FastEthernet0/0` | Interface shows `pppoe enable group tst` |
| 25 | Confirm PPPoE sessions after client connection | PPPoE-AC | `show pppoe session` | Client sessions appear in locally terminated PTA state |
| 26 | Confirm dynamic Virtual-Access creation after client connection | PPPoE-AC | `show interface virtual-access <id>` | Virtual-Access is up/up and cloned from Virtual-Template123 |
| 27 | Confirm client address pool usage | PPPoE-AC | `show ip local pool` | Client addresses show as in use from `tst-pool` |
# PPPoE_Server_Access_Concentrator_BBA_Group_And_Virtual_Template_Skeleton
! PPPoE ACCESS CONCENTRATOR / SERVER
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
 exit
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
# PPPoE_Server_Access_Concentrator_BBA_Group_And_Virtual_Template_Verification_Commands
| Verification Goal | Device | Command | Good Output |
|---|---|---|---|
| Confirm server physical interface state | PPPoE-AC | `show ip interface brief` | Client-facing Ethernet interface is up/up |
| Confirm loopback endpoint | PPPoE-AC | `show ip interface brief Loopback123` | Loopback123 shows 123.1.1.1 and up/up |
| Confirm Virtual-Template configuration | PPPoE-AC | `show running-config interface Virtual-Template123` | Output shows `ip unnumbered Loopback123`, `peer default ip address pool tst-pool`, `mtu 1492`, and `ip tcp adjust-mss 1452` |
| Confirm local address pool | PPPoE-AC | `show running-config | include ip local pool` | Output shows `ip local pool tst-pool 123.1.1.2 123.1.1.3` |
| Confirm bba-group binding | PPPoE-AC | `show running-config | section bba-group` | Output shows `bba-group pppoe tst` and `virtual-template 123` |
| Confirm Ethernet interface attachment | PPPoE-AC | `show running-config interface FastEthernet0/0` | Output shows `pppoe enable group tst` |
| Confirm active PPPoE sessions | PPPoE-AC | `show pppoe session` | Client sessions appear in `LOCALLY_TERMINATED (PTA)` state |
| Confirm Session ID assignment | PPPoE-AC | `show pppoe session` | Output shows a unique PPPoE SID per client |
| Confirm Virtual-Access clone | PPPoE-AC | `show interface virtual-access <id>` | Output shows `PPPoE vaccess, cloned from Virtual-Template123` |
| Confirm borrowed server address | PPPoE-AC | `show interface virtual-access <id>` | Output shows the interface is unnumbered and uses Loopback123 address |
| Confirm PPP state | PPPoE-AC | `show interface virtual-access <id>` | Output shows `Encapsulation PPP, LCP Open` |
| Confirm IPCP state | PPPoE-AC | `show interface virtual-access <id>` | Output shows `Open: IPCP` |
| Confirm pool consumption | PPPoE-AC | `show ip local pool` | Pool shows client addresses in use |
| Confirm server reachability from client | PPPoE-Client | `ping 123.1.1.1` | Ping succeeds to the access concentrator loopback endpoint |
# PPPoE_Server_Access_Concentrator_BBA_Group_And_Virtual_Template_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter global configuration mode | PPPoE-AC | `configure terminal` | Device enters configuration mode |
| 2 | Enter the PPPoE client-facing interface | PPPoE-AC | `interface FastEthernet0/0` | Server access interface is selected |
| 3 | Disable PPPoE server on the access interface | PPPoE-AC | `no pppoe enable group tst` | Interface stops listening for PPPoE client discovery |
| 4 | Remove the lab MAC address if it was only used for this lab | PPPoE-AC | `no mac-address 0000.1111.1111` | Interface returns to default hardware MAC behavior |
| 5 | Exit interface configuration mode | PPPoE-AC | `exit` | Device returns to global configuration mode |
| 6 | Remove the PPPoE bba-group | PPPoE-AC | `no bba-group pppoe tst` | PPPoE server instance is removed |
| 7 | Remove the Virtual-Template | PPPoE-AC | `no interface Virtual-Template123` | Template used for Virtual-Access clones is removed |
| 8 | Remove the local client address pool | PPPoE-AC | `no ip local pool tst-pool 123.1.1.2 123.1.1.3` | PPPoE client address pool is removed |
| 9 | Remove the loopback if it was created only for PPPoE | PPPoE-AC | `no interface Loopback123` | Server PPP endpoint loopback is removed |
| 10 | Return to privileged EXEC mode | PPPoE-AC | `end` | Device exits configuration mode |
| 11 | Confirm PPPoE sessions are gone | PPPoE-AC | `show pppoe session` | No active PPPoE sessions remain |
| 12 | Confirm bba-group removal | PPPoE-AC | `show running-config | section bba-group` | No `bba-group pppoe tst` remains |
| 13 | Save rollback state | PPPoE-AC | `write memory` | Rollback configuration is saved |
# PPPoE_Server_Access_Concentrator_BBA_Group_And_Virtual_Template_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Client sends PPPoE discovery but no session forms | PPPoE server not enabled on the access interface | PPPoE-AC | `show running-config interface FastEthernet0/0` | Add `pppoe enable group tst` |
| Client discovery reaches server but no Virtual-Access appears | bba-group does not reference the Virtual-Template | PPPoE-AC | `show running-config | section bba-group` | Add `virtual-template 123` under `bba-group pppoe tst` |
| PPPoE server configuration exists but does not listen on the expected segment | bba-group was not attached to the correct Ethernet interface or subinterface | PPPoE-AC | `show running-config | include pppoe enable group` | Attach `pppoe enable group tst` to the correct client-facing interface |
| Virtual-Access comes up with wrong server endpoint | Virtual-Template is not using the intended loopback | PPPoE-AC | `show running-config interface Virtual-Template123` | Add or correct `ip unnumbered Loopback123` |
| Client session forms but client receives no IP address | Missing peer pool reference under Virtual-Template | PPPoE-AC | `show running-config interface Virtual-Template123` | Add `peer default ip address pool tst-pool` |
| First client works but second client fails | Local address pool is too small or exhausted | PPPoE-AC | `show ip local pool` | Expand the pool range |
| Client address assignment fails | Local pool name mismatch | PPPoE-AC | `show running-config interface Virtual-Template123` and `show running-config | include ip local pool` | Make the pool name under `peer default ip address pool` match the `ip local pool` name |
| Session forms but large TCP traffic breaks | MTU or MSS missing from Virtual-Template | PPPoE-AC | `show running-config interface Virtual-Template123` | Add `mtu 1492` and `ip tcp adjust-mss 1452` |
| Virtual-Access does not show PPP open | PPP negotiation failed after PPPoE session establishment | PPPoE-AC | `show interface virtual-access <id>` | Check for `LCP Open` and `Open: IPCP`; fix PPP template settings |
| Authentication lab fails after server-side discovery succeeds | Authentication settings belong on the PPP interface template, not the Ethernet interface | PPPoE-AC | `show running-config interface Virtual-Template123` | Add PAP or CHAP authentication under Virtual-Template in the authentication note |
| Server appears configured but clients never connect | Physical Ethernet interface is shutdown or down/down | PPPoE-AC | `show ip interface brief` | Apply `no shutdown` and fix physical or switching path |
| Lab verification differs from expected MAC output | Lab MAC address was not set | PPPoE-AC | `show interface FastEthernet0/0` | Add `mac-address 0000.1111.1111` if the lab expects fixed MACs |
# Index
PPPoE_Server_Access_Concentrator_BBA_Group_And_Virtual_Template.md
# PPPoE_Server_Access_Concentrator_BBA_Group_And_Virtual_Template
# PPPoE_Server_Access_Concentrator_BBA_Group_And_Virtual_Template_Related_Labs
# PPPoE_Server_Access_Concentrator_BBA_Group_And_Virtual_Template_Mental_Model
# PPPoE_Server_Access_Concentrator_BBA_Group_And_Virtual_Template_Configuration_Checklist
# PPPoE_Server_Access_Concentrator_BBA_Group_And_Virtual_Template_Skeleton
# PPPoE_Server_Access_Concentrator_BBA_Group_And_Virtual_Template_Verification_Commands
# PPPoE_Server_Access_Concentrator_BBA_Group_And_Virtual_Template_Rollback
# PPPoE_Server_Access_Concentrator_BBA_Group_And_Virtual_Template_Failure_Checks
# Index

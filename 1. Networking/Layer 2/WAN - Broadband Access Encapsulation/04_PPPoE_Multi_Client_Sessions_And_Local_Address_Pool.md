PPPoE_Multi_Client_Sessions_And_Local_Address_Pool.md
# PPPoE_Multi_Client_Sessions_And_Local_Address_Pool
# PPPoE_Multi_Client_Sessions_And_Local_Address_Pool_Related_Labs
- pppoe-server-two-clients-final
# PPPoE_Multi_Client_Sessions_And_Local_Address_Pool_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Multi-client PPPoE | One access concentrator can terminate more than one PPPoE client on the same Ethernet segment |
| One bba-group | The server does not need a separate bba-group per client when all clients share the same service behavior |
| One Virtual-Template | The template is reused for every client session |
| Multiple Virtual-Access interfaces | Each client gets its own dynamic Virtual-Access interface cloned from the same Virtual-Template |
| One Session ID per client | Each PPPoE client session receives its own Session ID from the access concentrator |
| Local address pool | The access concentrator assigns client IP addresses from the configured local pool using PPP IPCP |
| Pool sizing | The pool must contain at least one usable address per simultaneous client |
| No DHCP requirement | In this model, client addressing comes from `ip local pool`, not DHCP |
| Unique client MAC addresses | Each client must appear as a unique PPPoE endpoint on the shared Ethernet segment |
| Same client Dialer model | R2 and R3 can use the same local Dialer number and dialer pool number because those numbers are local to each router |
| Server-side visibility | The access concentrator should show multiple sessions, multiple remote MACs, multiple Session IDs, and multiple Virtual-Access interfaces |
| Client-side visibility | Each client Dialer should be up/up with a unique IPCP-learned address |
# PPPoE_Multi_Client_Sessions_And_Local_Address_Pool_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the access concentrator interface toward clients exists | R1 / PPPoE-AC | `show ip interface brief` | FastEthernet0/0 is present |
| 2 | Confirm both client interfaces exist | R2 / R3 | `show ip interface brief` | FastEthernet0/0 is present on each client |
| 3 | Enter global configuration mode on the access concentrator | R1 / PPPoE-AC | `configure terminal` | Device enters configuration mode |
| 4 | Create the server loopback endpoint | R1 / PPPoE-AC | `interface Loopback123` | Loopback123 is created |
| 5 | Assign the shared server-side PPP endpoint address | R1 / PPPoE-AC | `ip address 123.1.1.1 255.255.255.0` | Loopback123 owns 123.1.1.1/24 |
| 6 | Create the Virtual-Template used by all clients | R1 / PPPoE-AC | `interface Virtual-Template123` | Virtual-Template123 is created |
| 7 | Make all client sessions borrow the loopback address | R1 / PPPoE-AC | `ip unnumbered Loopback123` | All cloned Virtual-Access interfaces use 123.1.1.1 as the server endpoint |
| 8 | Point PPP IPCP client assignment to the local pool | R1 / PPPoE-AC | `peer default ip address pool tst-pool` | PPP clients receive addresses from `tst-pool` |
| 9 | Set PPPoE-safe MTU on the shared template | R1 / PPPoE-AC | `mtu 1492` | All client Virtual-Access sessions inherit MTU 1492 |
| 10 | Clamp TCP MSS for all PPPoE clients | R1 / PPPoE-AC | `ip tcp adjust-mss 1452` | All client sessions inherit TCP MSS 1452 |
| 11 | Create a local pool large enough for two clients | R1 / PPPoE-AC | `ip local pool tst-pool 123.1.1.2 123.1.1.3` | Pool has one address for R2 and one address for R3 |
| 12 | Create the PPPoE server instance | R1 / PPPoE-AC | `bba-group pppoe tst` | PPPoE bba-group `tst` is created |
| 13 | Bind the PPPoE server instance to the shared Virtual-Template | R1 / PPPoE-AC | `virtual-template 123` | Every accepted client clones a Virtual-Access interface from Virtual-Template123 |
| 14 | Enter the Ethernet interface facing both clients | R1 / PPPoE-AC | `interface FastEthernet0/0` | Server access interface is selected |
| 15 | Set the server lab MAC address if required | R1 / PPPoE-AC | `mac-address 0000.1111.1111` | Server uses the expected lab MAC |
| 16 | Enable PPPoE server service on the client-facing Ethernet segment | R1 / PPPoE-AC | `pppoe enable group tst` | R1 listens for PPPoE discovery frames from multiple clients |
| 17 | Bring up the server access interface | R1 / PPPoE-AC | `no shutdown` | FastEthernet0/0 is enabled |
| 18 | Create R2 client Dialer interface | R2 / PPPoE-Client-1 | `interface Dialer123` | Dialer123 is created on R2 |
| 19 | Set R2 Dialer encapsulation to PPP | R2 / PPPoE-Client-1 | `encapsulation ppp` | R2 Dialer uses PPP |
| 20 | Configure R2 to receive an IPCP address | R2 / PPPoE-Client-1 | `ip address negotiated` | R2 expects an address from R1 |
| 21 | Set R2 PPPoE-safe MTU | R2 / PPPoE-Client-1 | `mtu 1492` | R2 Dialer uses MTU 1492 |
| 22 | Clamp R2 TCP MSS | R2 / PPPoE-Client-1 | `ip tcp adjust-mss 1452` | R2 Dialer adjusts TCP MSS to 1452 |
| 23 | Bind R2 Dialer to dialer pool 100 | R2 / PPPoE-Client-1 | `dialer pool 100` | R2 Dialer can use PPPoE client instances from pool 100 |
| 24 | Enter R2 physical PPPoE interface | R2 / PPPoE-Client-1 | `interface FastEthernet0/0` | R2 access interface is selected |
| 25 | Set R2 lab MAC address | R2 / PPPoE-Client-1 | `mac-address 0000.2222.2222` | R2 presents a unique PPPoE client MAC |
| 26 | Bind R2 physical interface to the Dialer pool | R2 / PPPoE-Client-1 | `pppoe-client dial-pool-number 100` | R2 starts PPPoE discovery for Dialer123 |
| 27 | Bring up R2 physical interface | R2 / PPPoE-Client-1 | `no shutdown` | R2 can form the first PPPoE session |
| 28 | Create R3 client Dialer interface | R3 / PPPoE-Client-2 | `interface Dialer123` | Dialer123 is created on R3 |
| 29 | Set R3 Dialer encapsulation to PPP | R3 / PPPoE-Client-2 | `encapsulation ppp` | R3 Dialer uses PPP |
| 30 | Configure R3 to receive an IPCP address | R3 / PPPoE-Client-2 | `ip address negotiated` | R3 expects an address from R1 |
| 31 | Set R3 PPPoE-safe MTU | R3 / PPPoE-Client-2 | `mtu 1492` | R3 Dialer uses MTU 1492 |
| 32 | Clamp R3 TCP MSS | R3 / PPPoE-Client-2 | `ip tcp adjust-mss 1452` | R3 Dialer adjusts TCP MSS to 1452 |
| 33 | Bind R3 Dialer to dialer pool 100 | R3 / PPPoE-Client-2 | `dialer pool 100` | R3 Dialer can use PPPoE client instances from pool 100 |
| 34 | Enter R3 physical PPPoE interface | R3 / PPPoE-Client-2 | `interface FastEthernet0/0` | R3 access interface is selected |
| 35 | Set R3 lab MAC address | R3 / PPPoE-Client-2 | `mac-address 0000.3333.3333` | R3 presents a unique PPPoE client MAC |
| 36 | Bind R3 physical interface to the Dialer pool | R3 / PPPoE-Client-2 | `pppoe-client dial-pool-number 100` | R3 starts PPPoE discovery for Dialer123 |
| 37 | Bring up R3 physical interface | R3 / PPPoE-Client-2 | `no shutdown` | R3 can form the second PPPoE session |
| 38 | Confirm two PPPoE sessions exist | R1 / PPPoE-AC | `show pppoe session` | Output shows 2 sessions in locally terminated PTA state |
| 39 | Confirm two unique client MAC addresses | R1 / PPPoE-AC | `show pppoe session` | Output shows 0000.2222.2222 and 0000.3333.3333 |
| 40 | Confirm two unique Session IDs | R1 / PPPoE-AC | `show pppoe session` | Output shows separate PPPoE SIDs for R2 and R3 |
| 41 | Confirm two Virtual-Access interfaces | R1 / PPPoE-AC | `show pppoe session` | Output shows separate VA entries such as Vi2.1 and Vi2.2 |
| 42 | Confirm R2 received its negotiated address | R2 / PPPoE-Client-1 | `show ip interface brief Dialer123` | Dialer123 is up/up with 123.1.1.2 |
| 43 | Confirm R3 received its negotiated address | R3 / PPPoE-Client-2 | `show ip interface brief Dialer123` | Dialer123 is up/up with 123.1.1.3 |
| 44 | Confirm R2 can reach the access concentrator PPP endpoint | R2 / PPPoE-Client-1 | `ping 123.1.1.1` | Ping succeeds |
| 45 | Confirm R3 can reach the access concentrator PPP endpoint | R3 / PPPoE-Client-2 | `ping 123.1.1.1` | Ping succeeds |
| 46 | Save the working configuration | R1 / R2 / R3 | `write memory` | PPPoE multi-client configuration is saved |
# PPPoE_Multi_Client_Sessions_And_Local_Address_Pool_Skeleton
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
! R2 PPPoE CLIENT 1
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
! R3 PPPoE CLIENT 2
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
# PPPoE_Multi_Client_Sessions_And_Local_Address_Pool_Verification_Commands
| Verification Goal | Device | Command | Good Output |
|---|---|---|---|
| Confirm server interface state | R1 / PPPoE-AC | `show ip interface brief FastEthernet0/0` | FastEthernet0/0 is up/up |
| Confirm R2 client interface state | R2 / PPPoE-Client-1 | `show ip interface brief FastEthernet0/0` | FastEthernet0/0 is up/up |
| Confirm R3 client interface state | R3 / PPPoE-Client-2 | `show ip interface brief FastEthernet0/0` | FastEthernet0/0 is up/up |
| Confirm server Virtual-Template | R1 / PPPoE-AC | `show running-config interface Virtual-Template123` | Template shows `ip unnumbered Loopback123`, `peer default ip address pool tst-pool`, `mtu 1492`, and `ip tcp adjust-mss 1452` |
| Confirm local pool range | R1 / PPPoE-AC | `show running-config | include ip local pool` | Output shows `ip local pool tst-pool 123.1.1.2 123.1.1.3` |
| Confirm bba-group binding | R1 / PPPoE-AC | `show running-config | section bba-group` | Output shows `bba-group pppoe tst` and `virtual-template 123` |
| Confirm PPPoE service on access interface | R1 / PPPoE-AC | `show running-config interface FastEthernet0/0` | Output shows `pppoe enable group tst` |
| Confirm two server-side sessions | R1 / PPPoE-AC | `show pppoe session` | Output shows `2 sessions in LOCALLY_TERMINATED (PTA) State` |
| Confirm R2 session identity | R1 / PPPoE-AC | `show pppoe session` | Output shows remote MAC 0000.2222.2222 |
| Confirm R3 session identity | R1 / PPPoE-AC | `show pppoe session` | Output shows remote MAC 0000.3333.3333 |
| Confirm unique Session IDs | R1 / PPPoE-AC | `show pppoe session` | R2 and R3 show different PPPoE SIDs |
| Confirm unique Virtual-Access interfaces | R1 / PPPoE-AC | `show pppoe session` | R2 and R3 map to separate VA interfaces |
| Confirm first Virtual-Access state | R1 / PPPoE-AC | `show interface virtual-access <r2-va-id>` | Interface is up/up, PPP LCP is open, and IPCP is open |
| Confirm second Virtual-Access state | R1 / PPPoE-AC | `show interface virtual-access <r3-va-id>` | Interface is up/up, PPP LCP is open, and IPCP is open |
| Confirm R2 negotiated address | R2 / PPPoE-Client-1 | `show ip interface brief Dialer123` | Dialer123 is up/up with 123.1.1.2 |
| Confirm R3 negotiated address | R3 / PPPoE-Client-2 | `show ip interface brief Dialer123` | Dialer123 is up/up with 123.1.1.3 |
| Confirm R2 reachability to server endpoint | R2 / PPPoE-Client-1 | `ping 123.1.1.1` | Ping succeeds |
| Confirm R3 reachability to server endpoint | R3 / PPPoE-Client-2 | `ping 123.1.1.1` | Ping succeeds |
| Confirm local pool allocation state | R1 / PPPoE-AC | `show ip local pool` | Pool shows client addresses allocated or in use |
# PPPoE_Multi_Client_Sessions_And_Local_Address_Pool_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter R3 global configuration mode | R3 / PPPoE-Client-2 | `configure terminal` | R3 enters configuration mode |
| 2 | Enter R3 physical PPPoE interface | R3 / PPPoE-Client-2 | `interface FastEthernet0/0` | R3 physical access interface is selected |
| 3 | Remove R3 PPPoE client binding | R3 / PPPoE-Client-2 | `no pppoe-client dial-pool-number 100` | R3 stops initiating PPPoE discovery |
| 4 | Remove R3 lab MAC address if it was only used for the lab | R3 / PPPoE-Client-2 | `no mac-address 0000.3333.3333` | R3 returns to the default interface MAC |
| 5 | Remove R3 Dialer interface | R3 / PPPoE-Client-2 | `no interface Dialer123` | R3 logical PPP client interface is removed |
| 6 | Confirm only one PPPoE client remains | R1 / PPPoE-AC | `show pppoe session` | Only the R2 session remains |
| 7 | Enter R1 global configuration mode if reducing pool to one client | R1 / PPPoE-AC | `configure terminal` | R1 enters configuration mode |
| 8 | Remove the two-client local pool | R1 / PPPoE-AC | `no ip local pool tst-pool 123.1.1.2 123.1.1.3` | Original two-client local pool is removed |
| 9 | Recreate a one-client local pool if only R2 should remain | R1 / PPPoE-AC | `ip local pool tst-pool 123.1.1.2 123.1.1.2` | Pool now supports only one PPPoE client |
| 10 | Save rollback state | R1 / R3 | `write memory` | Rollback configuration is saved |
# PPPoE_Multi_Client_Sessions_And_Local_Address_Pool_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| First client works but second client fails | Local address pool only has one usable address | R1 / PPPoE-AC | `show running-config | include ip local pool` | Expand the pool to include one address per simultaneous client |
| First client works but second client has no IP address | Pool is exhausted | R1 / PPPoE-AC | `show ip local pool` | Add more addresses to `tst-pool` |
| R2 and R3 do not both appear in PPPoE sessions | One client is missing physical PPPoE binding | R2 / R3 | `show running-config interface FastEthernet0/0` | Add `pppoe-client dial-pool-number 100` |
| Client Dialer stays down/down | Dialer pool mismatch | R2 / R3 | `show running-config interface Dialer123` and `show running-config interface FastEthernet0/0` | Match `dialer pool 100` to `pppoe-client dial-pool-number 100` |
| Both clients show same or unexpected identity | Duplicate lab MAC address | R2 / R3 | `show interface FastEthernet0/0` | Use unique MAC addresses for each client |
| Server shows only one client MAC | Second client interface is shutdown or disconnected | R3 / PPPoE-Client-2 | `show ip interface brief FastEthernet0/0` | Apply `no shutdown` and fix the L2 path |
| Server has no sessions | PPPoE bba-group not enabled on client-facing interface | R1 / PPPoE-AC | `show running-config interface FastEthernet0/0` | Add `pppoe enable group tst` |
| Sessions form but no Virtual-Access interfaces are usable | bba-group is not bound to the Virtual-Template | R1 / PPPoE-AC | `show running-config | section bba-group` | Add `virtual-template 123` under `bba-group pppoe tst` |
| Client receives no address even though session forms | Virtual-Template does not reference the local pool | R1 / PPPoE-AC | `show running-config interface Virtual-Template123` | Add `peer default ip address pool tst-pool` |
| R2 gets address but R3 does not | Pool range does not include enough addresses | R1 / PPPoE-AC | `show running-config | include ip local pool` | Use `ip local pool tst-pool 123.1.1.2 123.1.1.3` for two clients |
| Ping to 123.1.1.1 fails from one client | That client Dialer did not receive an IPCP address | R2 / R3 | `show ip interface brief Dialer123` | Fix client Dialer and server local pool configuration |
| Large TCP traffic fails while ping works | MTU or MSS missing from template or Dialer | R1 / R2 / R3 | `show running-config interface Virtual-Template123` and `show running-config interface Dialer123` | Use `mtu 1492` and `ip tcp adjust-mss 1452` |
| Server shows two sessions but one is unstable | PPP LCP or IPCP not open on one Virtual-Access interface | R1 / PPPoE-AC | `show interface virtual-access <id>` | Confirm `Encapsulation PPP, LCP Open` and `Open: IPCP` |
| Authentication lab fails after multi-client discovery works | Authentication is a PPP phase issue, not a PPPoE discovery issue | R1 / R2 / R3 | `debug ppp authentication` | Fix PAP or CHAP settings in the authentication note |
# Index
PPPoE_Multi_Client_Sessions_And_Local_Address_Pool.md
# PPPoE_Multi_Client_Sessions_And_Local_Address_Pool
# PPPoE_Multi_Client_Sessions_And_Local_Address_Pool_Related_Labs
# PPPoE_Multi_Client_Sessions_And_Local_Address_Pool_Mental_Model
# PPPoE_Multi_Client_Sessions_And_Local_Address_Pool_Configuration_Checklist
# PPPoE_Multi_Client_Sessions_And_Local_Address_Pool_Skeleton
# PPPoE_Multi_Client_Sessions_And_Local_Address_Pool_Verification_Commands
# PPPoE_Multi_Client_Sessions_And_Local_Address_Pool_Rollback
# PPPoE_Multi_Client_Sessions_And_Local_Address_Pool_Failure_Checks
# Index

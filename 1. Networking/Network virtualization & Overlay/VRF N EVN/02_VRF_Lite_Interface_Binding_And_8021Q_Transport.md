
VRF_Lite_Interface_Binding_And_8021Q_Transport.md

VRF_Lite_Interface_Binding_And_8021Q_Transport

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Cisco Press / ENARSI source mapping | Confirms Cisco Press ENARSI material as the primary local source for VRF-Lite configuration |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 18, `Implementing and Verifying VRF-Lite` | Defines VRF-Lite as separate virtual routers with isolated interfaces, routing tables, and forwarding tables |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | `Creating and Verifying VRF Instances` | Supports `vrf definition`, `address-family ipv4 unicast`, `show vrf`, and `show ip vrf` |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Example 18-3, `Assigning Interfaces to the VRF Instances` | Provides `vrf forwarding <VRF>` and `ip vrf forwarding <VRF>` interface binding syntax |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Example 18-5, `Creating Subinterfaces on R1 and Assigning Them to the Correct VRF Instances` | Supports using subinterfaces when one physical link must carry multiple VRFs |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Example 18-6, `Configuring R1’s Interfaces and Subinterfaces with IPv4 and IPv6 Addresses` | Provides `encapsulation dot1Q <vlan-id>` and IP addressing sequence for VRF subinterfaces |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Example 18-7, `Verifying Interface IPv4 and IPv6 Address, VRF, and Protocol Configurations` | Provides `show vrf ipv4 unicast interfaces` and `show ip vrf interfaces` verification commands |
# VRF_Lite_Interface_Binding_And_8021Q_Transport_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Interface binding | A VRF does not receive traffic until an interface or subinterface is assigned to it |
| One interface, one VRF | A routed interface can belong to only one VRF at a time |
| Subinterface transport | If one physical link must carry multiple VRFs, create one 802.1Q subinterface per VRF |
| VLAN tag | The dot1Q VLAN tag separates VRF traffic across the shared physical link |
| Matching VLANs | Both ends of the VRF trunk must use matching VLAN tags for the same VRF |
| VRF before IP | Apply `vrf forwarding <VRF>` before assigning the IP address because VRF binding removes existing IP addresses |
| Global table separation | Once an interface is in a VRF, its connected route appears in the VRF table, not the global table |
| Case sensitivity | VRF names are case-sensitive; `GREEN` and `Green` are different VRFs |
| Failure boundary | Most failures are wrong VRF name, missing dot1Q tag, mismatched VLAN tag, or IP address configured before VRF binding |
# VRF_Lite_Interface_Binding_And_8021Q_Transport_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the physical link plan before binding interfaces | All routers | `show ip interface brief` | Interfaces that will become VRF access links or VRF trunks are identified |
| 2 | Confirm existing IP addresses before applying VRF binding | All routers | `show running-config interface <interface>` | You know which IP addresses will be removed when `vrf forwarding` is applied |
| 3 | Verify the required VRFs already exist | Router | `show vrf` | Required VRFs appear with `ipv4` listed under Protocols |
| 4 | Create the RED VRF if missing | Router | `vrf definition RED` then `address-family ipv4 unicast` | RED VRF exists and supports IPv4 unicast |
| 5 | Create the GREEN VRF if missing | Router | `vrf definition GREEN` then `address-family ipv4 unicast` | GREEN VRF exists and supports IPv4 unicast |
| 6 | Create the BLUE VRF if missing | Router | `vrf definition BLUE` then `address-family ipv4 unicast` | BLUE VRF exists and supports IPv4 unicast |
| 7 | Bind a dedicated routed interface to one VRF | Router | `interface GigabitEthernet0/0` then `vrf forwarding RED` | Interface is moved into the RED VRF |
| 8 | Assign the IP address after VRF binding | Router | `ip address 10.0.1.1 255.255.255.0` | IP address is installed under the RED VRF |
| 9 | Enable the dedicated VRF interface | Router | `no shutdown` | Interface can transition up/up |
| 10 | Repeat dedicated interface binding for GREEN | Router | `interface GigabitEthernet1/0` then `vrf forwarding GREEN` then `ip address 172.16.1.1 255.255.255.0` | GREEN access interface is bound and addressed |
| 11 | Repeat dedicated interface binding for BLUE | Router | `interface GigabitEthernet2/0` then `vrf forwarding BLUE` then `ip address 192.168.1.1 255.255.255.0` | BLUE access interface is bound and addressed |
| 12 | Identify the physical link that must carry multiple VRFs | Router | `show cdp neighbors` or `show lldp neighbors` | Shared router-to-router or router-to-switch transport link is confirmed |
| 13 | Leave the parent physical interface unnumbered for subinterface transport | Router | `interface GigabitEthernet3/0` then `no ip address` then `no shutdown` | Parent link is active and ready to carry subinterfaces |
| 14 | Create RED transport subinterface | Router | `interface GigabitEthernet3/0.100` | RED subinterface is created |
| 15 | Bind RED subinterface to the RED VRF | Router | `vrf forwarding RED` | RED subinterface belongs to the RED VRF |
| 16 | Apply RED dot1Q tag | Router | `encapsulation dot1Q 100` | RED traffic uses VLAN 100 across the shared link |
| 17 | Assign RED transport IP address | Router | `ip address 10.0.12.1 255.255.255.0` | RED connected transport route is installed in the RED VRF |
| 18 | Create GREEN transport subinterface | Router | `interface GigabitEthernet3/0.200` | GREEN subinterface is created |
| 19 | Bind GREEN subinterface to the GREEN VRF | Router | `vrf forwarding GREEN` | GREEN subinterface belongs to the GREEN VRF |
| 20 | Apply GREEN dot1Q tag | Router | `encapsulation dot1Q 200` | GREEN traffic uses VLAN 200 across the shared link |
| 21 | Assign GREEN transport IP address | Router | `ip address 172.16.12.1 255.255.255.0` | GREEN connected transport route is installed in the GREEN VRF |
| 22 | Create BLUE transport subinterface | Router | `interface GigabitEthernet3/0.300` | BLUE subinterface is created |
| 23 | Bind BLUE subinterface to the BLUE VRF | Router | `vrf forwarding BLUE` | BLUE subinterface belongs to the BLUE VRF |
| 24 | Apply BLUE dot1Q tag | Router | `encapsulation dot1Q 300` | BLUE traffic uses VLAN 300 across the shared link |
| 25 | Assign BLUE transport IP address | Router | `ip address 192.168.12.1 255.255.255.0` | BLUE connected transport route is installed in the BLUE VRF |
| 26 | Mirror the same VRF-to-VLAN mapping on the far-end router | Neighbor router | `interface GigabitEthernet3/0.100` then `vrf forwarding RED` then `encapsulation dot1Q 100` | Far-end RED subinterface uses the same VLAN tag |
| 27 | Verify interface-to-VRF mapping | Router | `show vrf ipv4 unicast interfaces` | Each physical interface and subinterface appears under the correct VRF |
| 28 | Verify legacy interface-to-VRF view if needed | Router | `show ip vrf interfaces` | Interface, IP address, VRF, and protocol state are visible |
| 29 | Verify parent and subinterface line state | Router | `show ip interface brief` | Parent and subinterfaces are up/up or expected state |
| 30 | Verify connected routes are not in the global table | Router | `show ip route` | VRF-connected transport routes are absent from the global table |
| 31 | Verify RED connected transport route | Router | `show ip route vrf RED connected` | RED subinterface connected network appears only in RED |
| 32 | Verify GREEN connected transport route | Router | `show ip route vrf GREEN connected` | GREEN subinterface connected network appears only in GREEN |
| 33 | Verify BLUE connected transport route | Router | `show ip route vrf BLUE connected` | BLUE subinterface connected network appears only in BLUE |
| 34 | Test RED transport reachability using the RED VRF | Router | `ping vrf RED 10.0.12.2` | RED far-end transport IP replies |
| 35 | Test GREEN transport reachability using the GREEN VRF | Router | `ping vrf GREEN 172.16.12.2` | GREEN far-end transport IP replies |
| 36 | Test BLUE transport reachability using the BLUE VRF | Router | `ping vrf BLUE 192.168.12.2` | BLUE far-end transport IP replies |
| 37 | Save the working interface binding baseline | Router | `write memory` | VRF interface and 802.1Q transport configuration survives reload |
# VRF_Lite_Interface_Binding_And_8021Q_Transport_Skeleton
configure terminal
vrf definition RED
 address-family ipv4 unicast
 exit
exit
vrf definition GREEN
 address-family ipv4 unicast
 exit
exit
vrf definition BLUE
 address-family ipv4 unicast
 exit
exit
interface GigabitEthernet0/0
 description RED_ACCESS_LINK
 vrf forwarding RED
 ip address 10.0.1.1 255.255.255.0
 no shutdown
interface GigabitEthernet1/0
 description GREEN_ACCESS_LINK
 vrf forwarding GREEN
 ip address 172.16.1.1 255.255.255.0
 no shutdown
interface GigabitEthernet2/0
 description BLUE_ACCESS_LINK
 vrf forwarding BLUE
 ip address 192.168.1.1 255.255.255.0
 no shutdown
interface GigabitEthernet3/0
 description VRF_LITE_8021Q_TRANSPORT_TO_PEER
 no ip address
 no shutdown
interface GigabitEthernet3/0.100
 description RED_TRANSPORT_OVER_DOT1Q_100
 vrf forwarding RED
 encapsulation dot1Q 100
 ip address 10.0.12.1 255.255.255.0
 no shutdown
interface GigabitEthernet3/0.200
 description GREEN_TRANSPORT_OVER_DOT1Q_200
 vrf forwarding GREEN
 encapsulation dot1Q 200
 ip address 172.16.12.1 255.255.255.0
 no shutdown
interface GigabitEthernet3/0.300
 description BLUE_TRANSPORT_OVER_DOT1Q_300
 vrf forwarding BLUE
 encapsulation dot1Q 300
 ip address 192.168.12.1 255.255.255.0
 no shutdown
end
write memory
# VRF_Lite_Interface_Binding_And_8021Q_Transport_Verification_Commands
show vrf
show ip vrf
show vrf ipv4 unicast interfaces
show ip vrf interfaces
show ip interface brief
show running-config | section ^vrf definition
show running-config interface GigabitEthernet0/0
show running-config interface GigabitEthernet1/0
show running-config interface GigabitEthernet2/0
show running-config interface GigabitEthernet3/0
show running-config interface GigabitEthernet3/0.100
show running-config interface GigabitEthernet3/0.200
show running-config interface GigabitEthernet3/0.300
show ip route
show ip route vrf RED connected
show ip route vrf GREEN connected
show ip route vrf BLUE connected
ping vrf RED 10.0.12.2
ping vrf GREEN 172.16.12.2
ping vrf BLUE 192.168.12.2
traceroute vrf RED 10.0.12.2
traceroute vrf GREEN 172.16.12.2
traceroute vrf BLUE 192.168.12.2
# VRF_Lite_Interface_Binding_And_8021Q_Transport_Rollback
configure terminal
interface GigabitEthernet0/0
 no ip address
 no vrf forwarding RED
 shutdown
 no description
interface GigabitEthernet1/0
 no ip address
 no vrf forwarding GREEN
 shutdown
 no description
interface GigabitEthernet2/0
 no ip address
 no vrf forwarding BLUE
 shutdown
 no description
interface GigabitEthernet3/0.100
 no ip address
 no vrf forwarding RED
 no encapsulation dot1Q 100
 shutdown
 no description
interface GigabitEthernet3/0.200
 no ip address
 no vrf forwarding GREEN
 no encapsulation dot1Q 200
 shutdown
 no description
interface GigabitEthernet3/0.300
 no ip address
 no vrf forwarding BLUE
 no encapsulation dot1Q 300
 shutdown
 no description
interface GigabitEthernet3/0
 shutdown
 no description
no vrf definition RED
no vrf definition GREEN
no vrf definition BLUE
end
write memory
# VRF_Lite_Interface_Binding_And_8021Q_Transport_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| IP address disappears after applying VRF | IP was configured before `vrf forwarding` | `show running-config interface <interface>` | Reapply the IP address after `vrf forwarding <VRF>` |
| Interface does not appear under the expected VRF | Wrong VRF name or case mismatch | `show vrf ipv4 unicast interfaces` | Correct the VRF name exactly, including case |
| Subinterface cannot forward traffic | Missing dot1Q encapsulation | `show running-config interface <subinterface>` | Add `encapsulation dot1Q <vlan-id>` before relying on the subinterface |
| RED works but GREEN or BLUE fails | VLAN tag mismatch between routers | Compare `show running-config interface <subinterface>` on both ends | Use the same VLAN ID for the same VRF on both ends |
| Parent interface is down | Physical link is shut, disconnected, or wrong adapter selected in CML | `show ip interface brief` | Fix cabling, CML link, or `no shutdown` on parent interface |
| Subinterfaces are down/down | Parent physical interface is down | `show ip interface brief` | Bring up the parent interface first |
| Ping fails from default prompt | Ping is using global table instead of VRF table | `show ip route` and `show ip route vrf <VRF>` | Use `ping vrf <VRF> <destination>` |
| Connected route appears missing | Looking in global table instead of VRF table | `show ip route` then `show ip route vrf <VRF> connected` | Verify inside the correct VRF |
| Far-end does not reply inside same VRF | Far-end subinterface has wrong VRF, VLAN, or IP subnet | `show vrf ipv4 unicast interfaces` on both routers | Match VRF, VLAN tag, and subnet on both ends |
| Traffic crosses between VRFs unexpectedly | Route leaking or shared services configured elsewhere | `show running-config | include ip route vrf|route-target|import|export` | Remove unintended leaking before testing pure interface isolation |
##### Source_Basis
# VRF_Lite_Interface_Binding_And_8021Q_Transport_Mental_Model
# VRF_Lite_Interface_Binding_And_8021Q_Transport_Configuration_Checklist
# VRF_Lite_Interface_Binding_And_8021Q_Transport_Skeleton
# VRF_Lite_Interface_Binding_And_8021Q_Transport_Verification_Commands
# VRF_Lite_Interface_Binding_And_8021Q_Transport_Rollback
# VRF_Lite_Interface_Binding_And_8021Q_Transport_Failure_Checks

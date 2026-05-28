# VRF_Lite_Routing_Table_Isolation_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Global routing table | The default routing table. Interfaces stay here unless explicitly moved into a VRF |
| VRF instance | A separate routing and forwarding table on the same physical router |
| VRF-Lite | VRF segmentation without MPLS |
| Interface binding | Traffic enters a VRF only when the ingress interface is assigned to that VRF |
| Address order | Apply `vrf forwarding` before IP addressing because assigning a VRF removes existing IP addresses |
| Subinterfaces | Used when one physical trunk must carry multiple VRFs |
| Isolation proof | Connected routes for VRF interfaces must appear under `show ip route vrf <VRF>` and not in the global table |
| Case sensitivity | `RED`, `Red`, and `red` are different VRF names |
# VRF_Lite_Routing_Table_Isolation_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Record the intended VRF names, interfaces, VLAN tags, and IP subnets | All routers | `show ip interface brief` | Baseline confirms which interfaces will be moved into each VRF |
| 2 | Confirm no existing production IP address will be lost unexpectedly | All routers | `show running-config interface <interface>` | You know whether `vrf forwarding` will remove an existing address |
| 3 | Enter global configuration mode | Router | `configure terminal` | Device enters configuration mode |
| 4 | Create the first VRF using modern IOS XE syntax | Router | `vrf definition RED` | VRF definition mode opens for `RED` |
| 5 | Enable IPv4 unicast routing context inside the VRF | Router | `address-family ipv4 unicast` | IPv4 unicast address family exists under the VRF |
| 6 | Exit the VRF address family | Router | `exit` | Device returns to VRF definition mode |
| 7 | Repeat VRF creation for each required isolated routing table | Router | `vrf definition GREEN` then `address-family ipv4 unicast` | Each VRF has its own IPv4 routing table |
| 8 | Verify VRFs exist before attaching interfaces | Router | `show vrf` | VRFs appear with `ipv4` listed under protocols |
| 9 | Enter the physical interface that belongs to one VRF only | Router | `interface GigabitEthernet0/0` | Interface configuration mode opens |
| 10 | Attach the interface to the correct VRF before assigning IP | Router | `vrf forwarding RED` | Interface is moved from global table into `RED` |
| 11 | Assign the interface IP address after VRF binding | Router | `ip address 10.0.1.1 255.255.255.0` | IP address belongs to the `RED` VRF, not global |
| 12 | Bring up the interface if administratively down | Router | `no shutdown` | Interface can come up/up if Layer 1 and Layer 2 are valid |
| 13 | Create a subinterface when one physical link carries multiple VRFs | Router | `interface GigabitEthernet3/0.100` | Subinterface configuration mode opens |
| 14 | Attach the subinterface to the correct VRF first | Router | `vrf forwarding RED` | Subinterface belongs to `RED` |
| 15 | Apply matching 802.1Q tag on the subinterface | Router | `encapsulation dot1Q 100` | Subinterface can carry VLAN 100 traffic |
| 16 | Assign the subinterface IP address | Router | `ip address 10.0.12.1 255.255.255.0` | Connected route is installed in `RED` only |
| 17 | Repeat subinterface binding for each VRF carried across the trunk | Router | `interface GigabitEthernet3/0.200` then `vrf forwarding GREEN` then `encapsulation dot1Q 200` | Each VRF has a separate tagged subinterface |
| 18 | Verify interface-to-VRF mapping | Router | `show vrf ipv4 unicast interfaces` | Interfaces show the correct VRF, protocol state, and IP address |
| 19 | Verify the global table does not contain VRF connected routes | Router | `show ip route` | VRF interface connected networks are absent from the global table |
| 20 | Verify the RED VRF routing table | Router | `show ip route vrf RED` | Only RED connected and local routes appear |
| 21 | Verify the GREEN VRF routing table | Router | `show ip route vrf GREEN` | Only GREEN connected and local routes appear |
| 22 | Verify the BLUE VRF routing table if used | Router | `show ip route vrf BLUE` | Only BLUE connected and local routes appear |
| 23 | Test reachability inside the correct VRF | Router | `ping vrf RED 10.0.12.2` | Ping succeeds only if RED adjacency and addressing are correct |
| 24 | Confirm default/global ping does not accidentally use the VRF route | Router | `ping 10.0.12.2` | Ping should fail unless the global table also has a valid route |
| 25 | Save the working baseline | Router | `write memory` | VRF isolation baseline survives reload |
# VRF_Lite_Routing_Table_Isolation_Skeleton
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
 vrf forwarding RED
 ip address 10.0.1.1 255.255.255.0
 no shutdown
interface GigabitEthernet1/0
 vrf forwarding GREEN
 ip address 172.16.1.1 255.255.255.0
 no shutdown
interface GigabitEthernet2/0
 vrf forwarding BLUE
 ip address 192.168.1.1 255.255.255.0
 no shutdown
interface GigabitEthernet3/0.100
 vrf forwarding RED
 encapsulation dot1Q 100
 ip address 10.0.12.1 255.255.255.0
 no shutdown
interface GigabitEthernet3/0.200
 vrf forwarding GREEN
 encapsulation dot1Q 200
 ip address 172.16.12.1 255.255.255.0
 no shutdown
interface GigabitEthernet3/0.300
 vrf forwarding BLUE
 encapsulation dot1Q 300
 ip address 192.168.12.1 255.255.255.0
 no shutdown
end
write memory
# VRF_Lite_Routing_Table_Isolation_Verification_Commands
show vrf
show ip vrf
show vrf ipv4 unicast interfaces
show ip vrf interfaces
show ip interface brief
show running-config | section ^vrf definition
show running-config interface GigabitEthernet0/0
show running-config interface GigabitEthernet3/0.100
show ip route
show ip route vrf RED
show ip route vrf GREEN
show ip route vrf BLUE
ping vrf RED <RED_NEXT_HOP_OR_HOST>
ping vrf GREEN <GREEN_NEXT_HOP_OR_HOST>
ping vrf BLUE <BLUE_NEXT_HOP_OR_HOST>
traceroute vrf RED <RED_DESTINATION>
# VRF_Lite_Routing_Table_Isolation_Rollback
configure terminal
interface GigabitEthernet0/0
 no ip address
 no vrf forwarding RED
 shutdown
interface GigabitEthernet1/0
 no ip address
 no vrf forwarding GREEN
 shutdown
interface GigabitEthernet2/0
 no ip address
 no vrf forwarding BLUE
 shutdown
interface GigabitEthernet3/0.100
 no ip address
 no vrf forwarding RED
 no encapsulation dot1Q 100
 shutdown
interface GigabitEthernet3/0.200
 no ip address
 no vrf forwarding GREEN
 no encapsulation dot1Q 200
 shutdown
interface GigabitEthernet3/0.300
 no ip address
 no vrf forwarding BLUE
 no encapsulation dot1Q 300
 shutdown
no vrf definition RED
no vrf definition GREEN
no vrf definition BLUE
end
write memory
# VRF_Lite_Routing_Table_Isolation_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| VRF does not appear in verification | VRF was not created or wrong syntax was used | `show vrf` | Recreate with `vrf definition <VRF>` and `address-family ipv4 unicast` |
| Interface has no IP after VRF binding | IP was configured before `vrf forwarding` | `show running-config interface <interface>` | Reapply IP address after `vrf forwarding` |
| Interface appears in wrong VRF | Wrong VRF name or case mismatch | `show vrf ipv4 unicast interfaces` | Remove incorrect VRF binding and apply the correct case-sensitive VRF name |
| Connected route appears missing | Looking in the global table instead of the VRF table | `show ip route` then `show ip route vrf <VRF>` | Verify routes under the correct VRF table |
| Subinterface does not pass traffic | Missing or mismatched dot1Q VLAN tag | `show running-config interface <subinterface>` | Configure matching `encapsulation dot1Q <vlan-id>` on both ends |
| Ping fails even though IPs are configured | Ping was sourced from global table | `ping <destination>` then `ping vrf <VRF> <destination>` | Use `ping vrf <VRF> <destination>` |
| VRF route table is empty | Interface is down or not assigned to the VRF | `show vrf ipv4 unicast interfaces` | Fix interface state, cabling, encapsulation, or VRF binding |
| Multiple VRFs leak reachability unexpectedly | Static or dynamic route leaking exists elsewhere | `show running-config | include ip route vrf|route-target|import|export` | Remove unintended leaking configuration |
##### Source_Basis
# VRF_Lite_Routing_Table_Isolation_Mental_Model
# VRF_Lite_Routing_Table_Isolation_Configuration_Checklist
# VRF_Lite_Routing_Table_Isolation_Skeleton
# VRF_Lite_Routing_Table_Isolation_Verification_Commands
# VRF_Lite_Routing_Table_Isolation_Rollback
# VRF_Lite_Routing_Table_Isolation_Failure_Checks
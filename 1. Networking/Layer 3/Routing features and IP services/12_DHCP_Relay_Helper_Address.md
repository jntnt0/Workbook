DHCP_Relay_Helper_Address.md
# DHCP_Relay_Helper_Address

# DHCP_Relay_Helper_Address_Mental_Model

| Concept | Operational Meaning |
|---|---|
| DHCP relay | A Layer 3 device forwards DHCP client broadcasts to a DHCP server on another subnet |
| `ip helper-address` | Interface command that tells the gateway where to forward DHCP requests |
| Client broadcast boundary | DHCP Discover is broadcast by the client and does not cross a router by itself |
| Relay gateway | The routed interface or SVI closest to the client is where the helper address belongs |
| DHCP server subnet selection | The DHCP server uses the relay source or `giaddr` information to choose the correct pool |
| Multiple helpers | Multiple DHCP servers can be configured by adding multiple `ip helper-address` commands |
| UDP forwarding behavior | IOS helper forwarding is traditionally used for DHCP/BOOTP and other UDP services unless narrowed with UDP forwarding controls |
| Same-subnet DHCP | If the DHCP server and client are in the same subnet, relay is not needed |
| Remote DHCP | If the DHCP server is on a different subnet, the client gateway must relay the request |
| Return path | The DHCP server must be able to route replies back to the relay interface address |
| Pool dependency | Relay does not create addresses; the DHCP server must already have the correct scope or pool |
| Troubleshooting rule | Put the helper on the client-facing L3 interface, not on the server-facing interface |

# DHCP_Relay_Helper_Address_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify the client VLAN or subnet that needs DHCP relay | RTR/L3SW | `show ip interface brief` | Client gateway interface or SVI is identified |
| 2 | Confirm the client-facing Layer 3 interface is up | RTR/L3SW | `show ip interface brief` | Client gateway interface is `up/up` with the correct IP address |
| 3 | Confirm Layer 3 routing is enabled on a multilayer switch | L3SW | `show running-config | include ^ip routing` | `ip routing` is present |
| 4 | Enable Layer 3 routing if needed on the multilayer switch | L3SW | `conf t`<br>`ip routing`<br>`end` | Switch can route between VLANs |
| 5 | Confirm the DHCP server IP address is known | RTR/L3SW | `ping <DHCP_SERVER_IP>` | Relay device can reach the DHCP server |
| 6 | Confirm the relay device has a route to the DHCP server | RTR/L3SW | `show ip route <DHCP_SERVER_IP>` | Routing table has a valid path toward the DHCP server |
| 7 | Confirm the DHCP server has a return route to the client gateway or relay source subnet | DHCP Server/Upstream RTR | `show ip route <CLIENT_GATEWAY_IP>` | Server side can route replies back to the relay device |
| 8 | Confirm the DHCP server has the correct client subnet pool or scope | DHCP Server | `show running-config | section ip dhcp pool` | Server has a DHCP pool matching the remote client subnet |
| 9 | Configure DHCP relay on the client-facing interface or SVI | RTR/L3SW | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` ip helper-address <DHCP_SERVER_IP>`<br>`end` | DHCP broadcasts arriving on the client gateway are relayed to the DHCP server |
| 10 | Configure additional helper address if using redundant DHCP servers | RTR/L3SW | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` ip helper-address <SECONDARY_DHCP_SERVER_IP>`<br>`end` | Client DHCP requests are relayed to both configured DHCP servers |
| 11 | Verify helper address placement | RTR/L3SW | `show running-config interface <CLIENT_GATEWAY_INTERFACE>` | Client-facing interface contains the correct `ip helper-address` |
| 12 | Confirm helper is not accidentally placed only on the server-facing interface | RTR/L3SW | `show running-config | include ip helper-address` | Helper address is attached to the client gateway interface |
| 13 | Confirm client access VLAN exists if relay is on an SVI | L3SW | `show vlan brief` | Client VLAN exists and has active ports |
| 14 | Confirm trunk allows the client VLAN if relay is on an upstream SVI | L3SW | `show interfaces trunk` | Client VLAN is allowed and forwarding across trunks |
| 15 | Confirm SVI line protocol is up if using a VLAN interface | L3SW | `show ip interface brief | include Vlan<CLIENT_VLAN_ID>` | SVI is `up/up` |
| 16 | Renew DHCP lease on a client in the remote subnet | Client | `ipconfig /release`<br>`ipconfig /renew` | Client sends DHCP Discover and receives a lease if relay and server are correct |
| 17 | Verify DHCP binding on an IOS DHCP server if the server is IOS-based | DHCP Server | `show ip dhcp binding` | Client MAC or client ID is bound to an address from the correct remote pool |
| 18 | Verify DHCP pool utilization on an IOS DHCP server | DHCP Server | `show ip dhcp pool` | Correct pool shows leased and available addresses |
| 19 | Verify client received the expected address information | Client | `ipconfig /all` | Client has IP address, mask, gateway, DNS, DHCP server, and lease information |
| 20 | Verify client reaches its default gateway | Client | `ping <CLIENT_GATEWAY_IP>` | Client can reach the relay gateway |
| 21 | Verify client reaches remote networks after DHCP assignment | Client | `ping <REMOTE_IP>`<br>`tracert <REMOTE_IP>` | Client can route beyond its subnet |
| 22 | Optional: verify DHCP packet handling with server-side debug in lab | DHCP Server | `debug ip dhcp server events` | Server shows DHCP Discover, Offer, Request, and Ack activity |
| 23 | Optional: verify relay packet forwarding with packet debugging in lab | RTR/L3SW | `debug ip udp` | Relay activity is visible for forwarded UDP DHCP packets |
| 24 | Disable debugging after testing | RTR/L3SW | `undebug all` | Debugging is disabled |
| 25 | Save the working relay configuration | RTR/L3SW | `copy running-config startup-config` | Helper address configuration survives reload |

# DHCP_Relay_Helper_Address_Skeleton

conf t
!
! Required on multilayer switches acting as client gateways.
ip routing
!
interface <CLIENT_GATEWAY_INTERFACE>
 description CLIENT_DEFAULT_GATEWAY_DHCP_RELAY
 ip address <CLIENT_GATEWAY_IP> <CLIENT_SUBNET_MASK>
 ip helper-address <DHCP_SERVER_IP>
 no shutdown
!
end
write memory

# DHCP_Relay_Redundant_Server_Skeleton

conf t
!
interface <CLIENT_GATEWAY_INTERFACE>
 description CLIENT_DEFAULT_GATEWAY_DHCP_RELAY
 ip address <CLIENT_GATEWAY_IP> <CLIENT_SUBNET_MASK>
 ip helper-address <PRIMARY_DHCP_SERVER_IP>
 ip helper-address <SECONDARY_DHCP_SERVER_IP>
 no shutdown
!
end
write memory

# DHCP_Relay_SVI_Client_VLAN_Skeleton

conf t
!
ip routing
!
vlan <CLIENT_VLAN_ID>
 name <CLIENT_VLAN_NAME>
!
interface Vlan<CLIENT_VLAN_ID>
 description CLIENT_GATEWAY_DHCP_RELAY
 ip address <CLIENT_GATEWAY_IP> <CLIENT_SUBNET_MASK>
 ip helper-address <DHCP_SERVER_IP>
 no shutdown
!
interface <CLIENT_ACCESS_PORT>
 switchport mode access
 switchport access vlan <CLIENT_VLAN_ID>
 spanning-tree portfast
!
end
write memory

# DHCP_Server_Remote_Pool_Example_Skeleton

conf t
!
service dhcp
!
ip dhcp excluded-address <CLIENT_GATEWAY_IP>
ip dhcp excluded-address <LOW_RESERVED_IP> <HIGH_RESERVED_IP>
!
ip dhcp pool <REMOTE_CLIENT_POOL_NAME>
 network <CLIENT_SUBNET> <CLIENT_SUBNET_MASK>
 default-router <CLIENT_GATEWAY_IP>
 dns-server <DNS_SERVER_1> <DNS_SERVER_2>
 domain-name <DOMAIN_NAME>
 lease <DAYS> <HOURS> <MINUTES>
!
end
write memory

# DHCP_Relay_Helper_Address_Verification_Commands

show running-config | include ^ip routing
show ip interface brief
show running-config interface <CLIENT_GATEWAY_INTERFACE>
show running-config | include ip helper-address
show ip route <DHCP_SERVER_IP>
show ip route <CLIENT_SUBNET>
show vlan brief
show interfaces trunk
show spanning-tree interface <CLIENT_ACCESS_PORT> detail
show interfaces <CLIENT_ACCESS_PORT>
show ip arp
ping <DHCP_SERVER_IP>
ping <CLIENT_GATEWAY_IP>
debug ip udp
show debugging
undebug all

# IOS_DHCP_Server_Verification_Commands

show running-config | include ^service dhcp
show running-config | section ip dhcp pool
show running-config | include ^ip dhcp excluded-address
show ip dhcp pool
show ip dhcp binding
show ip dhcp conflict
show ip route <CLIENT_SUBNET>
debug ip dhcp server events
debug ip dhcp server packet
show debugging
undebug all

# Client_Verification_Commands

ipconfig /all
ipconfig /release
ipconfig /renew
ping <CLIENT_GATEWAY_IP>
ping <DHCP_SERVER_IP>
ping <DNS_SERVER_IP>
ping <REMOTE_IP>
nslookup <TEST_FQDN>
tracert <REMOTE_IP>

# DHCP_Relay_Helper_Address_Rollback

conf t
!
interface <CLIENT_GATEWAY_INTERFACE>
 no ip helper-address <DHCP_SERVER_IP>
 no ip helper-address <SECONDARY_DHCP_SERVER_IP>
!
end
write memory

# DHCP_Relay_SVI_Rollback

conf t
!
interface Vlan<CLIENT_VLAN_ID>
 no ip helper-address <DHCP_SERVER_IP>
 no ip helper-address <SECONDARY_DHCP_SERVER_IP>
!
end
write memory

# DHCP_Relay_Helper_Address_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Client receives APIPA address | DHCP Discover is not reaching a server | Client `ipconfig /all` and relay `show running-config interface <CLIENT_GATEWAY_INTERFACE>` | Add or correct `ip helper-address` on the client gateway |
| Helper is configured but client still fails | Helper is on the wrong interface | `show running-config | include ip helper-address` | Move helper to the client-facing routed interface or SVI |
| Relay cannot reach DHCP server | Missing route or broken upstream path | `show ip route <DHCP_SERVER_IP>` and `ping <DHCP_SERVER_IP>` | Fix routing, next hop, ACL, or upstream link |
| Server receives request but client gets no lease | DHCP server lacks correct remote subnet pool | Server `show running-config | section ip dhcp pool` | Create or correct pool matching the client subnet |
| Server offers wrong subnet | Pool selection is wrong because relay gateway/subnet is wrong | Client `ipconfig /all` and relay interface IP | Correct the client gateway IP, mask, or server pool |
| Client gets IP but wrong default gateway | DHCP pool has wrong `default-router` option | Client `ipconfig /all` | Set `default-router <CLIENT_GATEWAY_IP>` in the server pool |
| Client gets IP but no DNS | DHCP pool lacks DNS option | Client `ipconfig /all` | Add `dns-server <DNS_SERVER_IP>` to the DHCP pool |
| SVI is down/down | VLAN does not exist or has no active forwarding port | `show vlan brief` and `show ip interface brief` | Create VLAN, assign access port, or fix trunk |
| Client VLAN is not reaching relay SVI | Trunk does not allow VLAN | `show interfaces trunk` | Allow the client VLAN on the trunk |
| Client port is in wrong VLAN | Access port assignment is wrong | `show interfaces <CLIENT_ACCESS_PORT> switchport` | Move port to correct client VLAN |
| DHCP works for local subnet but not remote subnet | Relay missing on the remote subnet gateway | Remote gateway `show running-config interface <CLIENT_GATEWAY_INTERFACE>` | Add helper on each client subnet gateway that needs relay |
| DHCP reply cannot return to relay | Server lacks return route to relay/client subnet | Server-side `show ip route <CLIENT_SUBNET>` | Add route back to the client subnet through the relay path |
| Multiple servers give conflicting offers | Too many helper addresses or rogue DHCP server | Client lease details and DHCP snooping checks | Remove wrong helper or block rogue DHCP server |
| DHCP snooping blocks replies | Trusted boundary is wrong | `show ip dhcp snooping` and `show ip dhcp snooping statistics` | Trust legitimate DHCP server or relay uplink |
| DHCP pool is exhausted | No available addresses in the scope | Server `show ip dhcp pool` | Expand scope, reduce lease time, or clear stale bindings carefully |
| Address conflict prevents lease | Server conflict table contains the offered address | Server `show ip dhcp conflict` | Confirm address is unused, then clear stale conflict |
| IOS relay forwards unwanted UDP services | Default helper behavior forwards more than DHCP in some designs | `show running-config | include ip forward-protocol` | Limit forwarded protocols with `no ip forward-protocol udp <SERVICE>` where required |
| Relay debug is too noisy | Debugging left enabled | `show debugging` | Run `undebug all` |
| Client gets lease but cannot route off subnet | Gateway option or routing is wrong | Client `tracert <REMOTE_IP>` | Correct DHCP gateway option and upstream routing |
| Client can reach IP but not names | DNS option or DNS path is wrong | Client `nslookup <TEST_FQDN>` | Correct DHCP DNS option or DNS server reachability |
| Helper config disappears after reload | Configuration was not saved | `show startup-config interface <CLIENT_GATEWAY_INTERFACE>` | Reconfigure and save with `copy running-config startup-config` |

# Index

DHCP_Relay_Helper_Address.md
DHCP_Relay_Helper_Address
DHCP_Relay_Helper_Address_Mental_Model
DHCP_Relay_Helper_Address_Configuration_Checklist
DHCP_Relay_Helper_Address_Skeleton
DHCP_Relay_Redundant_Server_Skeleton
DHCP_Relay_SVI_Client_VLAN_Skeleton
DHCP_Server_Remote_Pool_Example_Skeleton
DHCP_Relay_Helper_Address_Verification_Commands
IOS_DHCP_Server_Verification_Commands
Client_Verification_Commands
DHCP_Relay_Helper_Address_Rollback
DHCP_Relay_SVI_Rollback
DHCP_Relay_Helper_Address_Failure_Checks
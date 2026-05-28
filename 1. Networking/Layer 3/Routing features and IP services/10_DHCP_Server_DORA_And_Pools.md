DHCP_Server_DORA_And_Pools.md
# DHCP_Server_DORA_And_Pools

# DHCP_Server_DORA_And_Pools_Mental_Model

| Concept | Operational Meaning |
|---|---|
| DHCP server | Router or multilayer switch leases IPv4 addressing information to clients |
| DORA | Discover, Offer, Request, Acknowledge is the normal DHCP lease process |
| DHCP Discover | Client broadcasts to find a DHCP server because it does not yet have a usable IP address |
| DHCP Offer | Server offers an available address and options from the matching pool |
| DHCP Request | Client requests the offered address from the selected DHCP server |
| DHCP Acknowledge | Server confirms the lease and sends final configuration options |
| DHCP pool | Defines the subnet and options the server can lease to clients |
| Excluded address | Address removed from dynamic leasing so infrastructure or static hosts do not get duplicated |
| Default router option | Tells clients which gateway to use for off-subnet traffic |
| DNS server option | Tells clients which DNS resolver to use |
| Lease timer | Controls how long the client may keep the address before renewal |
| Pool selection | The DHCP server selects a pool based on the client-facing interface subnet or relay gateway address |
| Binding database | Tracks leased IP address, client ID or MAC, and lease expiration |
| Conflict database | Tracks addresses the server believes are already in use |
| Relay boundary | If the client and server are on different subnets, use DHCP relay on the client gateway instead of expecting broadcasts to cross routers |
| Troubleshooting rule | If the client gets APIPA, check Layer 2, VLAN, trunks, gateway SVI, relay, pool, exclusions, and server bindings before blaming DHCP alone |

# DHCP_Server_DORA_And_Pools_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the DHCP server service is enabled | RTR/L3SW | `show running-config | include ^service dhcp` | `service dhcp` is present or absent because DHCP service is enabled by default |
| 2 | Enable DHCP service if it was disabled | RTR/L3SW | `conf t`<br>`service dhcp`<br>`end` | Device can operate as an IOS DHCP server |
| 3 | Confirm Layer 3 forwarding on a multilayer switch | L3SW | `show running-config | include ^ip routing` | `ip routing` is present if the switch routes between VLANs |
| 4 | Confirm client-facing interface or SVI is up | RTR/L3SW | `show ip interface brief` | Client gateway interface is `up/up` with the correct IPv4 address |
| 5 | Configure the client default gateway interface if needed | RTR/L3SW | `conf t`<br>`interface <CLIENT_GATEWAY_INTERFACE>`<br>` ip address <GATEWAY_IP> <MASK>`<br>` no shutdown`<br>`end` | Interface is active and owns the subnet that DHCP clients will use |
| 6 | Exclude the gateway address from leasing | RTR/L3SW | `conf t`<br>`ip dhcp excluded-address <GATEWAY_IP>`<br>`end` | Gateway address cannot be leased to a client |
| 7 | Exclude infrastructure and reserved address range | RTR/L3SW | `conf t`<br>`ip dhcp excluded-address <LOW_RESERVED_IP> <HIGH_RESERVED_IP>`<br>`end` | Reserved static addresses cannot be leased |
| 8 | Create the DHCP pool | RTR/L3SW | `conf t`<br>`ip dhcp pool <POOL_NAME>`<br>`end` | Named DHCP pool exists |
| 9 | Configure the lease network | RTR/L3SW | `conf t`<br>`ip dhcp pool <POOL_NAME>`<br>` network <CLIENT_SUBNET> <MASK>`<br>`end` | Pool is tied to the intended client subnet |
| 10 | Configure the default gateway option | RTR/L3SW | `conf t`<br>`ip dhcp pool <POOL_NAME>`<br>` default-router <GATEWAY_IP>`<br>`end` | Clients receive the correct default gateway |
| 11 | Configure DNS server option | RTR/L3SW | `conf t`<br>`ip dhcp pool <POOL_NAME>`<br>` dns-server <DNS_SERVER_1> <DNS_SERVER_2>`<br>`end` | Clients receive DNS resolver addresses |
| 12 | Configure domain name option if required | RTR/L3SW | `conf t`<br>`ip dhcp pool <POOL_NAME>`<br>` domain-name <DOMAIN_NAME>`<br>`end` | Clients receive DNS search domain information |
| 13 | Configure lease duration | RTR/L3SW | `conf t`<br>`ip dhcp pool <POOL_NAME>`<br>` lease <DAYS> <HOURS> <MINUTES>`<br>`end` | Clients receive the intended lease duration |
| 14 | Optional: configure TFTP server with option 150 for voice or boot use cases | RTR/L3SW | `conf t`<br>`ip dhcp pool <POOL_NAME>`<br>` option 150 ip <TFTP_SERVER_IP>`<br>`end` | Clients that use option 150 receive the TFTP server address |
| 15 | Optional: configure WINS server if legacy clients require it | RTR/L3SW | `conf t`<br>`ip dhcp pool <POOL_NAME>`<br>` netbios-name-server <WINS_SERVER_IP>`<br>`end` | Legacy clients receive WINS server information |
| 16 | Optional: configure a DHCP pool default gateway with multiple routers | RTR/L3SW | `conf t`<br>`ip dhcp pool <POOL_NAME>`<br>` default-router <GATEWAY_IP_1> <GATEWAY_IP_2>`<br>`end` | Clients receive multiple gateway options where supported |
| 17 | Verify DHCP pool configuration | RTR/L3SW | `show running-config | section ip dhcp pool` | Pool contains network, default-router, DNS, lease, and required options |
| 18 | Verify excluded addresses | RTR/L3SW | `show running-config | include ^ip dhcp excluded-address` | Gateway and static infrastructure ranges are excluded |
| 19 | Verify DHCP pool utilization | RTR/L3SW | `show ip dhcp pool` | Pool shows total, leased, excluded, and available address counts |
| 20 | Connect or renew a DHCP client | Client | `ipconfig /release`<br>`ipconfig /renew` | Client requests a new lease |
| 21 | Verify DHCP binding was created | RTR/L3SW | `show ip dhcp binding` | Client MAC or client ID maps to a leased IP address |
| 22 | Verify DHCP conflict table | RTR/L3SW | `show ip dhcp conflict` | No unexpected conflicts exist |
| 23 | Verify client received the expected address | Client | `ipconfig /all` | Client has IP, mask, default gateway, DNS, and lease information from the pool |
| 24 | Verify client can reach default gateway | Client | `ping <GATEWAY_IP>` | Client reaches its default gateway |
| 25 | Verify client can resolve DNS if DNS was assigned | Client | `nslookup <TEST_FQDN>` | Client resolves names through the assigned DNS server |
| 26 | Verify end-to-end routing beyond the gateway | Client | `ping <REMOTE_IP>`<br>`tracert <REMOTE_IP>` | Client reaches routed destinations if upstream routing is correct |
| 27 | Optional: observe DHCP server events during troubleshooting | RTR/L3SW | `debug ip dhcp server events` | Server shows lease events such as Discover, Offer, Request, and Ack |
| 28 | Optional: observe DHCP packet details during troubleshooting | RTR/L3SW | `debug ip dhcp server packet` | Server shows detailed DHCP packet handling |
| 29 | Disable debugging after testing | RTR/L3SW | `undebug all` | Debugging is disabled |
| 30 | Save the working DHCP configuration | RTR/L3SW | `copy running-config startup-config` | DHCP server configuration survives reload |

# DHCP_Server_DORA_And_Pools_Skeleton

conf t
!
service dhcp
!
! Required on multilayer switches if routing between VLANs.
ip routing
!
interface <CLIENT_GATEWAY_INTERFACE>
 description CLIENT_DEFAULT_GATEWAY
 ip address <GATEWAY_IP> <MASK>
 no shutdown
!
ip dhcp excluded-address <GATEWAY_IP>
ip dhcp excluded-address <LOW_RESERVED_IP> <HIGH_RESERVED_IP>
!
ip dhcp pool <POOL_NAME>
 network <CLIENT_SUBNET> <MASK>
 default-router <GATEWAY_IP>
 dns-server <DNS_SERVER_1> <DNS_SERVER_2>
 domain-name <DOMAIN_NAME>
 lease <DAYS> <HOURS> <MINUTES>
!
end
write memory

# DHCP_Server_With_Option_150_Skeleton

conf t
!
ip dhcp excluded-address <GATEWAY_IP>
ip dhcp excluded-address <LOW_RESERVED_IP> <HIGH_RESERVED_IP>
!
ip dhcp pool <VOICE_OR_BOOT_POOL_NAME>
 network <CLIENT_SUBNET> <MASK>
 default-router <GATEWAY_IP>
 dns-server <DNS_SERVER_1> <DNS_SERVER_2>
 option 150 ip <TFTP_SERVER_IP>
 lease <DAYS> <HOURS> <MINUTES>
!
end
write memory

# DHCP_Server_SVI_Client_VLAN_Skeleton

conf t
!
ip routing
service dhcp
!
vlan <CLIENT_VLAN_ID>
 name <CLIENT_VLAN_NAME>
!
interface Vlan<CLIENT_VLAN_ID>
 description CLIENT_GATEWAY_AND_DHCP_POOL_SELECTOR
 ip address <GATEWAY_IP> <MASK>
 no shutdown
!
interface <CLIENT_ACCESS_PORT>
 switchport mode access
 switchport access vlan <CLIENT_VLAN_ID>
 spanning-tree portfast
!
ip dhcp excluded-address <GATEWAY_IP>
ip dhcp excluded-address <LOW_RESERVED_IP> <HIGH_RESERVED_IP>
!
ip dhcp pool <POOL_NAME>
 network <CLIENT_SUBNET> <MASK>
 default-router <GATEWAY_IP>
 dns-server <DNS_SERVER_1> <DNS_SERVER_2>
 domain-name <DOMAIN_NAME>
 lease <DAYS> <HOURS> <MINUTES>
!
end
write memory

# DHCP_Server_DORA_And_Pools_Verification_Commands

show running-config | include ^service dhcp
show running-config | include ^ip routing
show ip interface brief
show running-config interface <CLIENT_GATEWAY_INTERFACE>
show running-config | include ^ip dhcp excluded-address
show running-config | section ip dhcp pool
show ip dhcp pool
show ip dhcp binding
show ip dhcp conflict
show ip route connected
show interfaces <CLIENT_GATEWAY_INTERFACE>
show vlan brief
show interfaces trunk
show spanning-tree interface <CLIENT_ACCESS_PORT> detail
show ip arp
ping <CLIENT_IP>
ping <GATEWAY_IP>
debug ip dhcp server events
debug ip dhcp server packet
show debugging
undebug all

# Client_Verification_Commands

ipconfig /all
ipconfig /release
ipconfig /renew
ping <GATEWAY_IP>
ping <DNS_SERVER_IP>
ping <REMOTE_IP>
nslookup <TEST_FQDN>
tracert <REMOTE_IP>

# DHCP_Server_DORA_And_Pools_Rollback

conf t
!
no ip dhcp pool <POOL_NAME>
no ip dhcp excluded-address <GATEWAY_IP>
no ip dhcp excluded-address <LOW_RESERVED_IP> <HIGH_RESERVED_IP>
!
end
write memory

# DHCP_Server_Interface_Rollback

conf t
!
interface <CLIENT_GATEWAY_INTERFACE>
 no ip address <GATEWAY_IP> <MASK>
 shutdown
!
end
write memory

# DHCP_Server_DORA_And_Pools_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Client receives APIPA address | Client did not receive a DHCP offer | Client `ipconfig /all` | Check VLAN, access port, trunk, SVI, DHCP pool, and server reachability |
| No DHCP binding appears | Client Discover never reached server or pool did not match | `show ip dhcp binding` and `debug ip dhcp server events` | Fix client VLAN path or pool network |
| Pool has no available addresses | Scope exhausted | `show ip dhcp pool` | Increase pool size, shorten lease, or clear stale bindings carefully |
| Server leases gateway address | Gateway was not excluded | `show running-config | include ^ip dhcp excluded-address` | Add `ip dhcp excluded-address <GATEWAY_IP>` |
| Duplicate IP conflicts appear | Exclusions missing or static host overlaps pool | `show ip dhcp conflict` | Exclude static ranges and clear conflict after fixing overlap |
| Client receives wrong default gateway | Wrong `default-router` option | `show running-config | section ip dhcp pool` | Correct `default-router <GATEWAY_IP>` |
| Client receives wrong DNS server | Wrong `dns-server` option | Client `ipconfig /all` | Correct `dns-server <DNS_SERVER_IP>` in pool |
| Client receives an address from wrong subnet | Wrong pool `network` statement or wrong client-facing interface | `show running-config | section ip dhcp pool` and `show ip interface brief` | Correct pool network or gateway interface IP |
| DHCP server does not respond | DHCP service disabled | `show running-config | include ^no service dhcp` | Configure `service dhcp` |
| Client-facing SVI is down | VLAN missing or no active ports in VLAN | `show ip interface brief` and `show vlan brief` | Create VLAN, assign access port, and bring port up |
| DHCP works on same VLAN but not across router | Missing relay on client gateway | `show running-config interface <CLIENT_GATEWAY_INTERFACE>` | Configure `ip helper-address <DHCP_SERVER_IP>` on client gateway |
| DHCP relay sends to wrong server | Incorrect helper address | `show running-config interface <CLIENT_GATEWAY_INTERFACE>` | Remove wrong helper and add correct DHCP server IP |
| DHCP offer reaches client but client still fails | DHCP snooping blocks server replies or trust boundary is wrong | `show ip dhcp snooping` and `show ip dhcp snooping statistics` | Trust the port toward the legitimate DHCP server or relay |
| Client cannot reach off-subnet networks | Missing or wrong default gateway option | Client `ipconfig /all` and `show running-config | section ip dhcp pool` | Correct `default-router` |
| Client can reach gateway but not remote networks | Upstream routing problem | `show ip route` and client `tracert <REMOTE_IP>` | Fix upstream and return routing |
| Client can ping IP but not names | DNS option or DNS reachability problem | Client `nslookup <TEST_FQDN>` | Correct DHCP DNS option or DNS server reachability |
| Lease duration causes pool exhaustion | Lease is too long for client churn | `show ip dhcp pool` | Reduce lease time with `lease <DAYS> <HOURS> <MINUTES>` |
| Lease duration is too short | Clients renew too frequently | `show ip dhcp binding` and client logs | Increase lease time |
| Option 150 phones fail to find TFTP | Missing or wrong option 150 | `show running-config | section ip dhcp pool` | Configure `option 150 ip <TFTP_SERVER_IP>` |
| Bindings survive after client disconnects | Normal lease behavior until expiration | `show ip dhcp binding` | Clear binding only if safe, or shorten lease |
| Debug output is noisy | DHCP debug left enabled | `show debugging` | Run `undebug all` |
| Configuration disappears after reload | Config was not saved | `show startup-config | section ip dhcp pool` | Reconfigure and save with `copy running-config startup-config` |

# Index

DHCP_Server_DORA_And_Pools.md
DHCP_Server_DORA_And_Pools
DHCP_Server_DORA_And_Pools_Mental_Model
DHCP_Server_DORA_And_Pools_Configuration_Checklist
DHCP_Server_DORA_And_Pools_Skeleton
DHCP_Server_With_Option_150_Skeleton
DHCP_Server_SVI_Client_VLAN_Skeleton
DHCP_Server_DORA_And_Pools_Verification_Commands
Client_Verification_Commands
DHCP_Server_DORA_And_Pools_Rollback
DHCP_Server_Interface_Rollback
DHCP_Server_DORA_And_Pools_Failure_Checks
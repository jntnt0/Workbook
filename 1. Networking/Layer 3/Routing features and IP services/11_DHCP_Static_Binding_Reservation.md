DHCP_Static_Binding_Reservation.md
# DHCP_Static_Binding_Reservation

# DHCP_Static_Binding_Reservation_Mental_Model

| Concept | Operational Meaning |
|---|---|
| DHCP static binding | DHCP server always leases a specific IP address to a specific client identity |
| Reservation | Operational term for static binding; the client still uses DHCP, but the assigned address is predictable |
| Not a manual host IP | The endpoint remains DHCP-configured; the server controls the reserved address |
| Binding identity | The server matches the client using `client-identifier` or `hardware-address` |
| Client identifier | DHCP option 61 value used by many clients; often not identical to the raw MAC address |
| Hardware address | MAC-based match used when the client identifier is unavailable or the platform expects MAC matching |
| Host command | Defines the exact reserved IP address and mask inside a manual DHCP pool |
| One host per manual pool | Classic IOS DHCP static bindings are normally built as individual pools for individual clients |
| Options inheritance boundary | Static binding pools should explicitly define default router, DNS, domain, lease, and any required options if the client must receive them |
| Dynamic pool boundary | Exclude or avoid the reserved address from dynamic pools so another client cannot receive it |
| Binding database | `show ip dhcp binding` proves which client received which address |
| Conflict database | `show ip dhcp conflict` shows addresses the server believes are already in use |
| Troubleshooting rule | If the reservation does not work, verify the client identity first; wrong client identifier is the usual failure |

# DHCP_Static_Binding_Reservation_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm DHCP service is enabled | RTR/L3SW | `show running-config | include ^service dhcp` | `service dhcp` is present or DHCP service is enabled by default |
| 2 | Enable DHCP service if it was disabled | RTR/L3SW | `conf t`<br>`service dhcp`<br>`end` | Device can answer DHCP requests |
| 3 | Confirm Layer 3 forwarding on multilayer switches | L3SW | `show running-config | include ^ip routing` | `ip routing` is present when the switch is acting as a routed gateway |
| 4 | Confirm client gateway interface or SVI is up | RTR/L3SW | `show ip interface brief` | Client-facing interface is `up/up` with the correct gateway IP address |
| 5 | Confirm the DHCP client can reach the server path | RTR/L3SW | `show vlan brief`<br>`show interfaces trunk`<br>`show running-config interface <CLIENT_GATEWAY_INTERFACE>` | VLAN, trunk, SVI, routed interface, or relay path is correct |
| 6 | Identify the reserved client MAC address | Client/Switch | `show mac address-table address <CLIENT_MAC>` | Client MAC is seen on the expected access port or VLAN |
| 7 | Identify the DHCP client identifier if already leased | RTR/L3SW | `show ip dhcp binding` | Existing binding shows the client ID or hardware address used by the DHCP server |
| 8 | Clear stale dynamic binding for the reserved client if needed | RTR/L3SW | `clear ip dhcp binding <CURRENT_CLIENT_IP>` | Old dynamic lease is removed from the DHCP server database |
| 9 | Check whether the planned reservation IP is already leased | RTR/L3SW | `show ip dhcp binding <RESERVED_IP>` | Reserved IP is not currently leased to another client |
| 10 | Check whether the planned reservation IP is in conflict state | RTR/L3SW | `show ip dhcp conflict` | Reserved IP is not listed as a conflict |
| 11 | Clear a stale conflict only after confirming the address is safe | RTR/L3SW | `clear ip dhcp conflict <RESERVED_IP>` | Stale conflict is removed |
| 12 | Exclude the reserved IP from the dynamic pool range | RTR/L3SW | `conf t`<br>`ip dhcp excluded-address <RESERVED_IP>`<br>`end` | Reserved IP cannot be dynamically leased to another client |
| 13 | Confirm dynamic pool does not unintentionally assign the reserved IP | RTR/L3SW | `show running-config | section ip dhcp pool` | Dynamic pool and exclusions do not overlap incorrectly |
| 14 | Create the manual DHCP pool for the reserved client | RTR/L3SW | `conf t`<br>`ip dhcp pool <STATIC_POOL_NAME>`<br>`end` | Static binding pool exists |
| 15 | Define the reserved host address and mask | RTR/L3SW | `conf t`<br>`ip dhcp pool <STATIC_POOL_NAME>`<br>` host <RESERVED_IP> <MASK>`<br>`end` | Pool reserves the exact host address for the client |
| 16 | Configure reservation by client identifier when option 61 is known | RTR/L3SW | `conf t`<br>`ip dhcp pool <STATIC_POOL_NAME>`<br>` client-identifier <CLIENT_IDENTIFIER>`<br>`end` | DHCP server matches the reservation using the DHCP client identifier |
| 17 | Configure reservation by hardware address when MAC matching is required | RTR/L3SW | `conf t`<br>`ip dhcp pool <STATIC_POOL_NAME>`<br>` hardware-address <CLIENT_MAC> ieee802`<br>`end` | DHCP server matches the reservation using the client MAC address |
| 18 | Configure client name for operational clarity | RTR/L3SW | `conf t`<br>`ip dhcp pool <STATIC_POOL_NAME>`<br>` client-name <CLIENT_NAME>`<br>`end` | Binding is easier to identify during operations |
| 19 | Configure the default gateway option | RTR/L3SW | `conf t`<br>`ip dhcp pool <STATIC_POOL_NAME>`<br>` default-router <GATEWAY_IP>`<br>`end` | Reserved client receives the correct gateway |
| 20 | Configure DNS server option | RTR/L3SW | `conf t`<br>`ip dhcp pool <STATIC_POOL_NAME>`<br>` dns-server <DNS_SERVER_1> <DNS_SERVER_2>`<br>`end` | Reserved client receives DNS resolver addresses |
| 21 | Configure domain name option if used | RTR/L3SW | `conf t`<br>`ip dhcp pool <STATIC_POOL_NAME>`<br>` domain-name <DOMAIN_NAME>`<br>`end` | Reserved client receives the DNS search domain |
| 22 | Configure lease timer if the reservation should use a defined renewal interval | RTR/L3SW | `conf t`<br>`ip dhcp pool <STATIC_POOL_NAME>`<br>` lease <DAYS> <HOURS> <MINUTES>`<br>`end` | Reserved client receives the intended lease duration |
| 23 | Optional: configure option 150 for phone or boot clients | RTR/L3SW | `conf t`<br>`ip dhcp pool <STATIC_POOL_NAME>`<br>` option 150 ip <TFTP_SERVER_IP>`<br>`end` | Reserved client receives the required TFTP server option |
| 24 | Verify the static pool configuration | RTR/L3SW | `show running-config | section ip dhcp pool <STATIC_POOL_NAME>` | Pool contains host address, identity match, and required options |
| 25 | Verify DHCP pool status | RTR/L3SW | `show ip dhcp pool` | Static pool is present and pool accounting looks correct |
| 26 | Renew the client DHCP lease | Client | `ipconfig /release`<br>`ipconfig /renew` | Client requests a new DHCP lease |
| 27 | Verify the reserved binding is active | RTR/L3SW | `show ip dhcp binding <RESERVED_IP>` | Reserved IP is bound to the expected client identifier or MAC |
| 28 | Verify client-side address details | Client | `ipconfig /all` | Client has the reserved IP, mask, default gateway, DNS, and lease details |
| 29 | Verify gateway reachability from the reserved client | Client | `ping <GATEWAY_IP>` | Reserved client reaches its default gateway |
| 30 | Verify DNS behavior if DNS options were provided | Client | `nslookup <TEST_FQDN>` | Reserved client resolves names using the assigned DNS server |
| 31 | Verify routed reachability beyond the gateway | Client | `ping <REMOTE_IP>`<br>`tracert <REMOTE_IP>` | Reserved client reaches remote networks if routing is correct |
| 32 | Optional: debug DHCP server events during reservation testing | RTR/L3SW | `debug ip dhcp server events` | DHCP server shows Discover, Offer, Request, and Ack activity |
| 33 | Optional: debug DHCP packet details when identity matching fails | RTR/L3SW | `debug ip dhcp server packet` | DHCP packet output reveals client identifier or hardware address behavior |
| 34 | Disable debugging after testing | RTR/L3SW | `undebug all` | DHCP debug is disabled |
| 35 | Save the working reservation configuration | RTR/L3SW | `copy running-config startup-config` | Static binding survives reload |

# DHCP_Static_Binding_Reservation_Skeleton

conf t
!
service dhcp
!
! Required on multilayer switches acting as routed gateways.
ip routing
!
interface <CLIENT_GATEWAY_INTERFACE>
 description CLIENT_DEFAULT_GATEWAY
 ip address <GATEWAY_IP> <MASK>
 no shutdown
!
! Keep the reserved address out of the dynamic lease space.
ip dhcp excluded-address <RESERVED_IP>
!
ip dhcp pool <STATIC_POOL_NAME>
 host <RESERVED_IP> <MASK>
 client-identifier <CLIENT_IDENTIFIER>
 client-name <CLIENT_NAME>
 default-router <GATEWAY_IP>
 dns-server <DNS_SERVER_1> <DNS_SERVER_2>
 domain-name <DOMAIN_NAME>
 lease <DAYS> <HOURS> <MINUTES>
!
end
write memory

# DHCP_Static_Binding_Hardware_Address_Skeleton

conf t
!
ip dhcp excluded-address <RESERVED_IP>
!
ip dhcp pool <STATIC_POOL_NAME>
 host <RESERVED_IP> <MASK>
 hardware-address <CLIENT_MAC> ieee802
 client-name <CLIENT_NAME>
 default-router <GATEWAY_IP>
 dns-server <DNS_SERVER_1> <DNS_SERVER_2>
 domain-name <DOMAIN_NAME>
 lease <DAYS> <HOURS> <MINUTES>
!
end
write memory

# DHCP_Static_Binding_With_Option_150_Skeleton

conf t
!
ip dhcp excluded-address <RESERVED_IP>
!
ip dhcp pool <STATIC_POOL_NAME>
 host <RESERVED_IP> <MASK>
 client-identifier <CLIENT_IDENTIFIER>
 client-name <CLIENT_NAME>
 default-router <GATEWAY_IP>
 dns-server <DNS_SERVER_1> <DNS_SERVER_2>
 option 150 ip <TFTP_SERVER_IP>
 lease <DAYS> <HOURS> <MINUTES>
!
end
write memory

# DHCP_Static_Binding_Reservation_Verification_Commands

show running-config | include ^service dhcp
show running-config | include ^ip routing
show ip interface brief
show running-config interface <CLIENT_GATEWAY_INTERFACE>
show running-config | include ^ip dhcp excluded-address
show running-config | section ip dhcp pool
show running-config | section ip dhcp pool <STATIC_POOL_NAME>
show ip dhcp pool
show ip dhcp binding
show ip dhcp binding <RESERVED_IP>
show ip dhcp conflict
show ip route connected
show ip arp
show mac address-table address <CLIENT_MAC>
show vlan brief
show interfaces trunk
show interfaces <CLIENT_ACCESS_PORT>
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

# DHCP_Static_Binding_Reservation_Rollback

conf t
!
no ip dhcp pool <STATIC_POOL_NAME>
no ip dhcp excluded-address <RESERVED_IP>
!
end
write memory

# DHCP_Static_Binding_Client_Reset

clear ip dhcp binding <RESERVED_IP>
clear ip dhcp conflict <RESERVED_IP>

# DHCP_Static_Binding_Reservation_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| Client receives a dynamic address instead of the reservation | Client identity does not match the static binding | `show ip dhcp binding` and `debug ip dhcp server packet` | Correct `client-identifier` or use `hardware-address <MAC> ieee802` |
| Client receives APIPA address | DHCP server did not offer a lease | Client `ipconfig /all` and `debug ip dhcp server events` | Check VLAN, SVI, relay, pool syntax, and DHCP service |
| Reserved IP is leased to another client | Reserved IP was not excluded from the dynamic pool | `show ip dhcp binding <RESERVED_IP>` | Exclude the address and clear the wrong binding during a maintenance window |
| Static pool exists but client does not match it | Wrong MAC format or wrong client identifier format | `debug ip dhcp server packet` | Copy the actual client identifier seen by the server into the static pool |
| Client identifier looks different from MAC address | DHCP option 61 is being used | `show ip dhcp binding` | Match using `client-identifier` instead of assuming raw MAC matching |
| Static binding fails after NIC replacement | Client identity changed | `show mac address-table` and client `ipconfig /all` | Update the `client-identifier` or `hardware-address` |
| Pool command is accepted but client lacks gateway | Static pool missing `default-router` | Client `ipconfig /all` | Add `default-router <GATEWAY_IP>` under the static pool |
| Client can reach gateway but not names | Static pool missing or has wrong DNS option | Client `nslookup <TEST_FQDN>` | Add or correct `dns-server <DNS_SERVER_IP>` |
| Client gets correct IP but cannot reach off-subnet destinations | Wrong gateway option or upstream routing issue | Client `ipconfig /all` and `tracert <REMOTE_IP>` | Correct default router option or fix upstream routing |
| Reservation IP shows conflict | Address is already used or stale conflict remains | `show ip dhcp conflict` | Confirm address is free, then clear stale conflict |
| `show ip dhcp binding` has stale lease | Client previously received another address | `show ip dhcp binding` | Clear stale binding and renew client lease |
| Static pool overlaps dynamic pool | Dynamic scope can still hand out reservation | `show running-config | section ip dhcp pool` | Exclude reservation or shrink dynamic range |
| Wrong subnet mask is assigned | `host` mask is wrong | Client `ipconfig /all` | Correct `host <RESERVED_IP> <MASK>` |
| Wrong reservation pool is selected | Multiple pools or relay gateway address point to another subnet | `show ip dhcp pool` and relay interface config | Correct subnet, relay gateway, and pool definitions |
| DHCP relay path fails for reserved client | Missing or wrong helper address on client gateway | `show running-config interface <CLIENT_GATEWAY_INTERFACE>` | Add `ip helper-address <DHCP_SERVER_IP>` on the client gateway |
| Local same-VLAN reservation works but remote reservation does not | Relay or routing issue between gateway and server | `ping <DHCP_SERVER_IP>` from gateway | Fix relay, routing, or ACL path |
| DHCP snooping blocks replies | Server-facing port is not trusted | `show ip dhcp snooping` | Trust the legitimate DHCP server or relay uplink |
| Client renews but keeps old address | Client did not release old lease or OS cached lease | Client `ipconfig /release` and `ipconfig /renew` | Force lease renewal or reboot client |
| Option 150 device fails to find TFTP | Static pool missing option 150 | `show running-config | section ip dhcp pool <STATIC_POOL_NAME>` | Add `option 150 ip <TFTP_SERVER_IP>` |
| Debug output floods console | DHCP debug left enabled | `show debugging` | Run `undebug all` |
| Reservation disappears after reload | Configuration not saved | `show startup-config | section ip dhcp pool` | Reconfigure and save with `copy running-config startup-config` |

# Index

DHCP_Static_Binding_Reservation.md
DHCP_Static_Binding_Reservation
DHCP_Static_Binding_Reservation_Mental_Model
DHCP_Static_Binding_Reservation_Configuration_Checklist
DHCP_Static_Binding_Reservation_Skeleton
DHCP_Static_Binding_Hardware_Address_Skeleton
DHCP_Static_Binding_With_Option_150_Skeleton
DHCP_Static_Binding_Reservation_Verification_Commands
Client_Verification_Commands
DHCP_Static_Binding_Reservation_Rollback
DHCP_Static_Binding_Client_Reset
DHCP_Static_Binding_Reservation_Failure_Checks
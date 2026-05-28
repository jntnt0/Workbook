Source checked: Cisco IOS XE 17.x documents rotary destination NAT with a rotary pool, an ACL matching the virtual host, ip nat inside destination list <acl> pool <pool>, and classic ip nat inside / ip nat outside interface roles. Cisco states that matching destination addresses are replaced from the rotary pool on a round-robin basis when a new outside-to-inside connection is opened.  

NAT_Inside_Destination_Rotary_Translation.md

NAT_Inside_Destination_Rotary_Translation

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | `iosxe_combined_pdfs_.md` NAT guide reference | Points IOS XE NAT configuration content to the IOS XE combined source |
| `iosxe_combined_pdfs_.md` | `NAT Configuration Guide` > `Configuring NAT for IP Address Conservation` | Supports classic NAT inside/outside roles and NAT translation behavior |
| `iosxe_combined_pdfs_.md` | `Avoiding Server Overload Using TCP Load Balancing` | Supports rotary destination NAT, virtual host ACL matching, and real-host rotary pools |
| Cisco IOS XE 17.x IP Addressing Configuration Guide | `Configuring Server TCP Load Balancing` | Confirms `ip nat pool ... type rotary`, `access-list ... permit <virtual-host>`, and `ip nat inside destination list <acl> pool <pool>` |
| Related lab | `nat-inside-destination-final` | Lab target for this mechanism note |
# NAT_Inside_Destination_Rotary_Translation_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Inside destination NAT | The router rewrites the destination address of outside-to-inside traffic. This is different from normal inside source NAT, which rewrites the source address of inside-to-outside traffic. |
| Virtual host | The outside client connects to one advertised destination IP. That IP is not the final real server address. |
| Real hosts | The rotary pool contains the real inside server addresses that should receive connections. |
| Rotary pool | A NAT pool marked with `type rotary`. New connections are distributed across the real-host addresses in the pool. |
| ACL purpose | The ACL matches the virtual destination address. It does not match the real servers. |
| Translation direction | Outside clients target the virtual address. The NAT router changes the destination to one real inside server address. |
| TCP load-sharing model | Each new outside-to-inside TCP connection can be assigned to a different real host from the rotary pool. |
| Not full load balancing | Rotary NAT does not check server health, load, CPU, or application response. It is simple round-robin destination translation. |
| Routing dependency | The NAT router still needs routes toward the real inside servers and toward outside clients. NAT does not replace routing. |
| NAT domain dependency | Classic rotary destination NAT still depends on correct `ip nat inside` and `ip nat outside` interface markings. |
# NAT_Inside_Destination_Rotary_Translation_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the NAT router interfaces are up | NAT-RTR | `show ip interface brief` | Inside and outside interfaces show up/up |
| 2 | Confirm the real inside servers are reachable from the NAT router | NAT-RTR | `ping <REAL_SERVER_1_IP>` | NAT router can reach the first real server |
| 3 | Confirm the second real inside server is reachable | NAT-RTR | `ping <REAL_SERVER_2_IP>` | NAT router can reach the second real server |
| 4 | Confirm the NAT router has an outside path | NAT-RTR | `show ip route <OUTSIDE_CLIENT_IP>` | Route points toward the outside interface or next hop |
| 5 | Enter global configuration mode | NAT-RTR | `configure terminal` | Device enters configuration mode |
| 6 | Create a rotary pool containing real inside servers | NAT-RTR | `ip nat pool <ROTARY_POOL_NAME> <REAL_SERVER_START_IP> <REAL_SERVER_END_IP> prefix-length <PREFIX_LENGTH> type rotary` | Pool is created with real inside server addresses and marked as rotary |
| 7 | Create an ACL matching the virtual host address | NAT-RTR | `access-list <ACL_ID> permit <VIRTUAL_HOST_IP>` | ACL permits the destination address outside clients will target |
| 8 | Bind the virtual host ACL to the rotary real-host pool | NAT-RTR | `ip nat inside destination list <ACL_ID> pool <ROTARY_POOL_NAME>` | Matching destination traffic is translated to real hosts from the rotary pool |
| 9 | Enter the inside-facing interface | NAT-RTR | `interface <INSIDE_INTERFACE>` | Device enters inside interface configuration mode |
| 10 | Mark the server-side interface as NAT inside | NAT-RTR | `ip nat inside` | Interface is classified as the inside NAT domain |
| 11 | Return to global configuration mode | NAT-RTR | `exit` | Device returns to global configuration mode |
| 12 | Enter the outside-facing interface | NAT-RTR | `interface <OUTSIDE_INTERFACE>` | Device enters outside interface configuration mode |
| 13 | Mark the client-side interface as NAT outside | NAT-RTR | `ip nat outside` | Interface is classified as the outside NAT domain |
| 14 | Return to privileged EXEC mode | NAT-RTR | `end` | Device exits configuration mode |
| 15 | Confirm the NAT rule is present | NAT-RTR | `show running-config | include ip nat` | Rotary pool and inside destination NAT rule are visible |
| 16 | Confirm the ACL matches the virtual host only | NAT-RTR | `show access-lists <ACL_ID>` | ACL permits the virtual host address, not the real-server range |
| 17 | Confirm NAT inside/outside interface roles | NAT-RTR | `show ip nat statistics` | Inside and outside interfaces are listed correctly |
| 18 | Generate the first outside-to-inside connection | Outside Host | `telnet <VIRTUAL_HOST_IP> <TCP_PORT>` | First connection is translated to one real inside server |
| 19 | Generate another new outside-to-inside connection | Outside Host | `telnet <VIRTUAL_HOST_IP> <TCP_PORT>` | Another new connection can be assigned to the next real server |
| 20 | Verify translation table behavior | NAT-RTR | `show ip nat translations` | Virtual host destination is translated to real inside server addresses |
| 21 | Verify detailed translation state | NAT-RTR | `show ip nat translations verbose` | Output shows protocol, inside local/global values, and active session details |
| 22 | Save the working configuration | NAT-RTR | `write memory` | Rotary destination NAT configuration survives reload |
# NAT_Inside_Destination_Rotary_Translation_Skeleton
conf t
!
! Real inside servers live in this pool.
! Outside clients do not connect directly to these addresses.
ip nat pool <ROTARY_POOL_NAME> <REAL_SERVER_START_IP> <REAL_SERVER_END_IP> prefix-length <PREFIX_LENGTH> type rotary
!
! ACL matches the virtual host address that outside clients target.
access-list <ACL_ID> permit <VIRTUAL_HOST_IP>
!
! Translate outside-to-inside destination traffic from virtual host to real hosts.
ip nat inside destination list <ACL_ID> pool <ROTARY_POOL_NAME>
!
interface <INSIDE_INTERFACE>
 description SERVER_SIDE_INSIDE
 ip address <INSIDE_INTERFACE_IP> <INSIDE_MASK>
 ip nat inside
 no shutdown
!
interface <OUTSIDE_INTERFACE>
 description CLIENT_SIDE_OUTSIDE
 ip address <OUTSIDE_INTERFACE_IP> <OUTSIDE_MASK>
 ip nat outside
 no shutdown
!
ip route <REAL_SERVER_SUBNET> <REAL_SERVER_MASK> <INSIDE_NEXT_HOP>
ip route 0.0.0.0 0.0.0.0 <OUTSIDE_NEXT_HOP>
!
end
write memory
# NAT_Inside_Destination_Rotary_Translation_Verification_Commands
show ip interface brief
show running-config | include ip nat
show running-config | section interface
show access-lists
show access-lists <ACL_ID>
show ip route <VIRTUAL_HOST_IP>
show ip route <REAL_SERVER_1_IP>
show ip route <REAL_SERVER_2_IP>
show ip route <OUTSIDE_CLIENT_IP>
show ip nat statistics
show ip nat translations
show ip nat translations verbose
ping <REAL_SERVER_1_IP>
ping <REAL_SERVER_2_IP>
telnet <VIRTUAL_HOST_IP> <TCP_PORT>
clear ip nat translation *
show ip nat translations
# NAT_Inside_Destination_Rotary_Translation_Rollback
conf t
!
no ip nat inside destination list <ACL_ID> pool <ROTARY_POOL_NAME>
no ip nat pool <ROTARY_POOL_NAME> <REAL_SERVER_START_IP> <REAL_SERVER_END_IP> prefix-length <PREFIX_LENGTH> type rotary
no access-list <ACL_ID>
!
interface <INSIDE_INTERFACE>
 no ip nat inside
!
interface <OUTSIDE_INTERFACE>
 no ip nat outside
!
end
clear ip nat translation *
write memory
# NAT_Inside_Destination_Rotary_Translation_Failure_Checks
| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| No destination translations appear | Traffic is not entering from the outside interface toward the virtual host | `show ip nat statistics` and `show ip route <VIRTUAL_HOST_IP>` | Correct inside/outside interface roles and make outside clients target the virtual host |
| ACL has no matches | ACL matches the wrong address | `show access-lists <ACL_ID>` | Make the ACL permit the virtual host IP, not the real server IPs |
| Traffic reaches only one server | Existing sessions are reused or testing is not creating new TCP sessions | `show ip nat translations verbose` | Clear translations and generate separate new TCP sessions |
| Router rejects the NAT command | Platform or IOS XE image does not support the exact command form | `ip nat inside destination ?` | Use the supported command syntax shown by context help or verify image support |
| Outside client cannot reach the virtual host | Outside routing does not deliver traffic to the NAT router | `show ip route <VIRTUAL_HOST_IP>` on upstream devices | Route the virtual host address toward the NAT router outside interface |
| NAT entry forms but server does not respond | NAT router cannot reach the selected real server | `ping <REAL_SERVER_IP>` from NAT-RTR | Fix inside routing, server gateway, VLAN, or firewall policy |
| Real server receives traffic but replies fail | Server default gateway does not point back through the NAT path | Server gateway check and `show arp` | Set server default gateway toward the NAT router or correct return routing |
| ICMP test is misleading | Rotary destination NAT is intended for new TCP connections in the common server load-sharing model | `show ip nat translations` while testing TCP | Test with the actual TCP service instead of relying only on ping |
| Wrong address is being translated | The virtual host and real server addressing model is reversed | `show running-config | include ip nat pool|access-list|destination` | Put real server addresses in the rotary pool and the virtual host in the ACL |
| Translations remain after config changes | Old NAT state is still active | `show ip nat translations verbose` | Clear translations with `clear ip nat translation *` during maintenance |
##### Source_Basis
# NAT_Inside_Destination_Rotary_Translation_Mental_Model
# NAT_Inside_Destination_Rotary_Translation_Configuration_Checklist
# NAT_Inside_Destination_Rotary_Translation_Skeleton
# NAT_Inside_Destination_Rotary_Translation_Verification_Commands
# NAT_Inside_Destination_Rotary_Translation_Rollback
# NAT_Inside_Destination_Rotary_Translation_Failure_Checks

NTPv4_Time_Synchronization.md
# NTPv4_Time_Synchronization

# NTPv4_Time_Synchronization_Mental_Model

| Concept | Operational Meaning |
|---|---|
| NTP | Network Time Protocol synchronizes device clocks so logs, certificates, routing events, and audits have reliable timestamps |
| NTPv4 | Current NTP version that supports IPv4 and IPv6 time synchronization |
| UTC | NTP synchronizes to Coordinated Universal Time; local timezone display is separate from the actual clock sync |
| Stratum | Distance from an authoritative time source; lower stratum is closer to the reference clock |
| NTP client | Device that synchronizes its clock from an NTP server |
| NTP server | Device that provides time to other clients |
| NTP peer | Device that forms a symmetric association with another NTP device |
| Preferred server | `prefer` marks the server the device should favor when multiple valid sources exist |
| NTP source interface | Stable source address used for NTP packets, usually a loopback or management interface |
| Authentication | NTP authentication validates time sources using configured keys |
| Trusted key | Key ID that the local device is willing to trust for NTP authentication |
| NTP ACL | Restricts which devices can query, serve, peer, or synchronize with the local device |
| Calendar update | Hardware calendar can be updated from the synchronized software clock if supported |
| Clock selection | Device may learn multiple servers but synchronize to one selected source |
| Verification rule | `show ntp status` proves sync state; `show ntp associations` proves server relationship and selection |

# NTPv4_Time_Synchronization_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm management reachability to the intended NTP server | IOS XE Device | `ping <NTP_SERVER_IP> source <SOURCE_INTERFACE_OR_IP>` | NTP server is reachable from the intended source |
| 2 | Confirm the route to the NTP server | IOS XE Device | `show ip route <NTP_SERVER_IP>` | Routing table has a valid path toward the NTP server |
| 3 | Confirm the device clock before configuration | IOS XE Device | `show clock detail` | Current software clock, timezone, and clock source are visible |
| 4 | Confirm existing NTP configuration | IOS XE Device | `show running-config | include ^ntp` | Existing NTP servers, peers, keys, source, ACLs, or master settings are known |
| 5 | Configure timezone for readable local timestamps | IOS XE Device | `conf t`<br>`clock timezone <ZONE_NAME> <UTC_OFFSET>`<br>`end` | Device displays local time using the configured timezone |
| 6 | Configure daylight saving time if required by the site | IOS XE Device | `conf t`<br>`clock summer-time <ZONE_NAME> recurring`<br>`end` | Device adjusts displayed local time during daylight saving time |
| 7 | Configure a stable NTP source interface | IOS XE Device | `conf t`<br>`ntp source <SOURCE_INTERFACE>`<br>`end` | NTP packets use the selected source interface address |
| 8 | Configure the primary NTPv4 server | IOS XE Device | `conf t`<br>`ntp server <PRIMARY_NTP_SERVER_IP> version 4 prefer`<br>`end` | Device forms an NTP client association with the preferred NTPv4 server |
| 9 | Configure the secondary NTPv4 server | IOS XE Device | `conf t`<br>`ntp server <SECONDARY_NTP_SERVER_IP> version 4`<br>`end` | Device has a backup NTP server association |
| 10 | Optional: configure an IPv6 NTPv4 server | IOS XE Device | `conf t`<br>`ntp server <NTP_SERVER_IPV6> version 4`<br>`end` | Device can synchronize through IPv6 if routing and platform support it |
| 11 | Optional: configure an NTP server inside a VRF | IOS XE Device | `conf t`<br>`ntp server vrf <VRF_NAME> <NTP_SERVER_IP> version 4`<br>`end` | Device sends NTP traffic through the specified VRF |
| 12 | Optional: configure an NTP peer | IOS XE Device | `conf t`<br>`ntp peer <PEER_IP> version 4`<br>`end` | Device forms a peer association with another NTP device |
| 13 | Enable NTP authentication if trusted time sources are required | IOS XE Device | `conf t`<br>`ntp authenticate`<br>`end` | NTP authentication is enabled globally |
| 14 | Define an NTP authentication key with HMAC SHA-256 where supported | IOS XE Device | `conf t`<br>`ntp authentication-key <KEY_ID> hmac-sha2-256 <KEY_VALUE>`<br>`end` | Authentication key exists for secure NTP associations |
| 15 | Define an NTP authentication key with CMAC AES-128 where supported | IOS XE Device | `conf t`<br>`ntp authentication-key <KEY_ID> cmac-aes-128 <KEY_VALUE>`<br>`end` | AES CMAC key exists if supported by image and design |
| 16 | Define an NTP authentication key with MD5 only for legacy compatibility | IOS XE Device | `conf t`<br>`ntp authentication-key <KEY_ID> md5 <KEY_VALUE>`<br>`end` | MD5 key exists for older NTP peers where required |
| 17 | Mark the authentication key as trusted | IOS XE Device | `conf t`<br>`ntp trusted-key <KEY_ID>`<br>`end` | Local device trusts the configured key ID |
| 18 | Configure authenticated primary NTP server | IOS XE Device | `conf t`<br>`ntp server <PRIMARY_NTP_SERVER_IP> version 4 key <KEY_ID> prefer`<br>`end` | Device uses authenticated NTP with the preferred server |
| 19 | Configure authenticated secondary NTP server | IOS XE Device | `conf t`<br>`ntp server <SECONDARY_NTP_SERVER_IP> version 4 key <KEY_ID>`<br>`end` | Device uses authenticated NTP with the secondary server |
| 20 | Create ACL for approved NTP sources or clients | IOS XE Device | `conf t`<br>`ip access-list standard ACL-NTP-SOURCES`<br>` permit <APPROVED_NTP_SERVER_IP>`<br>` permit <SECONDARY_NTP_SERVER_IP>`<br>` deny any log`<br>`end` | ACL permits only approved NTP sources |
| 21 | Restrict which devices can serve time to this device | IOS XE Device | `conf t`<br>`ntp access-group peer ACL-NTP-SOURCES`<br>`end` | Only approved NTP devices can form full NTP associations |
| 22 | Optional: restrict NTP queries | IOS XE Device | `conf t`<br>`ntp access-group query-only ACL-NTP-QUERY`<br>`end` | Only approved query sources can query local NTP status |
| 23 | Optional: allow selected clients to sync from this device | IOS XE Device | `conf t`<br>`ntp access-group serve-only ACL-NTP-CLIENTS`<br>`end` | Local device serves time only to approved clients |
| 24 | Optional: configure broadcast NTP on a LAN server interface | IOS XE Device | `conf t`<br>`interface <LAN_INTERFACE>`<br>` ntp broadcast version 4 key <KEY_ID>`<br>`end` | Interface sends NTPv4 broadcast packets using the configured key |
| 25 | Optional: configure broadcast NTP client on LAN client interface | IOS XE Device | `conf t`<br>`interface <LAN_INTERFACE>`<br>` ntp broadcast client`<br>`end` | Interface listens for NTP broadcast packets |
| 26 | Optional: configure local device as NTP master only in isolated lab or fallback design | IOS XE Device | `conf t`<br>`ntp master <STRATUM>`<br>`end` | Device can provide time as a local master when no better source exists |
| 27 | Optional: configure orphan mode for isolated NTP subnet behavior | IOS XE Device | `conf t`<br>`ntp orphan <STRATUM>`<br>`end` | NTP subnet can converge on a common time during upstream isolation |
| 28 | Optional: update hardware calendar from NTP-synchronized software clock | IOS XE Device | `conf t`<br>`ntp update-calendar`<br>`end` | Hardware calendar is updated from the NTP-synchronized clock if supported |
| 29 | Verify NTP configuration | IOS XE Device | `show running-config | include ^ntp` | NTP source, servers, authentication, keys, and access groups are visible |
| 30 | Verify NTP associations | IOS XE Device | `show ntp associations` | NTP servers appear with reachability and selection indicators |
| 31 | Verify detailed NTP association state | IOS XE Device | `show ntp associations detail` | Server reachability, delay, offset, dispersion, and authentication state are visible |
| 32 | Verify NTP synchronization status | IOS XE Device | `show ntp status` | Device shows synchronized state, stratum, reference, and clock precision |
| 33 | Verify local clock after synchronization | IOS XE Device | `show clock detail` | Clock source shows synchronized or NTP-derived time |
| 34 | Verify NTP traffic is allowed through ACLs or firewalls | IOS XE Device | `show access-lists ACL-NTP-SOURCES` | ACL counters increment for approved NTP traffic |
| 35 | Verify UDP 123 is not blocked on the path | IOS XE Device | `show access-lists` | No infrastructure ACL blocks NTP between client and server |
| 36 | Optional: debug NTP packet or event behavior in a lab | IOS XE Device | `debug ntp packets`<br>`debug ntp events` | NTP packet exchange and selection events are visible |
| 37 | Disable debugging after testing | IOS XE Device | `undebug all` | Debugging is disabled |
| 38 | Save the working configuration | IOS XE Device | `copy running-config startup-config` | NTP configuration survives reload |

# NTPv4_Time_Synchronization_Skeleton

conf t
!
clock timezone <ZONE_NAME> <UTC_OFFSET>
clock summer-time <ZONE_NAME> recurring
!
ntp source <SOURCE_INTERFACE>
ntp server <PRIMARY_NTP_SERVER_IP> version 4 prefer
ntp server <SECONDARY_NTP_SERVER_IP> version 4
!
end
write memory

# NTPv4_Authenticated_Client_Skeleton

conf t
!
ntp source <SOURCE_INTERFACE>
ntp authenticate
ntp authentication-key <KEY_ID> hmac-sha2-256 <KEY_VALUE>
ntp trusted-key <KEY_ID>
ntp server <PRIMARY_NTP_SERVER_IP> version 4 key <KEY_ID> prefer
ntp server <SECONDARY_NTP_SERVER_IP> version 4 key <KEY_ID>
!
end
write memory

# NTPv4_Legacy_MD5_Authentication_Skeleton

conf t
!
ntp authenticate
ntp authentication-key <KEY_ID> md5 <KEY_VALUE>
ntp trusted-key <KEY_ID>
ntp server <NTP_SERVER_IP> version 4 key <KEY_ID> prefer
!
end
write memory

# NTPv4_Access_Control_Skeleton

conf t
!
ip access-list standard ACL-NTP-SOURCES
 permit <PRIMARY_NTP_SERVER_IP>
 permit <SECONDARY_NTP_SERVER_IP>
 deny any log
!
ip access-list standard ACL-NTP-CLIENTS
 permit <APPROVED_CLIENT_SUBNET> <WILDCARD_MASK>
 deny any log
!
ntp access-group peer ACL-NTP-SOURCES
ntp access-group serve-only ACL-NTP-CLIENTS
!
end
write memory

# NTPv4_VRF_Client_Skeleton

conf t
!
ntp source <SOURCE_INTERFACE>
ntp server vrf <VRF_NAME> <PRIMARY_NTP_SERVER_IP> version 4 prefer
ntp server vrf <VRF_NAME> <SECONDARY_NTP_SERVER_IP> version 4
!
end
write memory

# NTPv4_Peer_Skeleton

conf t
!
ntp source <SOURCE_INTERFACE>
ntp peer <PEER_IP> version 4
!
end
write memory

# NTPv4_Broadcast_Server_Skeleton

conf t
!
ntp authenticate
ntp authentication-key <KEY_ID> hmac-sha2-256 <KEY_VALUE>
ntp trusted-key <KEY_ID>
!
interface <LAN_INTERFACE>
 ntp broadcast version 4 key <KEY_ID>
!
end
write memory

# NTPv4_Broadcast_Client_Skeleton

conf t
!
interface <LAN_INTERFACE>
 ntp broadcast client
!
end
write memory

# NTPv4_Isolated_Lab_Master_Skeleton

conf t
!
ntp master <STRATUM>
!
end
write memory

# NTPv4_Orphan_Mode_Skeleton

conf t
!
ntp server <NTP_SERVER_IP>
ntp peer <NTP_PEER_IP>
ntp orphan <STRATUM>
!
end
write memory

# NTPv4_Time_Synchronization_Verification_Commands

show clock detail
show running-config | include ^ntp
show running-config | section ip access-list
show ntp status
show ntp associations
show ntp associations detail
show ntp information
show ntp packets
show ip route <NTP_SERVER_IP>
show ipv6 route <NTP_SERVER_IPV6>
show ip interface brief
show ipv6 interface brief
show access-lists
show access-lists ACL-NTP-SOURCES
show access-lists ACL-NTP-CLIENTS
ping <NTP_SERVER_IP> source <SOURCE_INTERFACE_OR_IP>
traceroute <NTP_SERVER_IP> source <SOURCE_IP>
debug ntp events
debug ntp packets
debug ntp validity
show debugging
undebug all

# NTPv4_Time_Synchronization_Rollback

conf t
!
no ntp server <PRIMARY_NTP_SERVER_IP>
no ntp server <SECONDARY_NTP_SERVER_IP>
no ntp server <PRIMARY_NTP_SERVER_IP> version 4 prefer
no ntp server <SECONDARY_NTP_SERVER_IP> version 4
no ntp peer <PEER_IP>
no ntp source <SOURCE_INTERFACE>
!
no ntp access-group peer ACL-NTP-SOURCES
no ntp access-group serve-only ACL-NTP-CLIENTS
no ntp access-group query-only ACL-NTP-QUERY
!
no ntp trusted-key <KEY_ID>
no ntp authentication-key <KEY_ID>
no ntp authenticate
!
no ntp master <STRATUM>
no ntp orphan <STRATUM>
no ntp update-calendar
!
no ip access-list standard ACL-NTP-SOURCES
no ip access-list standard ACL-NTP-CLIENTS
no ip access-list standard ACL-NTP-QUERY
!
end
write memory

# NTPv4_Broadcast_Rollback

conf t
!
interface <LAN_INTERFACE>
 no ntp broadcast
 no ntp broadcast client
!
end
write memory

# NTPv4_VRF_Rollback

conf t
!
no ntp server vrf <VRF_NAME> <PRIMARY_NTP_SERVER_IP>
no ntp server vrf <VRF_NAME> <SECONDARY_NTP_SERVER_IP>
!
end
write memory

# NTPv4_Time_Synchronization_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| `show ntp status` shows unsynchronized | No valid NTP source selected | `show ntp associations` | Fix server reachability, authentication, ACLs, or source interface |
| NTP association is stuck unreachable | Routing or ACL path to server is broken | `ping <NTP_SERVER_IP> source <SOURCE_INTERFACE_OR_IP>` | Fix route, ACL, firewall, or source interface |
| Server appears but is not selected | Server stratum, reach, offset, or dispersion is poor | `show ntp associations detail` | Use a better server or fix path stability |
| Authentication fails | Key ID, algorithm, or key value mismatch | `show running-config | include ^ntp` | Match `ntp authentication-key`, `ntp trusted-key`, and server `key` ID |
| NTP server configured with key but not trusted | Missing `ntp trusted-key <KEY_ID>` | `show running-config | include ntp trusted-key` | Configure `ntp trusted-key <KEY_ID>` |
| NTP authentication enabled but server lacks key | Authenticated client requires matching server key | `show ntp associations detail` | Configure matching authenticated server or remove authentication requirement |
| Clock jumps or is far off initially | Local clock was badly wrong before sync | `show clock detail` | Set approximate clock manually, then let NTP discipline it |
| Timestamps show wrong local time but NTP is synced | Timezone or daylight saving setting is wrong | `show clock detail` | Correct `clock timezone` and `clock summer-time` |
| Device can ping server but NTP does not work | UDP 123 blocked or NTP ACL blocks association | `show access-lists` | Permit NTP UDP 123 and correct NTP access-group ACL |
| NTP source address is unexpected | `ntp source` missing or wrong | `show running-config | include ntp source` | Configure `ntp source <SOURCE_INTERFACE>` |
| Server has no return path to client source | NTP packets leave with unreachable source IP | Server side route check or local ping sourced from NTP source | Fix return routing or change NTP source |
| VRF NTP does not synchronize | Server configured outside the correct VRF | `show running-config | include ntp server vrf` | Use `ntp server vrf <VRF_NAME> <SERVER_IP>` |
| IPv6 NTP fails | IPv6 route or ACL is missing | `show ipv6 route <NTP_SERVER_IPV6>` | Fix IPv6 routing and filtering |
| Preferred server is ignored | Preferred server is invalid or worse than alternate sources | `show ntp associations detail` | Fix preferred server health or remove `prefer` |
| NTP ACL blocks legitimate server | ACL does not permit NTP server IP | `show access-lists ACL-NTP-SOURCES` | Add the server IP to the ACL |
| Device serves time to unauthorized clients | NTP serving not restricted | `show running-config | include ntp access-group` | Configure `ntp access-group serve-only <ACL>` |
| Unauthorized devices can query NTP | Query access not restricted | `show running-config | include ntp access-group` | Configure `ntp access-group query-only <ACL>` |
| Broadcast NTP does not work | Client interface is not configured for broadcast client or server is on wrong VLAN | `show running-config interface <LAN_INTERFACE>` | Configure `ntp broadcast` on server interface and `ntp broadcast client` on clients |
| Broadcast NTP with authentication fails | Broadcast key mismatch or untrusted key | `show running-config | include ^ntp` | Match key ID, algorithm, key value, and trusted key |
| NTP peer does not form | Peer address, routing, authentication, or ACL mismatch | `show ntp associations detail` | Fix bidirectional peer reachability and matching auth |
| Local device becomes bad time source | `ntp master` used without real upstream sync | `show ntp status` | Avoid `ntp master` outside isolated lab or controlled fallback designs |
| Orphan mode selects unexpected source | Orphan stratum or peer design is inconsistent | `show ntp associations detail` | Use consistent orphan stratum and peer configuration |
| Logs still show inconsistent time after NTP sync | Other devices are not synchronized to the same time source | Compare `show clock detail` across devices | Standardize NTP servers and timezone display across the network |
| Certificates fail despite NTP configured | Time still outside certificate validity window | `show clock detail` | Correct time sync and timezone, then retry certificate workflow |
| Debug output is too noisy | NTP debug left enabled | `show debugging` | Run `undebug all` |
| NTP config disappears after reload | Configuration was not saved | `show startup-config | include ^ntp` | Reconfigure and save with `copy running-config startup-config` |

# Index

NTPv4_Time_Synchronization.md
NTPv4_Time_Synchronization
NTPv4_Time_Synchronization_Mental_Model
NTPv4_Time_Synchronization_Configuration_Checklist
NTPv4_Time_Synchronization_Skeleton
NTPv4_Authenticated_Client_Skeleton
NTPv4_Legacy_MD5_Authentication_Skeleton
NTPv4_Access_Control_Skeleton
NTPv4_VRF_Client_Skeleton
NTPv4_Peer_Skeleton
NTPv4_Broadcast_Server_Skeleton
NTPv4_Broadcast_Client_Skeleton
NTPv4_Isolated_Lab_Master_Skeleton
NTPv4_Orphan_Mode_Skeleton
NTPv4_Time_Synchronization_Verification_Commands
NTPv4_Time_Synchronization_Rollback
NTPv4_Broadcast_Rollback
NTPv4_VRF_Rollback
NTPv4_Time_Synchronization_Failure_Checks
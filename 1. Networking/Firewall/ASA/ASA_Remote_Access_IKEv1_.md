

```
# Index
# Source_Basis
# ASA_Remote_Access_IKEv1_Filter_Mental_Model
# ASA_Remote_Access_IKEv1_Filter_Configuration_Checklist
# ASA_Remote_Access_IKEv1_Filter_VPN_Filter_Skeleton
# ASA_Remote_Access_IKEv1_Filter_DNS_And_Web_Skeleton
# ASA_Remote_Access_IKEv1_Filter_RDP_Only_Skeleton
# ASA_Remote_Access_IKEv1_Filter_Outside_ACL_Inspection_Skeleton
# ASA_Remote_Access_IKEv1_Filter_Verification_Commands
# ASA_Remote_Access_IKEv1_Filter_Rollback
# ASA_Remote_Access_IKEv1_Filter_Failure_Checks
```

# ASA_Remote_Access_IKEv1_Filter_Mental_Model
| Concept                           | Operational Meaning                                                                                                                                                             |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| VPN filter                        | Group-policy ACL that restricts what VPN users can access after the tunnel is established                                                                                       |
| Outside ACL inspection            | Alternative model where decrypted VPN traffic is forced through the outside interface ACL                                                                                       |
| `sysopt connection permit-vpn`    | Default ASA behavior that lets decrypted VPN traffic bypass interface ACL checks                                                                                                |
| `no sysopt connection permit-vpn` | Makes decrypted VPN traffic subject to interface ACL inspection                                                                                                                 |
| Group policy scope                | Best place to apply a filter when different VPN groups need different access                                                                                                    |
| Tunnel group                      | Connection profile that maps users to a default group policy                                                                                                                    |
| VPN pool source                   | The source in the filter ACL is usually the VPN client address pool                                                                                                             |
| Inside destination                | The destination in the filter ACL is usually the internal host, subnet, or service being allowed                                                                                |
| Filter ACL implicit deny          | Anything not explicitly permitted by the filter ACL is denied                                                                                                                   |
| Split tunnel ACL                  | Controls which routes are encrypted, not what users are allowed to access                                                                                                       |
| VPN filter vs split tunnel        | Split tunnel decides where traffic goes; VPN filter decides whether allowed VPN traffic is permitted                                                                            |
| VPN filter vs outside ACL         | VPN filter is per policy/user group; outside ACL inspection is global behavior after disabling `sysopt`                                                                         |
| Blunt rule                        | Use VPN filters for normal per-group access control. Do not disable `sysopt connection permit-vpn` unless you intentionally want interface ACLs to govern decrypted VPN traffic |
# ASA_Remote_Access_IKEv1_Filter_Configuration_Checklist
| Step | Task                                                                     | Device      | Command                                                                                                                          | Expected Result                                                                               |
| ---: | ------------------------------------------------------------------------ | ----------- | -------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
|    1 | Confirm ASA interface status                                             | ASA         | `show interface ip brief`                                                                                                        | Inside and outside interfaces are up/up                                                       |
|    2 | Confirm logical interface names                                          | ASA         | `show nameif`                                                                                                                    | Interfaces are named correctly, usually `inside` and `outside`                                |
|    3 | Confirm IKEv1 remote-access tunnel exists                                | ASA         | `show running-config crypto ikev1`                                                                                               | IKEv1 is enabled on outside and policy exists                                                 |
|    4 | Confirm remote-access tunnel group exists                                | ASA         | `show running-config tunnel-group <tunnel-group-name>`                                                                           | Tunnel group is type `remote-access`                                                          |
|    5 | Confirm tunnel group default group policy                                | ASA         | `show running-config tunnel-group <tunnel-group-name>`                                                                           | Tunnel group references the intended group policy                                             |
|    6 | Confirm group policy exists                                              | ASA         | `show running-config group-policy <group-policy-name>`                                                                           | Group policy exists and permits `vpn-tunnel-protocol ikev1`                                   |
|    7 | Confirm VPN address pool                                                 | ASA         | `show running-config ip local pool`                                                                                              | VPN pool exists and does not overlap inside LAN                                               |
|    8 | Confirm group policy address pool binding                                | ASA         | `show running-config group-policy <group-policy-name>`                                                                           | Group policy references the intended VPN pool                                                 |
|    9 | Confirm VPN pool network                                                 | ASA / Notes | `<vpn-pool-network> <vpn-pool-mask>`                                                                                             | VPN client source network is known                                                            |
|   10 | Confirm allowed inside destination                                       | ASA / Notes | `<inside-host-or-network> <inside-mask>`                                                                                         | Destination to be allowed is known                                                            |
|   11 | Confirm allowed service                                                  | ASA / Notes | `<protocol>/<port>`                                                                                                              | Allowed service is known, such as TCP/3389, TCP/443, TCP/22, or ICMP                          |
|   12 | Confirm whether split tunneling already includes destination             | ASA         | `show running-config group-policy <group-policy-name>`                                                                           | Split tunnel list includes the destination if split tunnel is used                            |
|   13 | Confirm current `sysopt` behavior                                        | ASA         | `show run all sysopt`                                                                                                            | Usually shows `sysopt connection permit-vpn` enabled                                          |
|   14 | Review existing VPN filter ACLs                                          | ASA         | `show running-config access-list`                                                                                                | Existing VPN filter ACL names and entries are known                                           |
|   15 | Review existing group-policy filters                                     | ASA         | `show running-config group-policy`                                                                                               | Existing `vpn-filter value` assignments are known                                             |
|   16 | Review outside ACL only if using interface ACL inspection                | ASA         | `show running-config access-group`                                                                                               | Outside ACL name and interface binding are known                                              |
|   17 | Enter configuration mode                                                 | ASA         | `configure terminal`                                                                                                             | ASA enters global configuration mode                                                          |
|   18 | Create VPN filter ACL remark                                             | ASA         | `access-list <vpn-filter-acl> remark Filter for <group-policy-name> IKEv1 remote-access users`                                   | Filter purpose is documented                                                                  |
|   19 | Permit VPN pool to specific inside TCP host/service                      | ASA         | `access-list <vpn-filter-acl> extended permit tcp <vpn-pool-network> <vpn-pool-mask> host <inside-host-ip> eq <port>`            | VPN users are allowed to reach one inside TCP service                                         |
|   20 | Permit VPN pool to specific inside subnet/service                        | ASA         | `access-list <vpn-filter-acl> extended permit tcp <vpn-pool-network> <vpn-pool-mask> <inside-network> <inside-mask> eq <port>`   | VPN users are allowed to reach that service across the inside subnet                          |
|   21 | Permit ICMP only if ping should work                                     | ASA         | `access-list <vpn-filter-acl> extended permit icmp <vpn-pool-network> <vpn-pool-mask> <inside-network> <inside-mask>`            | VPN users can ping allowed inside destinations                                                |
|   22 | Permit DNS to internal DNS if users need internal name resolution        | ASA         | `access-list <vpn-filter-acl> extended permit udp <vpn-pool-network> <vpn-pool-mask> host <dns-server-ip> eq 53`                 | VPN users can query internal DNS over UDP                                                     |
|   23 | Permit TCP DNS if needed                                                 | ASA         | `access-list <vpn-filter-acl> extended permit tcp <vpn-pool-network> <vpn-pool-mask> host <dns-server-ip> eq 53`                 | VPN users can use TCP DNS when needed                                                         |
|   24 | Permit required management service only if intended                      | ASA         | `access-list <vpn-filter-acl> extended permit tcp <vpn-pool-network> <vpn-pool-mask> host <inside-host-ip> eq <management-port>` | Management access is explicitly limited                                                       |
|   25 | Add explicit deny log if useful for troubleshooting                      | ASA         | `access-list <vpn-filter-acl> extended deny ip any any log`                                                                      | Denied VPN traffic is visible in ACL hits/logging                                             |
|   26 | Enter group policy attributes                                            | ASA         | `group-policy <group-policy-name> attributes`                                                                                    | ASA enters group-policy attribute mode                                                        |
|   27 | Apply VPN filter to group policy                                         | ASA         | `vpn-filter value <vpn-filter-acl>`                                                                                              | Users mapped to the group policy are filtered by the ACL                                      |
|   28 | Confirm split tunnel remains route-only control                          | ASA         | `show running-config group-policy <group-policy-name>`                                                                           | Split tunnel and VPN filter are both visible if configured                                    |
|   29 | Do not disable `sysopt` for normal per-group filtering                   | ASA / Notes | Keep `sysopt connection permit-vpn`                                                                                              | Interface ACL bypass remains enabled while VPN filter enforces per-group access               |
|   30 | Disable VPN ACL bypass only if using outside ACL inspection              | ASA         | `no sysopt connection permit-vpn`                                                                                                | Decrypted VPN traffic is forced through interface ACLs                                        |
|   31 | Create outside ACL permit if `sysopt` is disabled                        | ASA         | `access-list <outside-acl> extended permit tcp <vpn-pool-network> <vpn-pool-mask> host <inside-host-ip> eq <port>`               | Outside ACL permits decrypted VPN traffic to the inside service                               |
|   32 | Add outside ACL ICMP permit if `sysopt` is disabled and ping is required | ASA         | `access-list <outside-acl> extended permit icmp <vpn-pool-network> <vpn-pool-mask> <inside-network> <inside-mask>`               | Outside ACL permits decrypted ICMP from VPN users                                             |
|   33 | Apply outside ACL if not already applied                                 | ASA         | `access-group <outside-acl> in interface outside`                                                                                | Outside ACL is active on the outside interface                                                |
|   34 | Save configuration                                                       | ASA         | `write memory`                                                                                                                   | Filter configuration is saved                                                                 |
|   35 | Reconnect test VPN client                                                | Client      | `<disconnect and reconnect VPN>`                                                                                                 | Client receives group policy with filter applied                                              |
|   36 | Verify session group policy                                              | ASA         | `show vpn-sessiondb remote`                                                                                                      | User session shows expected group policy and tunnel group                                     |
|   37 | Verify VPN filter attachment                                             | ASA         | `show running-config group-policy <group-policy-name>`                                                                           | Output includes `vpn-filter value <vpn-filter-acl>`                                           |
|   38 | Test allowed service                                                     | VPN Client  | `Test-NetConnection <inside-host-ip> -Port <port>` or service-specific client test                                               | Allowed service succeeds                                                                      |
|   39 | Test denied service                                                      | VPN Client  | `Test-NetConnection <inside-host-ip> -Port <denied-port>`                                                                        | Denied service fails                                                                          |
|   40 | Verify VPN filter hits                                                   | ASA         | `show access-list <vpn-filter-acl>`                                                                                              | Permit and deny hit counts increment                                                          |
|   41 | Verify outside ACL hits if `sysopt` was disabled                         | ASA         | `show access-list <outside-acl>`                                                                                                 | Outside ACL hit counts increment                                                              |
|   42 | Verify IKEv1 session                                                     | ASA         | `show crypto ikev1 sa detail`                                                                                                    | Remote-access client shows `AM_ACTIVE`                                                        |
|   43 | Verify IPsec counters                                                    | ASA         | `show crypto ipsec sa`                                                                                                           | Encrypt/decrypt counters increment                                                            |
|   44 | Check drops if access fails                                              | ASA         | `show asp drop`                                                                                                                  | Drop reason points to VPN filter, outside ACL, NAT, route, inspection, or host firewall issue |
|   45 | Debug only if needed                                                     | ASA         | `debug crypto ikev1 127` and `debug crypto ipsec 127`                                                                            | Debug confirms tunnel/auth/IPsec behavior if the issue is not purely filtering                |
|   46 | Stop debugs                                                              | ASA         | `undebug all`                                                                                                                    | Debug output stops                                                                            |
```
# ASA_Remote_Access_IKEv1_Filter_VPN_Filter_Skeleton
configure terminal

access-list <vpn-filter-acl> remark Filter for <group-policy-name> IKEv1 remote-access users
access-list <vpn-filter-acl> extended permit tcp <vpn-pool-network> <vpn-pool-mask> host <inside-host-ip> eq <port>
access-list <vpn-filter-acl> extended permit icmp <vpn-pool-network> <vpn-pool-mask> <inside-network> <inside-mask>
access-list <vpn-filter-acl> extended deny ip any any log

group-policy <group-policy-name> attributes
 vpn-filter value <vpn-filter-acl>

write memory
```

```
# ASA_Remote_Access_IKEv1_Filter_DNS_And_Web_Skeleton
configure terminal

access-list <vpn-filter-acl> remark Allow VPN users to internal DNS and HTTPS app only
access-list <vpn-filter-acl> extended permit udp <vpn-pool-network> <vpn-pool-mask> host <dns-server-ip> eq 53
access-list <vpn-filter-acl> extended permit tcp <vpn-pool-network> <vpn-pool-mask> host <dns-server-ip> eq 53
access-list <vpn-filter-acl> extended permit tcp <vpn-pool-network> <vpn-pool-mask> host <inside-web-server-ip> eq 443
access-list <vpn-filter-acl> extended deny ip any any log

group-policy <group-policy-name> attributes
 vpn-filter value <vpn-filter-acl>

write memory
```

```
# ASA_Remote_Access_IKEv1_Filter_RDP_Only_Skeleton
configure terminal

access-list <vpn-filter-acl> remark Allow VPN users to RDP jump host only
access-list <vpn-filter-acl> extended permit tcp <vpn-pool-network> <vpn-pool-mask> host <rdp-jump-host-ip> eq 3389
access-list <vpn-filter-acl> extended deny ip any any log

group-policy <group-policy-name> attributes
 vpn-filter value <vpn-filter-acl>

write memory
```

`
```
# ASA_Remote_Access_IKEv1_Filter_Outside_ACL_Inspection_Skeleton
configure terminal

no sysopt connection permit-vpn

access-list <outside-acl> remark Permit decrypted VPN users to approved inside service
access-list <outside-acl> extended permit tcp <vpn-pool-network> <vpn-pool-mask> host <inside-host-ip> eq <port>
access-list <outside-acl> extended deny ip any any log

access-group <outside-acl> in interface outside

write memory
```

# ASA_Remote_Access_IKEv1_Filter_Verification_Commands
| Task                                              | Command                                                   | Expected Result                                                                                           |
| ------------------------------------------------- | --------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| Verify remote-access session                      | `show vpn-sessiondb remote`                               | User session appears with expected tunnel group, group policy, assigned IP, and public IP                 |
| Verify detailed session                           | `show vpn-sessiondb detail`                               | Active `IPSec Remote Access` session appears                                                              |
| Verify IKEv1 Phase 1                              | `show crypto ikev1 sa detail`                             | User session shows `AM_ACTIVE`                                                                            |
| Verify IPsec Phase 2                              | `show crypto ipsec sa`                                    | Dynamic peer, assigned client IP, and encrypt/decrypt counters appear                                     |
| Verify group policy filter assignment             | `show running-config group-policy <group-policy-name>`    | Output includes `vpn-filter value <vpn-filter-acl>`                                                       |
| Verify VPN filter ACL                             | `show access-list <vpn-filter-acl>`                       | Permit and deny entries exist in expected order                                                           |
| Verify VPN filter permit hits                     | `show access-list <vpn-filter-acl>`                       | Permit ACE hit count increments when allowed traffic is tested                                            |
| Verify VPN filter deny hits                       | `show access-list <vpn-filter-acl>`                       | Explicit deny/log ACE increments when blocked traffic is tested                                           |
| Verify `sysopt` state                             | `show run all sysopt`                                     | `sysopt connection permit-vpn` remains enabled for normal VPN-filter design                               |
| Verify outside ACL if `sysopt` disabled           | `show access-list <outside-acl>`                          | Outside ACL has permit entries for decrypted VPN traffic                                                  |
| Verify outside ACL binding                        | `show running-config access-group`                        | Outside ACL is applied inbound on outside if using interface ACL inspection                               |
| Verify split tunnel routes                        | `show running-config group-policy <group-policy-name>`    | Split tunnel list includes destinations users must reach                                                  |
| Verify NAT exemption                              | `show nat detail`                                         | Inside-to-VPN-pool identity NAT is present and hit count increments                                       |
| Test allowed TCP service                          | `Test-NetConnection <inside-host-ip> -Port <port>`        | Connection succeeds                                                                                       |
| Test denied TCP service                           | `Test-NetConnection <inside-host-ip> -Port <denied-port>` | Connection fails                                                                                          |
| Test allowed ICMP                                 | `ping <inside-host-ip>`                                   | Ping succeeds only if ICMP is permitted                                                                   |
| Verify ASA connection table                       | `show conn address <vpn-client-assigned-ip>`              | Client flows appear for allowed traffic                                                                   |
| Verify translations                               | `show xlate`                                              | VPN client traffic is not incorrectly PATed                                                               |
| Verify drops                                      | `show asp drop`                                           | No relevant VPN filter, ACL, NAT, route, inspection, or host firewall drops increment for allowed traffic |
| Debug IKEv1 only if tunnel setup fails            | `debug crypto ikev1 127`                                  | Shows tunnel-group, IKE, XAUTH, and group policy behavior                                                 |
| Debug IPsec only if tunnel setup or counters fail | `debug crypto ipsec 127`                                  | Shows IPsec transform and data-plane negotiation behavior                                                 |
| Stop debugs                                       | `undebug all`                                             | Debug output stops                                                                                        |
# ASA_Remote_Access_IKEv1_Filter_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify active VPN filter | ASA | `show running-config group-policy <group-policy-name>` | Current `vpn-filter value <vpn-filter-acl>` is identified |
| 2 | Identify VPN filter ACL entries | ASA | `show access-list <vpn-filter-acl>` | ACL entries and hit counts are visible |
| 3 | Identify outside ACL inspection state | ASA | `show run all sysopt` | Confirms whether `sysopt connection permit-vpn` is enabled or disabled |
| 4 | Identify outside ACL binding | ASA | `show running-config access-group` | Outside ACL binding is visible if configured |
| 5 | Enter configuration mode | ASA | `configure terminal` | ASA enters global configuration mode |
| 6 | Remove VPN filter from group policy | ASA | `group-policy <group-policy-name> attributes` then `no vpn-filter value <vpn-filter-acl>` | Group policy no longer applies the VPN filter |
| 7 | Remove VPN filter ACL if unused | ASA | `clear configure access-list <vpn-filter-acl>` | VPN filter ACL is removed |
| 8 | Remove outside ACL entry if used only for VPN filter test | ASA | `no access-list <outside-acl> extended permit tcp <vpn-pool-network> <vpn-pool-mask> host <inside-host-ip> eq <port>` | VPN-specific outside ACL permit is removed |
| 9 | Remove outside ACL explicit deny if lab-only | ASA | `no access-list <outside-acl> extended deny ip any any log` | Lab deny/log entry is removed |
| 10 | Restore default VPN ACL bypass if disabled | ASA | `sysopt connection permit-vpn` | Decrypted VPN traffic bypasses interface ACLs again |
| 11 | Remove outside access-group only if ACL was created only for this lab | ASA | `no access-group <outside-acl> in interface outside` | Outside ACL is no longer applied |
| 12 | Clear VPN session for testing | ASA | `vpn-sessiondb logoff name <username>` | User is disconnected and must reconnect with updated policy |
| 13 | Clear stale IPsec SAs | ASA | `clear crypto ipsec sa` | Old IPsec SAs are cleared |
| 14 | Clear stale connections | ASA | `clear conn address <vpn-client-assigned-ip>` | Old user connection state is cleared |
| 15 | Verify filter removal | ASA | `show running-config group-policy <group-policy-name>` | `vpn-filter value` no longer appears |
| 16 | Verify `sysopt` restored | ASA | `show run all sysopt` | `sysopt connection permit-vpn` is enabled unless intentionally left disabled |
| 17 | Save rollback state | ASA | `write memory` | Rollback is saved |

# ASA_Remote_Access_IKEv1_Filter_Failure_Checks
| Symptom                                           | Command                                                                                | What Usually Broke                                                                                     |
| ------------------------------------------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| VPN connects but all inside access fails          | `show access-list <vpn-filter-acl>`                                                    | VPN filter ACL has no permit, wrong source pool, wrong destination, or implicit deny blocks everything |
| Allowed service still fails                       | `show access-list <vpn-filter-acl>`                                                    | Permit ACE does not match protocol, source, destination, or port                                       |
| Filter ACL has zero hits                          | `show vpn-sessiondb remote` and `show running-config group-policy <group-policy-name>` | User is not landing on the group policy that has the filter                                            |
| Denied traffic still works                        | `show running-config group-policy <group-policy-name>`                                 | VPN filter is not applied, wrong group policy is used, or user-specific policy overrides it            |
| Outside ACL blocks VPN traffic                    | `show run all sysopt` and `show access-list <outside-acl>`                             | `no sysopt connection permit-vpn` is enabled and outside ACL lacks the needed permit                   |
| Outside ACL has zero hits                         | `show run all sysopt`                                                                  | `sysopt connection permit-vpn` is still enabled, so decrypted VPN traffic bypasses outside ACL         |
| VPN filter allows traffic but client has no route | `show running-config group-policy <group-policy-name>`                                 | Split tunnel ACL does not include the destination network                                              |
| VPN filter allows DNS but name resolution fails   | `show running-config group-policy <group-policy-name>`                                 | DNS server or default domain is not pushed to the client                                               |
| ICMP fails but TCP works                          | `show access-list <vpn-filter-acl>`                                                    | ICMP is not explicitly permitted or host firewall blocks ping                                          |
| TCP fails but ping works                          | `show access-list <vpn-filter-acl>`                                                    | TCP service port not permitted, service not listening, or host firewall blocks the port                |
| NAT exemption has zero hits                       | `show nat detail`                                                                      | VPN pool object, inside object, or interface pair is wrong                                             |
| VPN client traffic gets PATed                     | `show xlate` and `show nat detail`                                                     | Missing inside-to-VPN-pool identity NAT or broad PAT matched first                                     |
| IPsec counters do not increment                   | `show crypto ipsec sa`                                                                 | Client route/split tunnel issue, no interesting traffic, or tunnel data plane broken                   |
| VPN session shows wrong group policy              | `show vpn-sessiondb remote`                                                            | User selected wrong tunnel group, AAA assigned different policy, or default group policy is wrong      |
| Explicit deny logs too much traffic               | `show access-list <vpn-filter-acl>`                                                    | Filter is too narrow or client sends background traffic to internal destinations                       |
| Packet reaches ASA but not inside host            | `show conn`, `show asp drop`, inside host firewall check                               | VPN filter permits it, but route, host firewall, inspection, or return path breaks                     |
| Filter changed but behavior did not change        | `show vpn-sessiondb remote`                                                            | User has not reconnected, old session still has prior policy                                           |
| Debugs flood terminal                             | `show debug`                                                                           | Debugs left enabled; run `undebug all`                                                                 |
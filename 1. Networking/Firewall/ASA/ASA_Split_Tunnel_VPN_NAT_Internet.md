
```
# Index
# Source_Basis
# ASA_Split_Tunnel_VPN_NAT_Internet_Mental_Model
# ASA_Split_Tunnel_VPN_NAT_Internet_Configuration_Checklist
# ASA_Split_Tunnel_VPN_NAT_Internet_Split_Tunnel_Local_Internet_Skeleton
# ASA_Split_Tunnel_VPN_NAT_Internet_Inside_Dynamic_PAT_Skeleton
# ASA_Split_Tunnel_VPN_NAT_Internet_Full_Tunnel_ASA_Internet_PAT_Skeleton
# ASA_Split_Tunnel_VPN_NAT_Internet_Outside_ACL_Inspection_Skeleton
# ASA_Split_Tunnel_VPN_NAT_Internet_Verification_Commands
# ASA_Split_Tunnel_VPN_NAT_Internet_Rollback
# ASA_Split_Tunnel_VPN_NAT_Internet_Failure_Checks
```


# ASA_Split_Tunnel_VPN_NAT_Internet_Mental_M

| Concept                      | Operational Meaning                                                                                                                                                                                       |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Split tunnel                 | VPN client sends only listed internal networks through the VPN tunnel                                                                                                                                     |
| Local Internet breakout      | VPN client Internet traffic goes directly out the user's local network, not through the ASA                                                                                                               |
| Full tunnel                  | VPN client sends all traffic, including Internet traffic, through the ASA                                                                                                                                 |
| VPN pool                     | Address range assigned to remote-access VPN clients                                                                                                                                                       |
| Split tunnel ACL             | Standard ACL listing internal destination networks that should be tunneled                                                                                                                                |
| Group policy                 | Where split tunnel policy and split tunnel ACL are applied                                                                                                                                                |
| NAT exemption                | Inside-to-VPN-pool traffic must stay untranslated                                                                                                                                                         |
| VPN client Internet PAT      | Required only when full-tunnel VPN clients use the ASA as their Internet egress                                                                                                                           |
| `(outside,outside)` NAT      | Used when decrypted VPN client traffic enters outside and exits outside toward the Internet                                                                                                               |
| Same-interface forwarding    | Required for full-tunnel Internet hairpin because traffic enters and exits ASA outside                                                                                                                    |
| Inside Dynamic PAT           | Normal inside users need PAT to reach Internet                                                                                                                                                            |
| VPN Dynamic PAT              | Full-tunnel VPN users need PAT to reach Internet through ASA                                                                                                                                              |
| Split tunnel direct Internet | Does not need ASA VPN pool PAT for Internet traffic because the traffic never reaches ASA                                                                                                                 |
| Blunt rule                   | Split tunnel and ASA Internet hairpin are different designs. Split tunnel keeps Internet local. Full tunnel sends Internet through ASA and requires outside-to-outside PAT plus same-interface forwarding |
|                              |                                                                                                                                                                                                           |
# ASA_Split_Tunnel_VPN_NAT_Internet_Configuration_Checklist
| Step | Task                                                                           | Device      | Command                                                                                                                                               | Expected Result                                                                                     |
| ---: | ------------------------------------------------------------------------------ | ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
|    1 | Confirm ASA interface status                                                   | ASA         | `show interface ip brief`                                                                                                                             | Inside and outside interfaces are up/up                                                             |
|    2 | Confirm logical interface names                                                | ASA         | `show nameif`                                                                                                                                         | Interfaces are named correctly, usually `inside` and `outside`                                      |
|    3 | Confirm outside route to Internet                                              | ASA         | `show route`                                                                                                                                          | ASA has default route out the outside interface                                                     |
|    4 | Confirm inside protected networks                                              | ASA / Notes | `<inside-network-list>`                                                                                                                               | Internal networks that VPN users should reach are known                                             |
|    5 | Confirm VPN pool network                                                       | ASA / Notes | `<vpn-pool-network> <vpn-pool-mask>`                                                                                                                  | VPN client assigned subnet is known                                                                 |
|    6 | Confirm remote-access group policy                                             | ASA         | `show running-config group-policy <group-policy-name>`                                                                                                | Correct group policy is identified                                                                  |
|    7 | Confirm remote-access tunnel group                                             | ASA         | `show running-config tunnel-group <tunnel-group-name>`                                                                                                | Tunnel group maps users to expected group policy                                                    |
|    8 | Confirm VPN address pool assignment                                            | ASA         | `show running-config group-policy <group-policy-name>` or `show running-config tunnel-group <tunnel-group-name>`                                      | VPN users receive expected pool                                                                     |
|    9 | Confirm current split tunnel state                                             | ASA         | `show running-config group-policy <group-policy-name>`                                                                                                | Existing split tunnel settings are visible                                                          |
|   10 | Confirm current NAT table                                                      | ASA         | `show nat`                                                                                                                                            | NAT sections and rule order are visible                                                             |
|   11 | Confirm NAT details and hit counts                                             | ASA         | `show nat detail`                                                                                                                                     | Identity NAT and PAT hit counts are visible                                                         |
|   12 | Decide design mode                                                             | ASA / Notes | `split tunnel` or `full tunnel`                                                                                                                       | Internet path is clearly chosen before changing NAT                                                 |
|   13 | Use split tunnel for local Internet breakout                                   | ASA / Notes | `split-tunnel-policy tunnelspecified`                                                                                                                 | Only internal networks are tunneled                                                                 |
|   14 | Use full tunnel only when ASA must inspect/PAT VPN Internet traffic            | ASA / Notes | `no split tunnel` or tunnel-all policy                                                                                                                | VPN client Internet traffic must hairpin through ASA                                                |
|   15 | Enter configuration mode                                                       | ASA         | `configure terminal`                                                                                                                                  | ASA enters global configuration mode                                                                |
|   16 | Create split tunnel ACL remark                                                 | ASA         | `access-list <split-acl> remark Internal networks tunneled by VPN clients`                                                                            | Split tunnel ACL purpose is documented                                                              |
|   17 | Add inside network to split tunnel ACL                                         | ASA         | `access-list <split-acl> standard permit <inside-network-1> <inside-mask-1>`                                                                          | VPN clients tunnel this inside network                                                              |
|   18 | Add additional inside network if needed                                        | ASA         | `access-list <split-acl> standard permit <inside-network-2> <inside-mask-2>`                                                                          | Additional internal network is tunneled                                                             |
|   19 | Enter group policy attributes                                                  | ASA         | `group-policy <group-policy-name> attributes`                                                                                                         | ASA enters group-policy attribute mode                                                              |
|   20 | Enable split tunnel policy                                                     | ASA         | `split-tunnel-policy tunnelspecified`                                                                                                                 | Only listed networks are tunneled                                                                   |
|   21 | Attach split tunnel ACL                                                        | ASA         | `split-tunnel-network-list value <split-acl>`                                                                                                         | Group policy pushes split tunnel route list to VPN clients                                          |
|   22 | Push internal DNS if users need internal name resolution                       | ASA         | `dns-server value <dns1> <dns2>`                                                                                                                      | VPN clients receive internal DNS servers                                                            |
|   23 | Push internal DNS suffix                                                       | ASA         | `default-domain value <domain-name>`                                                                                                                  | VPN clients receive internal DNS suffix                                                             |
|   24 | Create inside network object for NAT exemption                                 | ASA         | `object network <inside-object>`                                                                                                                      | ASA enters inside object mode                                                                       |
|   25 | Define inside protected subnet                                                 | ASA         | `subnet <inside-network> <inside-mask>`                                                                                                               | Inside protected network object is defined                                                          |
|   26 | Create VPN pool object                                                         | ASA         | `object network <vpn-pool-object>`                                                                                                                    | ASA enters VPN pool object mode                                                                     |
|   27 | Define VPN pool subnet                                                         | ASA         | `subnet <vpn-pool-network> <vpn-pool-mask>`                                                                                                           | VPN pool object is defined                                                                          |
|   28 | Configure inside-to-VPN identity NAT                                           | ASA         | `nat (inside,outside) source static <inside-object> <inside-object> destination static <vpn-pool-object> <vpn-pool-object> no-proxy-arp route-lookup` | Inside-to-VPN-pool traffic is not translated                                                        |
|   29 | Confirm inside Dynamic PAT for normal inside Internet users                    | ASA         | `show running-config object network <inside-object>` or `show nat detail`                                                                             | Inside users have Dynamic PAT to outside if required                                                |
|   30 | Configure inside Dynamic PAT if missing                                        | ASA         | `object network <inside-object>` then `nat (inside,outside) dynamic interface`                                                                        | Inside users can PAT to the outside interface                                                       |
|   31 | Stop here for normal split tunnel local Internet design                        | ASA / Notes | `No VPN pool Internet PAT required`                                                                                                                   | VPN users use local Internet and ASA handles only internal destinations                             |
|   32 | Enable same-interface forwarding only for full-tunnel ASA Internet egress      | ASA         | `same-security-traffic permit intra-interface`                                                                                                        | ASA can hairpin VPN Internet traffic out outside                                                    |
|   33 | Configure VPN pool PAT only for full tunnel                                    | ASA         | `object network <vpn-pool-object>` then `nat (outside,outside) dynamic interface`                                                                     | VPN client Internet traffic PATs to ASA outside interface                                           |
|   34 | Confirm no split tunnel if using ASA full-tunnel Internet egress               | ASA         | `show running-config group-policy <group-policy-name>`                                                                                                | Split tunnel is absent or policy tunnels all traffic                                                |
|   35 | Confirm VPN filter allows intended access if configured                        | ASA         | `show access-list <vpn-filter-acl>`                                                                                                                   | VPN filter permits inside access and does not accidentally block required traffic                   |
|   36 | Confirm default VPN ACL bypass behavior                                        | ASA         | `show run all sysopt`                                                                                                                                 | `sysopt connection permit-vpn` is enabled unless intentionally changed                              |
|   37 | Configure outside ACL inspection only if required                              | ASA         | `no sysopt connection permit-vpn`                                                                                                                     | Decrypted VPN traffic must pass outside ACL                                                         |
|   38 | Permit VPN pool to inside if outside ACL inspection is used                    | ASA         | `access-list <outside-acl> extended permit ip <vpn-pool-network> <vpn-pool-mask> <inside-network> <inside-mask>`                                      | Outside ACL permits decrypted VPN traffic                                                           |
|   39 | Permit VPN pool to Internet if outside ACL inspection is used with full tunnel | ASA         | `access-list <outside-acl> extended permit ip <vpn-pool-network> <vpn-pool-mask> any`                                                                 | Outside ACL permits VPN Internet traffic if full tunnel is used                                     |
|   40 | Apply outside ACL if required                                                  | ASA         | `access-group <outside-acl> in interface outside`                                                                                                     | Outside ACL is active                                                                               |
|   41 | Save configuration                                                             | ASA         | `write memory`                                                                                                                                        | Split tunnel and NAT configuration is saved                                                         |
|   42 | Reconnect VPN client                                                           | Client      | `<disconnect and reconnect VPN>`                                                                                                                      | Client receives updated group policy and route list                                                 |
|   43 | Verify assigned VPN address                                                    | Client      | `ipconfig /all` or client statistics                                                                                                                  | Client receives address from VPN pool                                                               |
|   44 | Verify split tunnel routes                                                     | Client      | `route print` or AnyConnect route details                                                                                                             | Only internal networks are routed through VPN for split tunnel design                               |
|   45 | Verify Internet stays local for split tunnel                                   | Client      | `tracert 8.8.8.8`                                                                                                                                     | First hop is local client gateway, not ASA VPN path                                                 |
|   46 | Verify internal route uses VPN                                                 | Client      | `tracert <inside-host-ip>`                                                                                                                            | Path goes into VPN tunnel                                                                           |
|   47 | Test internal reachability                                                     | Client      | `ping <inside-host-ip>` or service test                                                                                                               | VPN client reaches permitted inside resource                                                        |
|   48 | Test Internet reachability                                                     | Client      | `curl https://ifconfig.me` or browser test                                                                                                            | Internet works through local breakout for split tunnel                                              |
|   49 | Verify NAT exemption hit count                                                 | ASA         | `show nat detail`                                                                                                                                     | Inside-to-VPN identity NAT hit count increments                                                     |
|   50 | Verify VPN pool PAT hits only for full tunnel                                  | ASA         | `show nat detail`                                                                                                                                     | `(outside,outside)` VPN pool PAT hit count increments only if full tunnel is used                   |
|   51 | Verify xlate table                                                             | ASA         | `show xlate`                                                                                                                                          | Split tunnel shows no Internet PAT for VPN pool; full tunnel shows VPN pool PAT to outside          |
|   52 | Verify connection table                                                        | ASA         | `show conn`                                                                                                                                           | VPN client flows appear for tunneled destinations                                                   |
|   53 | Verify VPN session                                                             | ASA         | `show vpn-sessiondb remote` or `show vpn-sessiondb anyconnect`                                                                                        | User session shows expected group policy and assigned IP                                            |
|   54 | Verify IPsec or SSL counters                                                   | ASA         | `show crypto ipsec sa` or `show vpn-sessiondb detail anyconnect`                                                                                      | VPN traffic counters increment                                                                      |
|   55 | Check drops if traffic fails                                                   | ASA         | `show asp drop`                                                                                                                                       | Drop reason points to split tunnel, NAT, ACL, route, VPN filter, inspection, or host firewall issue |

```
# ASA_Split_Tunnel_VPN_NAT_Internet_Split_Tunnel_Local_Internet_Skeleton
configure terminal

access-list <split-acl> remark Internal networks tunneled by VPN clients
access-list <split-acl> standard permit <inside-network-1> <inside-mask-1>
access-list <split-acl> standard permit <inside-network-2> <inside-mask-2>

group-policy <group-policy-name> attributes
 split-tunnel-policy tunnelspecified
 split-tunnel-network-list value <split-acl>
 dns-server value <dns1> <dns2>
 default-domain value <domain-name>

object network <inside-object>
 subnet <inside-network> <inside-mask>

object network <vpn-pool-object>
 subnet <vpn-pool-network> <vpn-pool-mask>

nat (inside,outside) source static <inside-object> <inside-object> destination static <vpn-pool-object> <vpn-pool-object> no-proxy-arp route-lookup

write memory
```

```
# ASA_Split_Tunnel_VPN_NAT_Internet_Inside_Dynamic_PAT_Skeleton
configure terminal

object network <inside-object>
 subnet <inside-network> <inside-mask>
 nat (inside,outside) dynamic interface

write memory
```

```
# ASA_Split_Tunnel_VPN_NAT_Internet_Full_Tunnel_ASA_Internet_PAT_Skeleton
configure terminal

same-security-traffic permit intra-interface

object network <vpn-pool-object>
 subnet <vpn-pool-network> <vpn-pool-mask>
 nat (outside,outside) dynamic interface

object network <inside-object>
 subnet <inside-network> <inside-mask>

nat (inside,outside) source static <inside-object> <inside-object> destination static <vpn-pool-object> <vpn-pool-object> no-proxy-arp route-lookup

write memory
```

```
# ASA_Split_Tunnel_VPN_NAT_Internet_Outside_ACL_Inspection_Skeleton
configure terminal

no sysopt connection permit-vpn

access-list <outside-acl> remark Permit decrypted VPN users to inside resources
access-list <outside-acl> extended permit ip <vpn-pool-network> <vpn-pool-mask> <inside-network> <inside-mask>

access-list <outside-acl> remark Permit full-tunnel VPN users to Internet only if ASA Internet egress is intended
access-list <outside-acl> extended permit ip <vpn-pool-network> <vpn-pool-mask> any

access-group <outside-acl> in interface outside

write memory
```


# ASA_Split_Tunnel_VPN_NAT_Internet_Verification_Commands
| Task                                      | Command                                                                                          | Expected Result                                                                              |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------- |
| Verify interface state                    | `show interface ip brief`                                                                        | Inside and outside interfaces are up/up                                                      |
| Verify interface names                    | `show nameif`                                                                                    | Interface names match NAT and VPN config                                                     |
| Verify outside default route              | `show route`                                                                                     | ASA has outside route to Internet                                                            |
| Verify group policy                       | `show running-config group-policy <group-policy-name>`                                           | Group policy has expected split tunnel policy and network list                               |
| Verify split tunnel ACL                   | `show access-list <split-acl>`                                                                   | ACL lists only internal networks that should be tunneled                                     |
| Verify tunnel group policy mapping        | `show running-config tunnel-group <tunnel-group-name>`                                           | Tunnel group maps users to expected group policy                                             |
| Verify VPN session policy                 | `show vpn-sessiondb remote` or `show vpn-sessiondb anyconnect`                                   | User session shows expected group policy and assigned VPN IP                                 |
| Verify client route table                 | `route print`                                                                                    | Internal networks route through VPN; Internet route stays local for split tunnel             |
| Verify local Internet breakout            | `tracert 8.8.8.8`                                                                                | First hop is local gateway for split tunnel design                                           |
| Verify inside destination uses VPN        | `tracert <inside-host-ip>`                                                                       | Traffic goes through VPN path                                                                |
| Verify inside reachability                | `ping <inside-host-ip>`                                                                          | VPN client reaches permitted inside host                                                     |
| Verify Internet reachability              | `curl https://ifconfig.me` or browser test                                                       | Internet works                                                                               |
| Verify NAT table order                    | `show nat`                                                                                       | Identity NAT appears before broad PAT rules                                                  |
| Verify NAT details                        | `show nat detail`                                                                                | Inside-to-VPN identity NAT is present and hit count increments                               |
| Verify VPN pool PAT only if full tunnel   | `show nat detail`                                                                                | `(outside,outside)` VPN pool PAT has hits only when full-tunnel Internet through ASA is used |
| Verify xlate behavior                     | `show xlate`                                                                                     | Split tunnel shows no VPN Internet PAT; full tunnel shows VPN pool PAT to outside interface  |
| Verify local-host state                   | `show local-host <vpn-client-assigned-ip>`                                                       | VPN client flows are visible                                                                 |
| Verify connection table                   | `show conn address <vpn-client-assigned-ip>`                                                     | Tunneling flows appear for internal destinations                                             |
| Verify VPN filter if configured           | `show access-list <vpn-filter-acl>`                                                              | Permit/deny hits match intended access                                                       |
| Verify outside ACL if `sysopt` disabled   | `show access-list <outside-acl>`                                                                 | Decrypted VPN traffic hits outside ACL                                                       |
| Verify default VPN ACL bypass             | `show run all sysopt`                                                                            | `sysopt connection permit-vpn` is enabled unless intentionally disabled                      |
| Verify AnyConnect session detail          | `show vpn-sessiondb detail anyconnect`                                                           | Assigned IP, group policy, split tunnel, DNS, and protocol details are visible               |
| Verify IPsec counters if IPsec RA is used | `show crypto ipsec sa`                                                                           | Encrypt/decrypt counters increment                                                           |
| Verify drops                              | `show asp drop`                                                                                  | No relevant split tunnel, NAT, ACL, route, VPN filter, or inspection drops increment         |
| Packet-tracer internal flow               | `packet-tracer input outside ip <vpn-client-assigned-ip> 0 <inside-host-ip> 0 detailed`          | Flow should match VPN/inside policy and avoid incorrect PAT                                  |
| Packet-tracer full-tunnel Internet flow   | `packet-tracer input outside tcp <vpn-client-assigned-ip> <src-port> <internet-ip> 443 detailed` | Full-tunnel design should match `(outside,outside)` PAT                                      |

# ASA_Split_Tunnel_VPN_NAT_Internet_Rollback
| Step | Task                                                               | Device | Command                                                                                                                                                  | Expected Result                                           |
| ---: | ------------------------------------------------------------------ | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
|    1 | Identify split tunnel config                                       | ASA    | `show running-config group-policy <group-policy-name>`                                                                                                   | Split tunnel policy and ACL are identified                |
|    2 | Identify NAT rules                                                 | ASA    | `show nat detail`                                                                                                                                        | Identity NAT, inside PAT, and VPN pool PAT are identified |
|    3 | Identify VPN session impact                                        | ASA    | `show vpn-sessiondb remote` or `show vpn-sessiondb anyconnect`                                                                                           | Active users and assigned policies are visible            |
|    4 | Enter configuration mode                                           | ASA    | `configure terminal`                                                                                                                                     | ASA enters global configuration mode                      |
|    5 | Remove split tunnel ACL from group policy                          | ASA    | `group-policy <group-policy-name> attributes` then `no split-tunnel-network-list value <split-acl>`                                                      | Split tunnel ACL is detached                              |
|    6 | Remove split tunnel policy                                         | ASA    | `group-policy <group-policy-name> attributes` then `no split-tunnel-policy tunnelspecified`                                                              | Group policy no longer uses that split tunnel setting     |
|    7 | Remove split tunnel ACL if unused                                  | ASA    | `clear configure access-list <split-acl>`                                                                                                                | Split tunnel ACL is removed                               |
|    8 | Remove VPN pool full-tunnel PAT if not needed                      | ASA    | `object network <vpn-pool-object>` then `no nat (outside,outside) dynamic interface`                                                                     | VPN pool no longer PATs to Internet through ASA           |
|    9 | Remove same-interface forwarding only if no other feature needs it | ASA    | `no same-security-traffic permit intra-interface`                                                                                                        | Outside hairpin forwarding is disabled                    |
|   10 | Remove identity NAT only if rolling back inside VPN access         | ASA    | `no nat (inside,outside) source static <inside-object> <inside-object> destination static <vpn-pool-object> <vpn-pool-object> no-proxy-arp route-lookup` | Inside-to-VPN identity NAT is removed                     |
|   11 | Remove inside Dynamic PAT only if lab-only                         | ASA    | `object network <inside-object>` then `no nat (inside,outside) dynamic interface`                                                                        | Inside users no longer PAT through outside                |
|   12 | Remove outside ACL VPN permit if lab-only                          | ASA    | `no access-list <outside-acl> extended permit ip <vpn-pool-network> <vpn-pool-mask> <inside-network> <inside-mask>`                                      | VPN-specific outside ACL entry is removed                 |
|   13 | Remove full-tunnel Internet outside ACL permit if lab-only         | ASA    | `no access-list <outside-acl> extended permit ip <vpn-pool-network> <vpn-pool-mask> any`                                                                 | VPN Internet outside ACL entry is removed                 |
|   14 | Restore default VPN ACL bypass if changed                          | ASA    | `sysopt connection permit-vpn`                                                                                                                           | Decrypted VPN traffic bypasses interface ACLs again       |
|   15 | Clear stale translations                                           | ASA    | `clear xlate`                                                                                                                                            | Old NAT translations are cleared                          |
|   16 | Clear stale connections for test client                            | ASA    | `clear conn address <vpn-client-assigned-ip>`                                                                                                            | Old client flows are cleared                              |
|   17 | Log off test VPN user                                              | ASA    | `vpn-sessiondb logoff name <username>`                                                                                                                   | User must reconnect and receive updated policy            |
|   18 | Verify rollback                                                    | ASA    | `show running-config group-policy <group-policy-name>`                                                                                                   | Removed split tunnel settings no longer appear            |
|   19 | Verify NAT rollback                                                | ASA    | `show nat`                                                                                                                                               | Removed NAT rules no longer appear                        |
|   20 | Save rollback state                                                | ASA    | `write memory`                                                                                                                                           | Rollback is saved                                         |



# ASA_Split_Tunnel_VPN_NAT_Internet_Failure_Checks
| Symptom                                                              | Command                                                                            | What Usually Broke                                                                                                          |
| -------------------------------------------------------------------- | ---------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| VPN client Internet exits through ASA when split tunnel was intended | `route print` on client and `show running-config group-policy <group-policy-name>` | Split tunnel policy missing, wrong ACL, wrong group policy, or user landed in wrong tunnel group                            |
| VPN client cannot reach inside network                               | `route print`, `show nat detail`, `show access-list <vpn-filter-acl>`              | Split tunnel ACL missing internal route, NAT exemption missing, VPN filter blocks access, or inside host firewall           |
| Split tunnel ACL has no effect                                       | `show vpn-sessiondb remote` or `show vpn-sessiondb anyconnect`                     | User is not using the group policy where split tunnel is configured                                                         |
| Client still has old routes after change                             | Client reconnect and `show vpn-sessiondb`                                          | User did not reconnect after policy change                                                                                  |
| Internet fails in split tunnel mode                                  | Client route table                                                                 | Local client gateway, local DNS, endpoint firewall, or local network issue, not ASA NAT                                     |
| Internet fails in full tunnel mode                                   | `show nat detail`, `show xlate`, `show route`                                      | Missing `(outside,outside)` VPN pool PAT, missing same-interface forwarding, or no outside route                            |
| Full tunnel Internet traffic drops                                   | `show asp drop`                                                                    | Same-interface forwarding missing, outside ACL blocks it, NAT missing, or route missing                                     |
| VPN pool PAT has zero hits in full tunnel                            | `show nat detail`                                                                  | Client is still split tunneling Internet locally or full tunnel routes are missing                                          |
| VPN pool PAT has hits in split tunnel design                         | `show nat detail`                                                                  | Client is tunneling Internet unexpectedly or group policy is wrong                                                          |
| Inside-to-VPN traffic gets PATed                                     | `show xlate` and `show nat detail`                                                 | Identity NAT missing or ordered below a broad PAT rule                                                                      |
| NAT exemption has zero hits                                          | `show nat detail`                                                                  | Wrong inside object, wrong VPN pool object, wrong interface pair, or no traffic is reaching ASA                             |
| VPN filter permits but access still fails                            | `show conn`, `show asp drop`, host firewall checks                                 | NAT, route, inspection, or endpoint firewall issue                                                                          |
| DNS works only off VPN                                               | `show running-config group-policy <group-policy-name>`                             | DNS server/default domain not pushed, split DNS missing, or VPN filter blocks DNS                                           |
| DNS fails in split tunnel                                            | Client DNS settings and `show running-config group-policy`                         | Client uses local DNS for internal names or internal DNS route is missing from split tunnel ACL                             |
| Outside ACL unexpectedly blocks VPN users                            | `show run all sysopt` and `show access-list <outside-acl>`                         | `no sysopt connection permit-vpn` is configured and outside ACL lacks VPN pool permits                                      |
| Packet-tracer allows but real client fails                           | `show conn`, `show xlate`, captures, `show asp drop`                               | Stale state, client route issue, host firewall, or wrong active group policy                                                |
| Multiple VPN groups behave differently                               | `show vpn-sessiondb remote`                                                        | Different group policies have different split tunnel, DNS, VPN filter, or pool settings                                     |
| ASA CPU/load increases after change                                  | `show cpu usage`, `show conn count`                                                | You accidentally moved users from split tunnel to full tunnel through ASA                                                   |
| Security team rejects split tunnel                                   | Design review                                                                      | Split tunnel allows local Internet while connected to corporate VPN; endpoint firewall and posture controls may be required |
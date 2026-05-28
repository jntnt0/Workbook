
```
# Index
# Source_Basis
# ASA_Site_to_Site_IKEv1_Dynamic_Peer_Mental_Model
# ASA_Site_to_Site_IKEv1_Dynamic_Peer_Configuration_Checklist
# ASA_Site_to_Site_IKEv1_Dynamic_Peer_Skeleton
# ASA_Site_to_Site_IKEv1_Dynamic_Peer_With_PFS_Skeleton
# ASA_Site_to_Site_IKEv1_Dynamic_Peer_With_Outside_ACL_Inspection_Skeleton
# ASA_Site_to_Site_IKEv1_Dynamic_Peer_Verification_Commands
# ASA_Site_to_Site_IKEv1_Dynamic_Peer_Rollback
# ASA_Site_to_Site_IKEv1_Dynamic_Peer_Failure_Checks
```
# ASA_Site_to_Site_IKEv1_Dynamic_Peer_Mental_Model
| Concept                   | Operational Meaning                                                                                                                                                        |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Dynamic peer              | Remote VPN peer has a changing outside/public IP address                                                                                                                   |
| Static hub ASA            | ASA has a stable public IP and waits for the dynamic peer to initiate                                                                                                      |
| Dynamic peer limitation   | The ASA cannot initiate the tunnel because it does not know the current remote public IP                                                                                   |
| `DefaultL2LGroup`         | Catch-all tunnel group used when the peer IP is not configured as a named tunnel group                                                                                     |
| Dynamic crypto map        | Crypto map entry that accepts peers without a fixed `set peer` statement                                                                                                   |
| Static crypto map wrapper | The dynamic crypto map is attached to the real outside crypto map using `ipsec-isakmp dynamic`                                                                             |
| High sequence number      | Dynamic crypto map entries usually sit late in the crypto map, commonly `65535`, so specific static peers are checked first                                                |
| Crypto ACL                | Restricts which protected networks are allowed through the dynamic L2L tunnel                                                                                              |
| NAT exemption             | Keeps VPN traffic real/untranslated before encryption                                                                                                                      |
| Preshared key             | Usually configured under `DefaultL2LGroup` for unknown dynamic peers                                                                                                       |
| Phase 1                   | IKEv1 management tunnel; expected ASA state is usually `MM_ACTIVE`                                                                                                         |
| Phase 2                   | IPsec data tunnel; verified by proxy identities and encrypt/decrypt counters                                                                                               |
| Proxy identity            | The local and remote protected networks negotiated by the peer                                                                                                             |
| Responder role            | ASA normally shows as responder because the dynamic peer initiates the connection                                                                                          |
| Blunt rule                | This design is useful for labs and small branches, but it is weaker than a named static peer because multiple unknown peers can hit `DefaultL2LGroup` if they know the key |
# ASA_Site_to_Site_IKEv1_Dynamic_Peer_Configuration_Checklist
| Step | Task                                                                    | Device      | Command                                                                                                                                         | Expected Result                                                                      |
| ---: | ----------------------------------------------------------------------- | ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
|    1 | Confirm ASA interface status                                            | ASA         | `show interface ip brief`                                                                                                                       | Inside and outside interfaces are up/up                                              |
|    2 | Confirm logical interface names                                         | ASA         | `show nameif`                                                                                                                                   | Interfaces are named correctly, usually `inside` and `outside`                       |
|    3 | Confirm outside interface IP                                            | ASA         | `show running-config interface <outside-interface>`                                                                                             | ASA has a stable VPN termination IP                                                  |
|    4 | Confirm inside interface IP                                             | ASA         | `show running-config interface <inside-interface>`                                                                                              | ASA has expected inside LAN gateway IP                                               |
|    5 | Confirm local protected network                                         | ASA / Notes | `<local-network> <local-mask>`                                                                                                                  | Local LAN behind ASA is known                                                        |
|    6 | Confirm remote protected network                                        | ASA / Notes | `<remote-network> <remote-mask>`                                                                                                                | Remote LAN behind dynamic peer is known                                              |
|    7 | Confirm ASA public peer IP                                              | ASA / Notes | `<asa-public-ip>`                                                                                                                               | Dynamic peer will initiate toward this ASA public IP                                 |
|    8 | Confirm remote peer is actually dynamic                                 | ASA / Notes | Remote peer outside uses DHCP/dynamic ISP IP                                                                                                    | ASA should not use a fixed `set peer` for this peer                                  |
|    9 | Confirm preshared key                                                   | ASA / Notes | `<pre-shared-key>`                                                                                                                              | Same key is configured on ASA and dynamic peer                                       |
|   10 | Confirm Phase 1 parameters                                              | ASA / Notes | `authentication pre-share, encryption aes-256, hash sha, group 2, lifetime 86400`                                                               | Both peers use matching IKEv1 parameters                                             |
|   11 | Confirm Phase 2 parameters                                              | ASA / Notes | `esp-aes-256 esp-sha-hmac`                                                                                                                      | Both peers use matching IPsec transform set                                          |
|   12 | Confirm outside route to Internet                                       | ASA         | `show route`                                                                                                                                    | ASA has default route or route toward the Internet                                   |
|   13 | Confirm route to local protected network                                | ASA         | `show route <local-host-ip>`                                                                                                                    | ASA knows how to reach local protected hosts                                         |
|   14 | Review existing crypto maps                                             | ASA         | `show running-config crypto map`                                                                                                                | Existing static and dynamic crypto map entries are visible                           |
|   15 | Review existing IKEv1 policies                                          | ASA         | `show running-config crypto ikev1`                                                                                                              | Existing IKEv1 policies are visible                                                  |
|   16 | Review existing tunnel groups                                           | ASA         | `show running-config tunnel-group`                                                                                                              | Existing tunnel groups and `DefaultL2LGroup` usage are identified                    |
|   17 | Review existing NAT table                                               | ASA         | `show nat detail`                                                                                                                               | Existing NAT rules and broad PAT are visible                                         |
|   18 | Enter configuration mode                                                | ASA         | `configure terminal`                                                                                                                            | ASA enters global configuration mode                                                 |
|   19 | Create local protected network object                                   | ASA         | `object network <local-object>`                                                                                                                 | ASA enters local object mode                                                         |
|   20 | Define local protected network                                          | ASA         | `subnet <local-network> <local-mask>`                                                                                                           | Local protected network object is defined                                            |
|   21 | Create remote protected network object                                  | ASA         | `object network <remote-object>`                                                                                                                | ASA enters remote object mode                                                        |
|   22 | Define remote protected network                                         | ASA         | `subnet <remote-network> <remote-mask>`                                                                                                         | Remote protected network object is defined                                           |
|   23 | Configure identity NAT exemption                                        | ASA         | `nat (inside,outside) source static <local-object> <local-object> destination static <remote-object> <remote-object> no-proxy-arp route-lookup` | VPN traffic keeps real source and destination addresses                              |
|   24 | Create crypto ACL remark                                                | ASA         | `access-list <crypto-acl> remark Dynamic peer VPN traffic: <local-network> to <remote-network>`                                                 | Crypto ACL purpose is documented                                                     |
|   25 | Create crypto ACL permit                                                | ASA         | `access-list <crypto-acl> extended permit ip <local-network> <local-mask> <remote-network> <remote-mask>`                                       | Interesting traffic is defined                                                       |
|   26 | Configure IKEv1 transform set                                           | ASA         | `crypto ipsec ikev1 transform-set <transform-set-name> esp-aes-256 esp-sha-hmac`                                                                | Phase 2 transform set exists                                                         |
|   27 | Create dynamic crypto map ACL match                                     | ASA         | `crypto dynamic-map <dynamic-map-name> <dynamic-seq> match address <crypto-acl>`                                                                | Dynamic map accepts only the expected protected networks                             |
|   28 | Attach transform set to dynamic crypto map                              | ASA         | `crypto dynamic-map <dynamic-map-name> <dynamic-seq> set ikev1 transform-set <transform-set-name>`                                              | Dynamic map has the required IKEv1 transform set                                     |
|   29 | Configure PFS only if the dynamic peer also uses it                     | ASA         | `crypto dynamic-map <dynamic-map-name> <dynamic-seq> set pfs group2`                                                                            | PFS is configured only when both sides match                                         |
|   30 | Link dynamic map to outside crypto map                                  | ASA         | `crypto map <outside-crypto-map-name> <static-wrapper-seq> ipsec-isakmp dynamic <dynamic-map-name>`                                             | Static crypto map wrapper references the dynamic map                                 |
|   31 | Apply outside crypto map                                                | ASA         | `crypto map <outside-crypto-map-name> interface outside`                                                                                        | Crypto map is active on outside interface                                            |
|   32 | Enable IKEv1 on outside                                                 | ASA         | `crypto ikev1 enable outside`                                                                                                                   | ASA listens for IKEv1 on outside                                                     |
|   33 | Create IKEv1 policy                                                     | ASA         | `crypto ikev1 policy <priority>`                                                                                                                | ASA enters IKEv1 policy mode                                                         |
|   34 | Set authentication method                                               | ASA         | `authentication pre-share`                                                                                                                      | IKEv1 uses preshared key                                                             |
|   35 | Set IKEv1 encryption                                                    | ASA         | `encryption aes-256`                                                                                                                            | Encryption matches dynamic peer                                                      |
|   36 | Set IKEv1 hash                                                          | ASA         | `hash sha`                                                                                                                                      | Hash matches dynamic peer                                                            |
|   37 | Set IKEv1 DH group                                                      | ASA         | `group 2`                                                                                                                                       | DH group matches dynamic peer                                                        |
|   38 | Set IKEv1 lifetime                                                      | ASA         | `lifetime 86400`                                                                                                                                | Lifetime matches or is acceptable to peer                                            |
|   39 | Create group policy for dynamic L2L peers                               | ASA         | `group-policy <group-policy-name> internal`                                                                                                     | Internal group policy exists                                                         |
|   40 | Enter group policy attributes                                           | ASA         | `group-policy <group-policy-name> attributes`                                                                                                   | ASA enters group policy attribute mode                                               |
|   41 | Restrict tunnel protocol to IKEv1                                       | ASA         | `vpn-tunnel-protocol ikev1`                                                                                                                     | Group policy allows IKEv1                                                            |
|   42 | Configure DefaultL2LGroup general attributes                            | ASA         | `tunnel-group DefaultL2LGroup general-attributes`                                                                                               | ASA enters default L2L general attributes                                            |
|   43 | Apply group policy to DefaultL2LGroup                                   | ASA         | `default-group-policy <group-policy-name>`                                                                                                      | Unknown dynamic L2L peers use intended group policy                                  |
|   44 | Configure DefaultL2LGroup IPsec attributes                              | ASA         | `tunnel-group DefaultL2LGroup ipsec-attributes`                                                                                                 | ASA enters default L2L IPsec attributes                                              |
|   45 | Configure IKEv1 preshared key                                           | ASA         | `ikev1 pre-shared-key <pre-shared-key>`                                                                                                         | Dynamic peer can authenticate using the shared key                                   |
|   46 | Confirm no static peer command exists for dynamic peer                  | ASA         | `show running-config crypto map`                                                                                                                | Dynamic map does not have `set peer <dynamic-peer-ip>`                               |
|   47 | Confirm static peer entries have lower sequence numbers if present      | ASA         | `show running-config crypto map`                                                                                                                | Specific static VPNs are evaluated before the catch-all dynamic map                  |
|   48 | Confirm default VPN ACL bypass behavior                                 | ASA         | `show run all sysopt`                                                                                                                           | `sysopt connection permit-vpn` is enabled unless intentionally changed               |
|   49 | Add VPN interface ACL only if `no sysopt connection permit-vpn` is used | ASA         | `access-list <outside-acl> extended permit ip <remote-network> <remote-mask> <local-network> <local-mask>`                                      | Decrypted VPN traffic is permitted by outside ACL                                    |
|   50 | Save configuration                                                      | ASA         | `write memory`                                                                                                                                  | Dynamic peer VPN configuration is saved                                              |
|   51 | Wait for dynamic peer to initiate                                       | Remote Peer | `ping <asa-side-protected-host>`                                                                                                                | Dynamic peer generates interesting traffic toward ASA                                |
|   52 | Verify IKEv1 Phase 1                                                    | ASA         | `show crypto ikev1 sa detail`                                                                                                                   | ASA shows L2L responder session, usually `MM_ACTIVE`                                 |
|   53 | Verify IPsec Phase 2                                                    | ASA         | `show crypto ipsec sa`                                                                                                                          | Local and remote proxy identities match expected networks                            |
|   54 | Verify encrypt/decrypt counters                                         | ASA         | `show crypto ipsec sa`                                                                                                                          | Encaps/encrypt and decaps/decrypt counters increment                                 |
|   55 | Verify NAT exemption hit count                                          | ASA         | `show nat detail`                                                                                                                               | Identity NAT hit count increments                                                    |
|   56 | Verify crypto ACL hit count                                             | ASA         | `show access-list <crypto-acl>`                                                                                                                 | Crypto ACL hit count increments for interesting traffic                              |
|   57 | Verify active VPN session                                               | ASA         | `show vpn-sessiondb summary`                                                                                                                    | Active IKEv1 IPsec L2L session appears                                               |
|   58 | Check drops if tunnel fails                                             | ASA         | `show asp drop`                                                                                                                                 | Drop reason points to NAT, ACL, crypto ACL, route, proposal, or inspection issue     |
|   59 | Debug IKEv1 only when needed                                            | ASA         | `debug crypto ikev1 127`                                                                                                                        | Debug shows dynamic peer landing on `DefaultL2LGroup` or failing proposal/key checks |
|   60 | Debug IPsec only when needed                                            | ASA         | `debug crypto ipsec 127`                                                                                                                        | Debug shows Phase 2 transform/proxy-ID negotiation                                   |
|   61 | Stop debugs                                                             | ASA         | `undebug all`                                                                                                                                   | Debug output stops                                                                   |
```
# ASA_Site_to_Site_IKEv1_Dynamic_Peer_Skeleton
configure terminal

object network <local-object>
 subnet <local-network> <local-mask>

object network <remote-object>
 subnet <remote-network> <remote-mask>

nat (inside,outside) source static <local-object> <local-object> destination static <remote-object> <remote-object> no-proxy-arp route-lookup

access-list <crypto-acl> remark Dynamic peer VPN traffic from <local-network> to <remote-network>
access-list <crypto-acl> extended permit ip <local-network> <local-mask> <remote-network> <remote-mask>

crypto ipsec ikev1 transform-set <transform-set-name> esp-aes-256 esp-sha-hmac

crypto dynamic-map <dynamic-map-name> <dynamic-seq> match address <crypto-acl>
crypto dynamic-map <dynamic-map-name> <dynamic-seq> set ikev1 transform-set <transform-set-name>

crypto map <outside-crypto-map-name> <static-wrapper-seq> ipsec-isakmp dynamic <dynamic-map-name>
crypto map <outside-crypto-map-name> interface outside

crypto ikev1 enable outside
crypto ikev1 policy <priority>
 authentication pre-share
 encryption aes-256
 hash sha
 group 2
 lifetime 86400

group-policy <group-policy-name> internal
group-policy <group-policy-name> attributes
 vpn-tunnel-protocol ikev1

tunnel-group DefaultL2LGroup general-attributes
 default-group-policy <group-policy-name>
tunnel-group DefaultL2LGroup ipsec-attributes
 ikev1 pre-shared-key <pre-shared-key>

write memory
```


```
# ASA_Site_to_Site_IKEv1_Dynamic_Peer_With_PFS_Skeleton
configure terminal

object network <local-object>
 subnet <local-network> <local-mask>

object network <remote-object>
 subnet <remote-network> <remote-mask>

nat (inside,outside) source static <local-object> <local-object> destination static <remote-object> <remote-object> no-proxy-arp route-lookup

access-list <crypto-acl> remark Dynamic peer VPN traffic from <local-network> to <remote-network>
access-list <crypto-acl> extended permit ip <local-network> <local-mask> <remote-network> <remote-mask>

crypto ipsec ikev1 transform-set <transform-set-name> esp-aes-256 esp-sha-hmac

crypto dynamic-map <dynamic-map-name> <dynamic-seq> match address <crypto-acl>
crypto dynamic-map <dynamic-map-name> <dynamic-seq> set ikev1 transform-set <transform-set-name>
crypto dynamic-map <dynamic-map-name> <dynamic-seq> set pfs group2

crypto map <outside-crypto-map-name> <static-wrapper-seq> ipsec-isakmp dynamic <dynamic-map-name>
crypto map <outside-crypto-map-name> interface outside

crypto ikev1 enable outside
crypto ikev1 policy <priority>
 authentication pre-share
 encryption aes-256
 hash sha
 group 2
 lifetime 86400

group-policy <group-policy-name> internal
group-policy <group-policy-name> attributes
 vpn-tunnel-protocol ikev1

tunnel-group DefaultL2LGroup general-attributes
 default-group-policy <group-policy-name>
tunnel-group DefaultL2LGroup ipsec-attributes
 ikev1 pre-shared-key <pre-shared-key>

write memory
```

```
# ASA_Site_to_Site_IKEv1_Dynamic_Peer_With_Outside_ACL_Inspection_Skeleton
configure terminal

no sysopt connection permit-vpn

access-list <outside-acl> extended permit ip <remote-network> <remote-mask> <local-network> <local-mask>
access-group <outside-acl> in interface outside

write memory
```

# ASA_Site_to_Site_IKEv1_Dynamic_Peer_Verification_Commands
| Task                                  | Command                                                | Expected Result                                                         |
| ------------------------------------- | ------------------------------------------------------ | ----------------------------------------------------------------------- |
| Verify interface state                | `show interface ip brief`                              | Inside and outside interfaces are up/up                                 |
| Verify interface names                | `show nameif`                                          | Interface names match the NAT and crypto map interface references       |
| Verify route to Internet              | `show route`                                           | ASA has default route or route toward outside                           |
| Verify local protected route          | `show route <local-host-ip>`                           | ASA knows local protected network path                                  |
| Verify local object                   | `show running-config object network <local-object>`    | Local protected object has correct subnet                               |
| Verify remote object                  | `show running-config object network <remote-object>`   | Remote protected object has correct subnet                              |
| Verify NAT exemption                  | `show nat`                                             | Identity NAT appears before broad Dynamic PAT                           |
| Verify NAT details                    | `show nat detail`                                      | Identity NAT rule shows local-to-local and remote-to-remote mapping     |
| Verify crypto ACL                     | `show access-list <crypto-acl>`                        | ACL permits local protected network to remote protected network         |
| Verify transform set                  | `show running-config crypto ipsec`                     | IKEv1 transform set exists                                              |
| Verify dynamic crypto map             | `show running-config crypto dynamic-map`               | Dynamic map has crypto ACL match and transform set                      |
| Verify static wrapper crypto map      | `show running-config crypto map`                       | Static crypto map links to dynamic map with `ipsec-isakmp dynamic`      |
| Verify outside crypto map application | `show running-config crypto map`                       | Crypto map is applied to outside interface                              |
| Verify IKEv1 enabled                  | `show running-config crypto ikev1`                     | `crypto ikev1 enable outside` and matching policy exist                 |
| Verify DefaultL2LGroup key            | `show running-config tunnel-group DefaultL2LGroup`     | `ikev1 pre-shared-key` is configured under IPsec attributes             |
| Verify group policy                   | `show running-config group-policy <group-policy-name>` | Group policy permits `vpn-tunnel-protocol ikev1`                        |
| Verify default VPN ACL bypass         | `show run all sysopt`                                  | `sysopt connection permit-vpn` is enabled unless intentionally disabled |
| Trigger from dynamic peer             | `ping <asa-side-protected-host>`                       | Dynamic peer initiates the tunnel                                       |
| Verify IKEv1 Phase 1                  | `show crypto ikev1 sa detail`                          | ASA shows L2L responder session, usually `MM_ACTIVE`                    |
| Verify old-style IKEv1 state if used  | `show crypto isakmp sa`                                | ASA shows peer in `MM_ACTIVE`                                           |
| Verify IPsec Phase 2                  | `show crypto ipsec sa`                                 | Inbound and outbound ESP SAs exist                                      |
| Verify dynamic peer endpoint          | `show crypto ipsec sa`                                 | `current_peer` shows current dynamic public IP                          |
| Verify proxy identities               | `show crypto ipsec sa`                                 | Local and remote idents match crypto ACL networks                       |
| Verify outbound encryption            | `show crypto ipsec sa`                                 | Encaps/encrypt counters increment                                       |
| Verify inbound decryption             | `show crypto ipsec sa`                                 | Decaps/decrypt counters increment                                       |
| Verify NAT hit count                  | `show nat detail`                                      | NAT exemption hit count increments                                      |
| Verify connection table               | `show conn`                                            | Local-to-remote VPN flows appear                                        |
| Verify translation table              | `show xlate`                                           | VPN traffic is not PATed to outside interface                           |
| Verify VPN session summary            | `show vpn-sessiondb summary`                           | Active IKEv1 IPsec L2L session appears                                  |
| Verify drops                          | `show asp drop`                                        | No relevant NAT, ACL, crypto, route, or inspection drops increment      |
| Debug IKEv1 if needed                 | `debug crypto ikev1 127`                               | Debug shows peer landing on `DefaultL2LGroup` and Phase 1 behavior      |
| Debug IPsec if needed                 | `debug crypto ipsec 127`                               | Debug shows Phase 2 and proxy-ID behavior                               |
| Stop debugs                           | `undebug all`                                          | Debug output stops                                                      |

# ASA_Site_to_Site_IKEv1_Dynamic_Peer_Rollback
| Step | Task                                                                     | Device | Command                                                                                                                                            | Expected Result                                                |
| ---: | ------------------------------------------------------------------------ | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
|    1 | Identify dynamic crypto map                                              | ASA    | `show running-config crypto dynamic-map`                                                                                                           | Dynamic map name and sequence are identified                   |
|    2 | Identify static wrapper crypto map                                       | ASA    | `show running-config crypto map`                                                                                                                   | Wrapper crypto map name and sequence are identified            |
|    3 | Identify DefaultL2LGroup settings                                        | ASA    | `show running-config tunnel-group DefaultL2LGroup`                                                                                                 | Default group policy and preshared key settings are identified |
|    4 | Identify NAT exemption                                                   | ASA    | `show nat detail`                                                                                                                                  | Identity NAT rule and object names are identified              |
|    5 | Enter configuration mode                                                 | ASA    | `configure terminal`                                                                                                                               | ASA enters global configuration mode                           |
|    6 | Remove crypto map from outside interface                                 | ASA    | `no crypto map <outside-crypto-map-name> interface outside`                                                                                        | Crypto map is detached from outside interface                  |
|    7 | Remove static wrapper dynamic map line                                   | ASA    | `no crypto map <outside-crypto-map-name> <static-wrapper-seq> ipsec-isakmp dynamic <dynamic-map-name>`                                             | Dynamic map is no longer linked to outside crypto map          |
|    8 | Remove PFS from dynamic map if configured                                | ASA    | `no crypto dynamic-map <dynamic-map-name> <dynamic-seq> set pfs group2`                                                                            | PFS setting is removed                                         |
|    9 | Remove dynamic map transform set                                         | ASA    | `no crypto dynamic-map <dynamic-map-name> <dynamic-seq> set ikev1 transform-set <transform-set-name>`                                              | Transform set is removed from dynamic map                      |
|   10 | Remove dynamic map ACL match                                             | ASA    | `no crypto dynamic-map <dynamic-map-name> <dynamic-seq> match address <crypto-acl>`                                                                | ACL match is removed from dynamic map                          |
|   11 | Remove crypto ACL                                                        | ASA    | `clear configure access-list <crypto-acl>`                                                                                                         | Crypto ACL is removed                                          |
|   12 | Remove NAT exemption                                                     | ASA    | `no nat (inside,outside) source static <local-object> <local-object> destination static <remote-object> <remote-object> no-proxy-arp route-lookup` | Identity NAT rule is removed                                   |
|   13 | Remove transform set                                                     | ASA    | `no crypto ipsec ikev1 transform-set <transform-set-name>`                                                                                         | IKEv1 transform set is removed                                 |
|   14 | Remove DefaultL2LGroup group policy assignment if used only for this lab | ASA    | `tunnel-group DefaultL2LGroup general-attributes` then `no default-group-policy <group-policy-name>`                                               | Default group policy override is removed                       |
|   15 | Remove DefaultL2LGroup preshared key if used only for this lab           | ASA    | `tunnel-group DefaultL2LGroup ipsec-attributes` then `no ikev1 pre-shared-key <pre-shared-key>`                                                    | Catch-all dynamic peer key is removed                          |
|   16 | Remove group policy if unused                                            | ASA    | `clear configure group-policy <group-policy-name>`                                                                                                 | Group policy is removed if not shared                          |
|   17 | Remove IKEv1 policy if unused                                            | ASA    | `clear configure crypto ikev1 policy <priority>`                                                                                                   | IKEv1 policy is removed                                        |
|   18 | Disable IKEv1 on outside only if no other IKEv1 VPNs use it              | ASA    | `no crypto ikev1 enable outside`                                                                                                                   | ASA no longer listens for IKEv1 on outside                     |
|   19 | Remove outside ACL permit if added only for this lab                     | ASA    | `no access-list <outside-acl> extended permit ip <remote-network> <remote-mask> <local-network> <local-mask>`                                      | VPN-specific ACL entry is removed                              |
|   20 | Restore default VPN ACL bypass if changed                                | ASA    | `sysopt connection permit-vpn`                                                                                                                     | Decrypted VPN traffic bypasses interface ACLs again            |
|   21 | Remove local object if unused                                            | ASA    | `no object network <local-object>`                                                                                                                 | Local object is removed                                        |
|   22 | Remove remote object if unused                                           | ASA    | `no object network <remote-object>`                                                                                                                | Remote object is removed                                       |
|   23 | Clear IKEv1 SAs                                                          | ASA    | `clear crypto ikev1 sa`                                                                                                                            | IKEv1 Phase 1 state is cleared                                 |
|   24 | Clear IPsec SAs                                                          | ASA    | `clear crypto ipsec sa`                                                                                                                            | IPsec Phase 2 state is cleared                                 |
|   25 | Clear stale translations                                                 | ASA    | `clear xlate`                                                                                                                                      | Old NAT translations are cleared                               |
|   26 | Clear stale connections                                                  | ASA    | `clear conn address <local-host-ip>`                                                                                                               | Old connection state is cleared                                |
|   27 | Verify rollback                                                          | ASA    | `show running-config crypto`                                                                                                                       | Removed dynamic VPN crypto config no longer appears            |
|   28 | Save rollback state                                                      | ASA    | `write memory`                                                                                                                                     | Rollback is saved                                              |

# ASA_Site_to_Site_IKEv1_Dynamic_Peer_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| ASA never sees the tunnel attempt | `show crypto ikev1 sa detail` | Dynamic peer has not initiated, wrong ASA public IP, upstream NAT/firewall issue, or UDP/500 blocked |
| ASA cannot initiate the tunnel | Design check | Expected behavior; dynamic peer must initiate because ASA does not know the peer's current public IP |
| Dynamic peer lands on `DefaultL2LGroup` | `debug crypto ikev1 127` | Expected behavior for unknown dynamic peer if no named peer mapping exists |
| Phase 1 fails after landing on `DefaultL2LGroup` | `debug crypto ikev1 127` | Preshared key mismatch or IKEv1 policy mismatch |
| Debug says proposal unacceptable | `debug crypto ikev1 127` | Phase 1 encryption, hash, authentication, DH group, or lifetime mismatch |
| Debug indicates hash/decrypt failure | `debug crypto ikev1 127` | Preshared key mismatch |
| Phase 1 reaches `MM_ACTIVE` but Phase 2 fails | `debug crypto ipsec 127` and `show crypto ipsec sa` | Transform set, PFS, crypto ACL, or proxy-ID mismatch |
| Debug says proxy IDs do not match | `debug crypto ikev1 127` | Local and remote protected networks do not mirror the peer configuration |
| Dynamic crypto map has no transform set | `show running-config crypto dynamic-map` | Missing `set ikev1 transform-set` under dynamic map |
| Dynamic map is configured but unused | `show running-config crypto map` | Dynamic map is not linked to static crypto map with `ipsec-isakmp dynamic` |
| Crypto map exists but tunnel never builds | `show running-config crypto map` | Crypto map not applied to outside interface |
| Static VPNs break after adding dynamic map | `show running-config crypto map` | Dynamic map sequence placed too early; it should usually be late, such as 65535 |
| NAT exemption has zero hits | `show nat detail` | Wrong local object, wrong remote object, wrong interface pair, or traffic is not hitting ASA |
| Traffic gets PATed instead of encrypted | `packet-tracer input inside ... detailed` | Missing identity NAT or broad Dynamic PAT matched first |
| Crypto ACL has zero hits | `show access-list <crypto-acl>` | No interesting traffic, wrong source/destination, or traffic starts from wrong side |
| Encrypt counter increments but decrypt does not | `show crypto ipsec sa` | Remote peer not returning traffic, remote NAT issue, peer crypto ACL mismatch, or remote firewall issue |
| Decrypt counter increments but local host gets no reply | `show conn`, `show asp drop`, host firewall check | ASA receives tunnel traffic but local ACL, route, host firewall, or return path fails |
| Dynamic peer public IP changes and tunnel drops | `show crypto ikev1 sa detail` | Expected after ISP/DHCP change; remote side must reinitiate |
| Multiple dynamic peers collide | `show vpn-sessiondb summary` and `show crypto ipsec sa` | Shared `DefaultL2LGroup` and broad crypto ACL/key allow ambiguous matching |
| Outside ACL blocks decrypted VPN traffic | `show run all sysopt` and `show access-list <outside-acl>` | `no sysopt connection permit-vpn` is configured but ACL does not permit remote-to-local traffic |
| Packet-tracer looks clean but real traffic fails | `show conn`, `show xlate`, captures, `show asp drop` | Stale state, remote peer issue, host firewall, or asymmetric routing |
| Debugs flood the terminal | `show debug` | Debugs left enabled; run `undebug all` |

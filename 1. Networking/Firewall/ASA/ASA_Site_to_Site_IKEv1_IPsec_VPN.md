```
# Index
# Source_Basis
# ASA_Site_to_Site_IKEv1_IPsec_VPN_Mental_Model
# ASA_Site_to_Site_IKEv1_IPsec_VPN_Configuration_Checklist
# ASA_Site_to_Site_IKEv1_IPsec_VPN_Skeleton
# ASA_Site_to_Site_IKEv1_IPsec_VPN_With_PFS_Skeleton
# ASA_Site_to_Site_IKEv1_IPsec_VPN_With_Outside_ACL_Inspection_Skeleton
# ASA_Site_to_Site_IKEv1_IPsec_VPN_Verification_Commands
# ASA_Site_to_Site_IKEv1_IPsec_VPN_Rollback
# ASA_Site_to_Site_IKEv1_IPsec_VPN_Failure_Checks
```
###### ASA_Site_to_Site_IKEv1_IPsec_VPN_Mental_Model
| Concept                        | Operational Meaning                                                                                                                                     |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| IKEv1 Phase 1                  | Builds the management/control channel between VPN peers                                                                                                 |
| IKEv1 Phase 2                  | Builds the IPsec data SAs for protected traffic                                                                                                         |
| ISAKMP policy                  | Must match peer on authentication, encryption, hash, DH group, and lifetime                                                                             |
| Transform set                  | Defines ESP encryption and integrity for protected traffic                                                                                              |
| Tunnel group                   | Ties the peer IP to L2L behavior and the preshared key                                                                                                  |
| Crypto ACL                     | Defines interesting traffic that should be encrypted                                                                                                    |
| Crypto map                     | Binds crypto ACL, peer IP, transform set, optional PFS, and outside interface together                                                                  |
| Identity NAT / NAT exemption   | Prevents VPN traffic from being PATed before encryption                                                                                                 |
| Proxy identities               | The local and remote networks negotiated in Phase 2                                                                                                     |
| `MM_ACTIVE`                    | IKEv1 Phase 1 is established                                                                                                                            |
| Encaps / encrypt counters      | ASA is sending traffic into the tunnel                                                                                                                  |
| Decaps / decrypt counters      | ASA is receiving traffic from the tunnel                                                                                                                |
| `sysopt connection permit-vpn` | Default behavior that lets decrypted VPN traffic bypass interface ACL checks                                                                            |
| PFS                            | Optional extra DH exchange for Phase 2 keying                                                                                                           |
| Blunt rule                     | If Phase 1 is down, check IKE policy and preshared key. If Phase 1 is up but traffic fails, check crypto ACL, NAT exemption, routes, and IPsec counters |
##### ASA_Site_to_Site_IKEv1_IPsec_VPN_Configuration_Checklist
| Step | Task                                                                     | Device              | Command                                                                                                                                         | Expected Result                                                                           |
| ---: | ------------------------------------------------------------------------ | ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
|    1 | Confirm ASA interface status                                             | ASA                 | `show interface ip brief`                                                                                                                       | Inside and outside interfaces are up/up                                                   |
|    2 | Confirm ASA logical names                                                | ASA                 | `show nameif`                                                                                                                                   | Interfaces are named correctly, usually `inside` and `outside`                            |
|    3 | Confirm outside interface IP                                             | ASA                 | `show running-config interface <outside-interface>`                                                                                             | Outside interface has expected public or transit IP                                       |
|    4 | Confirm inside interface IP                                              | ASA                 | `show running-config interface <inside-interface>`                                                                                              | Inside interface has expected LAN gateway IP                                              |
|    5 | Confirm local protected network                                          | ASA / Notes         | `<local-network> <local-mask>`                                                                                                                  | Local LAN behind this ASA is known                                                        |
|    6 | Confirm remote protected network                                         | ASA / Notes         | `<remote-network> <remote-mask>`                                                                                                                | Remote LAN behind peer ASA/router is known                                                |
|    7 | Confirm local peer public IP                                             | ASA / Notes         | `<local-peer-public-ip>`                                                                                                                        | Local ASA VPN termination address is known                                                |
|    8 | Confirm remote peer public IP                                            | ASA / Notes         | `<remote-peer-public-ip>`                                                                                                                       | Peer VPN termination address is known                                                     |
|    9 | Confirm preshared key                                                    | ASA / Notes         | `<pre-shared-key>`                                                                                                                              | Same key is agreed on both VPN peers                                                      |
|   10 | Confirm IKEv1 Phase 1 parameters                                         | ASA / Notes         | `authentication pre-share, encryption aes-256, hash sha, group 2, lifetime 86400`                                                               | Both peers use matching Phase 1 parameters                                                |
|   11 | Confirm IPsec Phase 2 parameters                                         | ASA / Notes         | `esp-aes-256 esp-sha-hmac`                                                                                                                      | Both peers use matching transform set                                                     |
|   12 | Confirm outside route to peer                                            | ASA                 | `show route <remote-peer-public-ip>`                                                                                                            | ASA has route toward the peer public IP                                                   |
|   13 | Confirm route to remote protected network expectation                    | ASA                 | `show route <remote-network-ip>`                                                                                                                | Route points toward outside/VPN path or crypto map behavior is expected                   |
|   14 | Review current NAT table                                                 | ASA                 | `show nat`                                                                                                                                      | Existing NAT rules are visible                                                            |
|   15 | Review current NAT order and hits                                        | ASA                 | `show nat detail`                                                                                                                               | Broad PAT rules and existing exemptions are visible                                       |
|   16 | Review current crypto config                                             | ASA                 | `show running-config crypto`                                                                                                                    | Existing IKE/IPsec/crypto map config is visible                                           |
|   17 | Review current tunnel groups                                             | ASA                 | `show running-config tunnel-group`                                                                                                              | Existing L2L peers and keys are identified                                                |
|   18 | Enter configuration mode                                                 | ASA                 | `configure terminal`                                                                                                                            | ASA enters global configuration mode                                                      |
|   19 | Create local protected network object                                    | ASA                 | `object network <local-object>`                                                                                                                 | ASA enters local object mode                                                              |
|   20 | Define local protected network                                           | ASA                 | `subnet <local-network> <local-mask>`                                                                                                           | Local protected network object is defined                                                 |
|   21 | Create remote protected network object                                   | ASA                 | `object network <remote-object>`                                                                                                                | ASA enters remote object mode                                                             |
|   22 | Define remote protected network                                          | ASA                 | `subnet <remote-network> <remote-mask>`                                                                                                         | Remote protected network object is defined                                                |
|   23 | Configure NAT exemption                                                  | ASA                 | `nat (inside,outside) source static <local-object> <local-object> destination static <remote-object> <remote-object> no-proxy-arp route-lookup` | VPN traffic keeps real source and destination addresses                                   |
|   24 | Create crypto ACL remark                                                 | ASA                 | `access-list <crypto-acl> remark VPN interesting traffic: <local-network> to <remote-network>`                                                  | Crypto ACL purpose is documented                                                          |
|   25 | Create crypto ACL permit                                                 | ASA                 | `access-list <crypto-acl> extended permit ip <local-network> <local-mask> <remote-network> <remote-mask>`                                       | ASA defines interesting traffic for encryption                                            |
|   26 | Configure IKEv1 transform set                                            | ASA                 | `crypto ipsec ikev1 transform-set <transform-set-name> esp-aes-256 esp-sha-hmac`                                                                | Phase 2 transform set exists                                                              |
|   27 | Create crypto map match statement                                        | ASA                 | `crypto map <crypto-map-name> <seq> match address <crypto-acl>`                                                                                 | Crypto map references interesting traffic ACL                                             |
|   28 | Set VPN peer                                                             | ASA                 | `crypto map <crypto-map-name> <seq> set peer <remote-peer-public-ip>`                                                                           | Crypto map points to remote VPN peer                                                      |
|   29 | Attach IKEv1 transform set to crypto map                                 | ASA                 | `crypto map <crypto-map-name> <seq> set ikev1 transform-set <transform-set-name>`                                                               | Crypto map uses the correct IKEv1 transform set                                           |
|   30 | Enable PFS only if both peers agree                                      | ASA                 | `crypto map <crypto-map-name> <seq> set pfs group2`                                                                                             | PFS is configured only when required by design                                            |
|   31 | Apply crypto map to outside interface                                    | ASA                 | `crypto map <crypto-map-name> interface outside`                                                                                                | Crypto map is bound to the VPN termination interface                                      |
|   32 | Enable IKEv1 on outside interface                                        | ASA                 | `crypto ikev1 enable outside`                                                                                                                   | ASA listens for IKEv1 on outside                                                          |
|   33 | Create IKEv1 policy                                                      | ASA                 | `crypto ikev1 policy <priority>`                                                                                                                | ASA enters IKEv1 policy mode                                                              |
|   34 | Set IKEv1 authentication                                                 | ASA                 | `authentication pre-share`                                                                                                                      | IKEv1 uses preshared key authentication                                                   |
|   35 | Set IKEv1 encryption                                                     | ASA                 | `encryption aes-256`                                                                                                                            | IKEv1 encryption matches peer                                                             |
|   36 | Set IKEv1 hash                                                           | ASA                 | `hash sha`                                                                                                                                      | IKEv1 integrity hash matches peer                                                         |
|   37 | Set IKEv1 DH group                                                       | ASA                 | `group 2`                                                                                                                                       | IKEv1 DH group matches peer                                                               |
|   38 | Set IKEv1 lifetime                                                       | ASA                 | `lifetime 86400`                                                                                                                                | IKEv1 lifetime matches peer or acceptable negotiation range                               |
|   39 | Create group policy                                                      | ASA                 | `group-policy <group-policy-name> internal`                                                                                                     | Internal group policy exists                                                              |
|   40 | Enter group policy attributes                                            | ASA                 | `group-policy <group-policy-name> attributes`                                                                                                   | ASA enters group policy attribute mode                                                    |
|   41 | Restrict tunnel protocol to IKEv1                                        | ASA                 | `vpn-tunnel-protocol ikev1`                                                                                                                     | Group policy allows IKEv1                                                                 |
|   42 | Create L2L tunnel group                                                  | ASA                 | `tunnel-group <remote-peer-public-ip> type ipsec-l2l`                                                                                           | Peer-specific L2L tunnel group exists                                                     |
|   43 | Enter tunnel group general attributes                                    | ASA                 | `tunnel-group <remote-peer-public-ip> general-attributes`                                                                                       | ASA enters tunnel group general attributes                                                |
|   44 | Apply default group policy                                               | ASA                 | `default-group-policy <group-policy-name>`                                                                                                      | Tunnel group uses intended group policy                                                   |
|   45 | Enter IPsec attributes                                                   | ASA                 | `tunnel-group <remote-peer-public-ip> ipsec-attributes`                                                                                         | ASA enters IPsec tunnel attributes                                                        |
|   46 | Configure IKEv1 preshared key                                            | ASA                 | `ikev1 pre-shared-key <pre-shared-key>`                                                                                                         | Peer key is configured                                                                    |
|   47 | Confirm default VPN ACL bypass behavior                                  | ASA                 | `show run all sysopt`                                                                                                                           | `sysopt connection permit-vpn` is enabled unless you intentionally require ACL inspection |
|   48 | Configure VPN ACL inspection only if required                            | ASA                 | `no sysopt connection permit-vpn`                                                                                                               | Decrypted VPN traffic is forced through interface ACLs                                    |
|   49 | Add outside ACL permit only if `no sysopt connection permit-vpn` is used | ASA                 | `access-list <outside-acl> extended permit ip <remote-network> <remote-mask> <local-network> <local-mask>`                                      | Interface ACL permits decrypted VPN traffic                                               |
|   50 | Apply outside ACL only if required                                       | ASA                 | `access-group <outside-acl> in interface outside`                                                                                               | Decrypted VPN traffic is inspected and allowed by ACL                                     |
|   51 | Save configuration                                                       | ASA                 | `write memory`                                                                                                                                  | VPN configuration is saved                                                                |
|   52 | Verify crypto map                                                        | ASA                 | `show running-config crypto map`                                                                                                                | Crypto ACL, peer, transform set, PFS, and interface binding are present                   |
|   53 | Verify IKEv1 policy                                                      | ASA                 | `show running-config crypto ikev1`                                                                                                              | IKEv1 enable and policy are present                                                       |
|   54 | Verify tunnel group                                                      | ASA                 | `show running-config tunnel-group <remote-peer-public-ip>`                                                                                      | Tunnel group and IKEv1 preshared key line are present                                     |
|   55 | Verify NAT exemption placement                                           | ASA                 | `show nat`                                                                                                                                      | Identity NAT appears before broad Dynamic PAT                                             |
|   56 | Simulate interesting traffic                                             | ASA                 | `packet-tracer input inside icmp <local-host-ip> 8 0 <remote-host-ip> detailed`                                                                 | Packet-tracer matches NAT exemption and crypto policy path                                |
|   57 | Generate real interesting traffic                                        | Local Host / Router | `ping <remote-host-ip>`                                                                                                                         | Traffic triggers tunnel negotiation                                                       |
|   58 | Verify IKEv1 Phase 1                                                     | ASA                 | `show crypto ikev1 sa detail`                                                                                                                   | State shows `MM_ACTIVE`                                                                   |
|   59 | Verify IPsec Phase 2                                                     | ASA                 | `show crypto ipsec sa`                                                                                                                          | Local/remote proxy IDs match expected networks                                            |
|   60 | Verify packet counters                                                   | ASA                 | `show crypto ipsec sa`                                                                                                                          | Encaps/encrypt and decaps/decrypt counters increment                                      |
|   61 | Verify VPN session summary                                               | ASA                 | `show vpn-sessiondb summary`                                                                                                                    | Active Site-to-Site VPN / IKEv1 IPsec session appears                                     |
|   62 | Check tunnel failure drops                                               | ASA                 | `show asp drop`                                                                                                                                 | Drop reason points to ACL, NAT, crypto ACL, route, or inspection issue                    |
|   63 | Debug only when needed                                                   | ASA                 | `debug crypto ikev1 127` and `debug crypto ipsec 127`                                                                                           | Debugs show Phase 1, Phase 2, key, proposal, or proxy-ID issue                            |
|   64 | Disable debugs after troubleshooting                                     | ASA                 | `undebug all`                                                                                                                                   | Debug output stops                                                                        |
```
# ASA_Site_to_Site_IKEv1_IPsec_VPN_Skeleton
configure terminal

object network <local-object>
 subnet <local-network> <local-mask>

object network <remote-object>
 subnet <remote-network> <remote-mask>

nat (inside,outside) source static <local-object> <local-object> destination static <remote-object> <remote-object> no-proxy-arp route-lookup

access-list <crypto-acl> remark VPN interesting traffic from <local-network> to <remote-network>
access-list <crypto-acl> extended permit ip <local-network> <local-mask> <remote-network> <remote-mask>

crypto ipsec ikev1 transform-set <transform-set-name> esp-aes-256 esp-sha-hmac

crypto map <crypto-map-name> <seq> match address <crypto-acl>
crypto map <crypto-map-name> <seq> set peer <remote-peer-public-ip>
crypto map <crypto-map-name> <seq> set ikev1 transform-set <transform-set-name>
crypto map <crypto-map-name> interface outside

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

tunnel-group <remote-peer-public-ip> type ipsec-l2l
tunnel-group <remote-peer-public-ip> general-attributes
 default-group-policy <group-policy-name>
tunnel-group <remote-peer-public-ip> ipsec-attributes
 ikev1 pre-shared-key <pre-shared-key>

write memory
```

```
# ASA_Site_to_Site_IKEv1_IPsec_VPN_With_PFS_Skeleton
configure terminal

object network <local-object>
 subnet <local-network> <local-mask>

object network <remote-object>
 subnet <remote-network> <remote-mask>

nat (inside,outside) source static <local-object> <local-object> destination static <remote-object> <remote-object> no-proxy-arp route-lookup

access-list <crypto-acl> remark VPN interesting traffic from <local-network> to <remote-network>
access-list <crypto-acl> extended permit ip <local-network> <local-mask> <remote-network> <remote-mask>

crypto ipsec ikev1 transform-set <transform-set-name> esp-aes-256 esp-sha-hmac

crypto map <crypto-map-name> <seq> match address <crypto-acl>
crypto map <crypto-map-name> <seq> set peer <remote-peer-public-ip>
crypto map <crypto-map-name> <seq> set ikev1 transform-set <transform-set-name>
crypto map <crypto-map-name> <seq> set pfs group2
crypto map <crypto-map-name> interface outside

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

tunnel-group <remote-peer-public-ip> type ipsec-l2l
tunnel-group <remote-peer-public-ip> general-attributes
 default-group-policy <group-policy-name>
tunnel-group <remote-peer-public-ip> ipsec-attributes
 ikev1 pre-shared-key <pre-shared-key>

write memory
```


```
# ASA_Site_to_Site_IKEv1_IPsec_VPN_With_Outside_ACL_Inspection_Skeleton
configure terminal

no sysopt connection permit-vpn

access-list <outside-acl> extended permit ip <remote-network> <remote-mask> <local-network> <local-mask>
access-group <outside-acl> in interface outside

write memory
```
# ASA_Site_to_Site_IKEv1_IPsec_VPN_Verification_Commands
| Task                              | Command                                                                                          | Expected Result                                                          |
| --------------------------------- | ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------ |
| Verify interface state            | `show interface ip brief`                                                                        | Inside and outside interfaces are up/up                                  |
| Verify logical interface names    | `show nameif`                                                                                    | `inside` and `outside` are assigned correctly                            |
| Verify outside route to peer      | `show route <remote-peer-public-ip>`                                                             | ASA has route toward remote VPN peer                                     |
| Verify local protected object     | `show running-config object network <local-object>`                                              | Local network object has correct subnet                                  |
| Verify remote protected object    | `show running-config object network <remote-object>`                                             | Remote network object has correct subnet                                 |
| Verify NAT exemption              | `show nat`                                                                                       | Identity NAT appears before broad PAT                                    |
| Verify NAT exemption details      | `show nat detail`                                                                                | Rule shows local-to-local and remote-to-remote static mapping            |
| Verify crypto ACL                 | `show access-list <crypto-acl>`                                                                  | ACL permits local protected network to remote protected network          |
| Verify transform set              | `show running-config crypto ipsec`                                                               | IKEv1 transform set uses expected ESP encryption/hash                    |
| Verify crypto map                 | `show running-config crypto map`                                                                 | Crypto map has match address, peer, transform set, and outside interface |
| Verify IKEv1 enabled              | `show running-config crypto ikev1`                                                               | `crypto ikev1 enable outside` and policy are present                     |
| Verify tunnel group               | `show running-config tunnel-group <remote-peer-public-ip>`                                       | Peer tunnel group is `ipsec-l2l` and has IKEv1 preshared key configured  |
| Verify group policy               | `show running-config group-policy <group-policy-name>`                                           | Group policy allows `vpn-tunnel-protocol ikev1`                          |
| Verify default VPN ACL bypass     | `show run all sysopt`                                                                            | `sysopt connection permit-vpn` is enabled unless intentionally disabled  |
| Simulate ICMP interesting traffic | `packet-tracer input inside icmp <local-host-ip> 8 0 <remote-host-ip> detailed`                  | Packet matches NAT exemption and crypto policy                           |
| Simulate TCP interesting traffic  | `packet-tracer input inside tcp <local-host-ip> <src-port> <remote-host-ip> <dst-port> detailed` | Packet matches NAT exemption and crypto policy                           |
| Generate real traffic             | `ping <remote-host-ip>`                                                                          | Tunnel negotiation is triggered                                          |
| Verify IKEv1 Phase 1              | `show crypto ikev1 sa detail`                                                                    | State is `MM_ACTIVE`                                                     |
| Verify IPsec Phase 2              | `show crypto ipsec sa`                                                                           | Inbound and outbound ESP SAs exist                                       |
| Verify proxy identities           | `show crypto ipsec sa`                                                                           | Local and remote ident match crypto ACL networks                         |
| Verify outbound tunnel traffic    | `show crypto ipsec sa`                                                                           | Encaps/encrypt counters increment                                        |
| Verify inbound tunnel traffic     | `show crypto ipsec sa`                                                                           | Decaps/decrypt counters increment                                        |
| Verify session count              | `show vpn-sessiondb summary`                                                                     | Active IKEv1 IPsec site-to-site session appears                          |
| Verify connections                | `show conn`                                                                                      | Connection table shows local-to-remote flows                             |
| Verify NAT translations           | `show xlate`                                                                                     | VPN traffic is not PATed to outside interface                            |
| Verify drops                      | `show asp drop`                                                                                  | No relevant ACL, NAT, crypto, route, or inspection drops increment       |
| Capture ISAKMP if needed          | `capture <capture-name> type isakmp interface outside`                                           | ASA captures IKE/ISAKMP packets on outside                               |
| Read ISAKMP capture               | `show capture <capture-name> decode`                                                             | IKE packets are visible                                                  |
| Debug IKEv1 if needed             | `debug crypto ikev1 127`                                                                         | Debugs show Phase 1 negotiation status                                   |
| Debug IPsec if needed             | `debug crypto ipsec 127`                                                                         | Debugs show Phase 2/IPsec negotiation status                             |
| Stop debugs                       | `undebug all`                                                                                    | Debug output stops                                                       |
# ASA_Site_to_Site_IKEv1_IPsec_VPN_Rollback
| Step | Task                                                        | Device | Command                                                                                                                                            | Expected Result                                                        |
| ---: | ----------------------------------------------------------- | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
|    1 | Identify crypto map configuration                           | ASA    | `show running-config crypto map`                                                                                                                   | Crypto map name, sequence, ACL, peer, and transform set are identified |
|    2 | Identify IKEv1 policy                                       | ASA    | `show running-config crypto ikev1`                                                                                                                 | IKEv1 policy priority is identified                                    |
|    3 | Identify tunnel group                                       | ASA    | `show running-config tunnel-group <remote-peer-public-ip>`                                                                                         | Tunnel group and group policy are identified                           |
|    4 | Identify NAT exemption                                      | ASA    | `show nat detail`                                                                                                                                  | Exact identity NAT rule and objects are identified                     |
|    5 | Enter configuration mode                                    | ASA    | `configure terminal`                                                                                                                               | ASA enters global configuration mode                                   |
|    6 | Remove crypto map from outside interface                    | ASA    | `no crypto map <crypto-map-name> interface outside`                                                                                                | Crypto map is detached from outside interface                          |
|    7 | Remove crypto map peer line                                 | ASA    | `no crypto map <crypto-map-name> <seq> set peer <remote-peer-public-ip>`                                                                           | Peer is removed from crypto map                                        |
|    8 | Remove crypto map transform set line                        | ASA    | `no crypto map <crypto-map-name> <seq> set ikev1 transform-set <transform-set-name>`                                                               | Transform set binding is removed                                       |
|    9 | Remove crypto map PFS if configured                         | ASA    | `no crypto map <crypto-map-name> <seq> set pfs group2`                                                                                             | PFS setting is removed                                                 |
|   10 | Remove crypto map ACL match                                 | ASA    | `no crypto map <crypto-map-name> <seq> match address <crypto-acl>`                                                                                 | Crypto ACL binding is removed                                          |
|   11 | Remove crypto ACL                                           | ASA    | `clear configure access-list <crypto-acl>`                                                                                                         | Crypto ACL is removed                                                  |
|   12 | Remove NAT exemption                                        | ASA    | `no nat (inside,outside) source static <local-object> <local-object> destination static <remote-object> <remote-object> no-proxy-arp route-lookup` | Identity NAT rule is removed                                           |
|   13 | Remove transform set                                        | ASA    | `no crypto ipsec ikev1 transform-set <transform-set-name>`                                                                                         | IKEv1 transform set is removed                                         |
|   14 | Remove tunnel group                                         | ASA    | `clear configure tunnel-group <remote-peer-public-ip>`                                                                                             | L2L tunnel group is removed                                            |
|   15 | Remove group policy if unused                               | ASA    | `clear configure group-policy <group-policy-name>`                                                                                                 | Group policy is removed if not shared                                  |
|   16 | Remove IKEv1 policy                                         | ASA    | `clear configure crypto ikev1 policy <priority>`                                                                                                   | IKEv1 policy is removed                                                |
|   17 | Disable IKEv1 on outside only if no other IKEv1 VPNs use it | ASA    | `no crypto ikev1 enable outside`                                                                                                                   | ASA no longer listens for IKEv1 on outside                             |
|   18 | Remove outside ACL inspection rule if added for VPN only    | ASA    | `no access-list <outside-acl> extended permit ip <remote-network> <remote-mask> <local-network> <local-mask>`                                      | VPN-specific outside ACL permit is removed                             |
|   19 | Restore VPN ACL bypass if intentionally changed             | ASA    | `sysopt connection permit-vpn`                                                                                                                     | Decrypted VPN traffic again bypasses interface ACL checks              |
|   20 | Remove local object if unused                               | ASA    | `no object network <local-object>`                                                                                                                 | Local object is removed                                                |
|   21 | Remove remote object if unused                              | ASA    | `no object network <remote-object>`                                                                                                                | Remote object is removed                                               |
|   22 | Clear IKEv1 SAs                                             | ASA    | `clear crypto ikev1 sa`                                                                                                                            | IKEv1 Phase 1 state is cleared                                         |
|   23 | Clear IPsec SAs                                             | ASA    | `clear crypto ipsec sa`                                                                                                                            | IPsec Phase 2 state is cleared                                         |
|   24 | Clear stale translations                                    | ASA    | `clear xlate`                                                                                                                                      | Old NAT translations are cleared                                       |
|   25 | Clear stale connections                                     | ASA    | `clear conn address <local-host-ip>`                                                                                                               | Old connection state is cleared                                        |
|   26 | Verify rollback                                             | ASA    | `show running-config crypto`                                                                                                                       | Removed VPN crypto config no longer appears                            |
|   27 | Save rollback state                                         | ASA    | `write memory`                                                                                                                                     | Rollback is saved                                                      |

# ASA_Site_to_Site_IKEv1_IPsec_VPN_Failure_Checks
| Symptom                                                 | Command                                                    | What Usually Broke                                                                                 |
| ------------------------------------------------------- | ---------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| No IKEv1 SA appears                                     | `show crypto ikev1 sa detail`                              | IKEv1 not enabled on outside, peer unreachable, UDP/500 blocked, or wrong peer IP                  |
| Phase 1 never reaches `MM_ACTIVE`                       | `debug crypto ikev1 127`                                   | IKE policy mismatch or preshared-key mismatch                                                      |
| Debug says `All SA proposals found unacceptable`        | `debug crypto ikev1 127`                                   | Phase 1 mismatch: encryption, hash, DH group, authentication, or lifetime                          |
| Debug says decrypt failed or invalid payloads           | `debug crypto ikev1 127`                                   | Preshared key mismatch                                                                             |
| Phase 1 is up but Phase 2 fails                         | `show crypto ipsec sa` and `debug crypto ipsec 127`        | Transform set, PFS, crypto ACL, or proxy identity mismatch                                         |
| Debug says `All IPSec SA proposals found unacceptable`  | `debug crypto ipsec 127`                                   | Phase 2 transform set mismatch                                                                     |
| Debug says `ACL does not match proxy IDs`               | `debug crypto ikev1 127`                                   | Local/remote crypto ACL networks do not mirror the peer                                            |
| Encrypt counter increments but decrypt does not         | `show crypto ipsec sa`                                     | Return path missing, peer ACL mismatch, remote firewall issue, or remote NAT issue                 |
| Decrypt counter increments but local host gets no reply | `show conn`, `show asp drop`, host firewall checks         | ASA forwards decrypted traffic but local ACL, route, host firewall, or return path breaks          |
| Encaps/encrypt counters stay zero                       | `show crypto ipsec sa`                                     | No interesting traffic, route issue, crypto ACL mismatch, or traffic is NATed before encryption    |
| Tunnel comes up only after ping                         | `show crypto ikev1 sa detail`                              | Normal behavior for crypto map VPNs because interesting traffic usually triggers negotiation       |
| NAT exemption has zero hits                             | `show nat detail`                                          | Wrong local object, wrong remote object, wrong interface pair, or broad PAT matched first          |
| Traffic gets PATed instead of encrypted                 | `packet-tracer input inside ... detailed`                  | Missing NAT exemption or NAT exemption placed below broad PAT                                      |
| Crypto ACL hit count is zero                            | `show access-list <crypto-acl>`                            | Traffic does not match interesting traffic ACL                                                     |
| Outside ACL blocks decrypted VPN traffic                | `show run all sysopt` and `show access-list <outside-acl>` | `no sysopt connection permit-vpn` was configured but outside ACL does not permit decrypted traffic |
| Ping fails but TCP works                                | `packet-tracer input inside icmp ... detailed`             | ICMP blocked by host firewall, ACL, inspection, or remote device policy                            |
| TCP fails but ping works                                | `packet-tracer input inside tcp ... detailed`              | Service firewall, server listener, ACL, or application issue                                       |
| Tunnel fails after enabling PFS                         | `show running-config crypto map`                           | PFS group mismatch between peers                                                                   |
| Tunnel fails after changing crypto ACL                  | `show crypto ipsec sa`                                     | Proxy identities no longer match peer selectors                                                    |
| Remote peer shows up under wrong tunnel group           | `debug crypto ikev1 127`                                   | Tunnel group name does not match peer IP or peer appears behind NAT unexpectedly                   |
| ASA shows no route to peer public IP                    | `show route <remote-peer-public-ip>`                       | Missing default route or outside routing problem                                                   |
| ASA shows no route to remote LAN                        | `show route <remote-network-ip>`                           | Route/crypto map expectation wrong, missing static route, or packet path not hitting outside       |
| Packet-tracer allows but real traffic fails             | `show conn`, `show xlate`, captures, `show asp drop`       | Stale state, peer-side issue, host firewall, or asymmetric routing                                 |
| Debugs flood the terminal                               | `show debug`                                               | Debugs left enabled; run `undebug all`                                                             |
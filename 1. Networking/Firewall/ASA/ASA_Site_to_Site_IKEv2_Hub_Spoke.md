
```
# Index
# Source_Basis
# ASA_Site_to_Site_IKEv2_Hub_Spoke_Mental_Model
# ASA_Site_to_Site_IKEv2_Hub_Spoke_Configuration_Checklist
# ASA_Site_to_Site_IKEv2_Hub_Spoke_Hub_Skeleton
# ASA_Site_to_Site_IKEv2_Hub_Spoke_Additional_Spoke_On_Hub_Skeleton
# ASA_Site_to_Site_IKEv2_Hub_Spoke_Spoke_Skeleton
# ASA_Site_to_Site_IKEv2_Hub_Spoke_With_PFS_Skeleton
# ASA_Site_to_Site_IKEv2_Hub_Spoke_With_Outside_ACL_Inspection_Skeleton
# ASA_Site_to_Site_IKEv2_Hub_Spoke_Verification_Commands
# ASA_Site_to_Site_IKEv2_Hub_Spoke_Rollback
# ASA_Site_to_Site_IKEv2_Hub_Spoke_Failure_Checks
```

# ASA_Site_to_Site_IKEv2_Hub_Spoke_Mental_Model
| Concept                      | Operational Meaning                                                                                                                                                            |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Hub ASA                      | Central ASA terminates one site-to-site VPN per spoke                                                                                                                          |
| Spoke ASA                    | Branch ASA builds one site-to-site VPN back to the hub                                                                                                                         |
| Hub-and-spoke                | Spokes reach hub networks directly; spoke-to-spoke traffic only works if the hub is designed to route it and crypto/NAT/routing allow it                                       |
| Crypto map sequence          | Each spoke tunnel gets its own sequence number under the same outside crypto map                                                                                               |
| One crypto map per interface | ASA applies one crypto map name to outside, but that map can contain many sequence numbers                                                                                     |
| IKEv2 Phase 1                | Builds the IKE SA between hub and each spoke                                                                                                                                   |
| IKEv2 Phase 2                | Builds CHILD SAs for protected traffic between hub and each spoke                                                                                                              |
| IKEv2 policy                 | Must match peer on encryption, integrity, PRF, DH group, and lifetime                                                                                                          |
| IKEv2 IPsec proposal         | Defines ESP encryption and integrity for data traffic                                                                                                                          |
| Tunnel group                 | Per-peer object that stores L2L behavior and IKEv2 authentication keys                                                                                                         |
| Group policy                 | Restricts the tunnel protocol to IKEv2 for the L2L tunnel                                                                                                                      |
| Crypto ACL                   | Defines interesting traffic between hub local networks and each spoke network                                                                                                  |
| NAT exemption                | Prevents VPN traffic from being translated before encryption                                                                                                                   |
| Reverse Route Injection      | Optional `set reverse-route` installs routes from crypto ACL/proxy IDs                                                                                                         |
| NAT-T                        | Helps IPsec cross NAT devices, using keepalives                                                                                                                                |
| Proxy identity               | The local and remote protected networks negotiated for the CHILD SA                                                                                                            |
| Blunt rule                   | Hub-spoke ASA VPN is just multiple normal L2L tunnels sharing the hub outside crypto map. The complexity is rule order, route symmetry, NAT exemption, and crypto ACL symmetry |
# ASA_Site_to_Site_IKEv2_Hub_Spoke_Configuration_Checklist
| Step | Task                                                      | Device                 | Command                                                                                                                                             | Expected Result                                                                  |
| ---: | --------------------------------------------------------- | ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
|    1 | Confirm ASA interface status                              | Hub ASA                | `show interface ip brief`                                                                                                                           | Hub inside and outside interfaces are up/up                                      |
|    2 | Confirm ASA interface status                              | Each Spoke ASA         | `show interface ip brief`                                                                                                                           | Spoke inside and outside interfaces are up/up                                    |
|    3 | Confirm logical interface names                           | Hub ASA                | `show nameif`                                                                                                                                       | Hub interfaces are named correctly, usually `inside` and `outside`               |
|    4 | Confirm logical interface names                           | Each Spoke ASA         | `show nameif`                                                                                                                                       | Spoke interfaces are named correctly, usually `inside` and `outside`             |
|    5 | Confirm hub outside IP                                    | Hub ASA                | `show running-config interface <hub-outside-interface>`                                                                                             | Hub has stable outside VPN termination IP                                        |
|    6 | Confirm spoke outside IPs                                 | Each Spoke ASA         | `show running-config interface <spoke-outside-interface>`                                                                                           | Each spoke has expected outside IP or NATed public reachability                  |
|    7 | Confirm hub protected networks                            | Hub ASA / Notes        | `<hub-network-list>`                                                                                                                                | Hub-side protected networks are known                                            |
|    8 | Confirm spoke protected networks                          | Each Spoke ASA / Notes | `<spoke-network-list>`                                                                                                                              | Each spoke-side protected network is known                                       |
|    9 | Confirm peer public IP mapping                            | Notes                  | `<hub-public-ip> <spoke-public-ip>`                                                                                                                 | Each ASA knows the peer public IP it must use                                    |
|   10 | Confirm IKEv2 Phase 1 parameters                          | Notes                  | `encryption <ike-encryption>, integrity <ike-integrity>, group <dh-group>, prf <prf>, lifetime <seconds>`                                           | Hub and all spokes agree on IKEv2 policy                                         |
|   11 | Confirm IPsec Phase 2 parameters                          | Notes                  | `esp encryption <esp-encryption>, esp integrity <esp-integrity>`                                                                                    | Hub and all spokes agree on IKEv2 IPsec proposal                                 |
|   12 | Confirm preshared keys per spoke                          | Notes                  | `<hub-local-key> / <spoke-remote-key>`                                                                                                              | Hub and each spoke have matching local/remote IKEv2 keys                         |
|   13 | Confirm hub can route to each spoke peer IP               | Hub ASA                | `show route <spoke-public-ip>`                                                                                                                      | Hub has outside route to each spoke public IP                                    |
|   14 | Confirm each spoke can route to hub peer IP               | Each Spoke ASA         | `show route <hub-public-ip>`                                                                                                                        | Each spoke has outside route to hub public IP                                    |
|   15 | Review existing hub crypto maps                           | Hub ASA                | `show running-config crypto map`                                                                                                                    | Existing crypto map names and sequence numbers are known                         |
|   16 | Review existing spoke crypto maps                         | Each Spoke ASA         | `show running-config crypto map`                                                                                                                    | Existing crypto map names and sequence numbers are known                         |
|   17 | Review existing NAT                                       | Hub ASA                | `show nat detail`                                                                                                                                   | Existing hub NAT and broad PAT rules are visible                                 |
|   18 | Review existing NAT                                       | Each Spoke ASA         | `show nat detail`                                                                                                                                   | Existing spoke NAT and broad PAT rules are visible                               |
|   19 | Enter configuration mode                                  | Hub ASA                | `configure terminal`                                                                                                                                | Hub ASA enters global configuration mode                                         |
|   20 | Create hub protected object or object-group               | Hub ASA                | `object-group network <hub-net-og>`                                                                                                                 | Hub network group is created                                                     |
|   21 | Add hub networks                                          | Hub ASA                | `network-object <hub-network> <hub-mask>`                                                                                                           | Hub protected networks are added                                                 |
|   22 | Create spoke protected object or object-group             | Hub ASA                | `object-group network <spoke-name-net-og>`                                                                                                          | Spoke network group is created on hub                                            |
|   23 | Add spoke networks                                        | Hub ASA                | `network-object <spoke-network> <spoke-mask>`                                                                                                       | Spoke protected networks are added                                               |
|   24 | Configure hub NAT exemption for spoke                     | Hub ASA                | `nat (inside,outside) source static <hub-net-og> <hub-net-og> destination static <spoke-name-net-og> <spoke-name-net-og> no-proxy-arp route-lookup` | Hub-to-spoke traffic is not translated                                           |
|   25 | Create hub crypto ACL remark                              | Hub ASA                | `access-list <spoke-crypto-acl> remark HUB to <spoke-name> IKEv2 VPN traffic`                                                                       | Crypto ACL purpose is documented                                                 |
|   26 | Create hub crypto ACL                                     | Hub ASA                | `access-list <spoke-crypto-acl> extended permit ip object-group <hub-net-og> object-group <spoke-name-net-og>`                                      | Hub interesting traffic for that spoke is defined                                |
|   27 | Create IKEv2 IPsec proposal                               | Hub ASA                | `crypto ipsec ikev2 ipsec-proposal <ipsec-proposal-name>`                                                                                           | Hub enters IPsec proposal mode                                                   |
|   28 | Set ESP encryption                                        | Hub ASA                | `protocol esp encryption <esp-encryption>`                                                                                                          | Data-plane encryption is configured                                              |
|   29 | Set ESP integrity                                         | Hub ASA                | `protocol esp integrity <esp-integrity>`                                                                                                            | Data-plane integrity is configured                                               |
|   30 | Create hub crypto map sequence for spoke                  | Hub ASA                | `crypto map <outside-crypto-map-name> <spoke-seq> match address <spoke-crypto-acl>`                                                                 | Crypto map sequence matches the spoke crypto ACL                                 |
|   31 | Set spoke peer IP                                         | Hub ASA                | `crypto map <outside-crypto-map-name> <spoke-seq> set peer <spoke-public-ip>`                                                                       | Hub crypto map points to spoke peer                                              |
|   32 | Attach IKEv2 IPsec proposal                               | Hub ASA                | `crypto map <outside-crypto-map-name> <spoke-seq> set ikev2 ipsec-proposal <ipsec-proposal-name>`                                                   | Crypto map uses intended IKEv2 IPsec proposal                                    |
|   33 | Enable RRI if hub should install spoke routes dynamically | Hub ASA                | `crypto map <outside-crypto-map-name> <spoke-seq> set reverse-route`                                                                                | Hub can install routes from negotiated proxy IDs                                 |
|   34 | Enable PFS only if all matching spokes require it         | Hub ASA                | `crypto map <outside-crypto-map-name> <spoke-seq> set pfs <pfs-group>`                                                                              | PFS is enabled only if peer matches                                              |
|   35 | Apply crypto map to outside                               | Hub ASA                | `crypto map <outside-crypto-map-name> interface outside`                                                                                            | Hub crypto map is active on outside                                              |
|   36 | Enable IKEv2 on outside                                   | Hub ASA                | `crypto ikev2 enable outside`                                                                                                                       | Hub listens for IKEv2 on outside                                                 |
|   37 | Create IKEv2 policy                                       | Hub ASA                | `crypto ikev2 policy <priority>`                                                                                                                    | Hub enters IKEv2 policy mode                                                     |
|   38 | Set IKEv2 encryption                                      | Hub ASA                | `encryption <ike-encryption>`                                                                                                                       | IKEv2 encryption is configured                                                   |
|   39 | Set IKEv2 integrity                                       | Hub ASA                | `integrity <ike-integrity>`                                                                                                                         | IKEv2 integrity is configured                                                    |
|   40 | Set IKEv2 DH group                                        | Hub ASA                | `group <dh-group-list>`                                                                                                                             | IKEv2 DH group list is configured                                                |
|   41 | Set IKEv2 PRF                                             | Hub ASA                | `prf <prf-algorithm>`                                                                                                                               | IKEv2 PRF is configured                                                          |
|   42 | Set IKEv2 lifetime                                        | Hub ASA                | `lifetime seconds <seconds>`                                                                                                                        | IKEv2 lifetime is configured                                                     |
|   43 | Create hub group policy                                   | Hub ASA                | `group-policy <group-policy-name> internal`                                                                                                         | Hub group policy exists                                                          |
|   44 | Enter hub group policy attributes                         | Hub ASA                | `group-policy <group-policy-name> attributes`                                                                                                       | Hub enters group policy attributes                                               |
|   45 | Restrict tunnel protocol to IKEv2                         | Hub ASA                | `vpn-tunnel-protocol ikev2`                                                                                                                         | Group policy allows IKEv2                                                        |
|   46 | Create hub tunnel group for spoke                         | Hub ASA                | `tunnel-group <spoke-public-ip> type ipsec-l2l`                                                                                                     | Peer-specific L2L tunnel group exists                                            |
|   47 | Apply group policy to spoke tunnel group                  | Hub ASA                | `tunnel-group <spoke-public-ip> general-attributes` then `default-group-policy <group-policy-name>`                                                 | Spoke tunnel group uses intended group policy                                    |
|   48 | Enter spoke tunnel group IPsec attributes                 | Hub ASA                | `tunnel-group <spoke-public-ip> ipsec-attributes`                                                                                                   | Hub enters IPsec attributes for that spoke                                       |
|   49 | Configure IKEv2 remote authentication key                 | Hub ASA                | `ikev2 remote-authentication pre-shared-key <spoke-key-presented-to-hub>`                                                                           | Hub can authenticate the spoke                                                   |
|   50 | Configure IKEv2 local authentication key                  | Hub ASA                | `ikev2 local-authentication pre-shared-key <hub-key-presented-to-spoke>`                                                                            | Hub presents the expected local key                                              |
|   51 | Repeat hub steps per spoke                                | Hub ASA                | `<repeat NAT exemption, crypto ACL, crypto map seq, tunnel group>`                                                                                  | Hub has one tunnel definition per spoke                                          |
|   52 | Save hub configuration                                    | Hub ASA                | `write memory`                                                                                                                                      | Hub configuration is saved                                                       |
|   53 | Enter configuration mode on spoke                         | Each Spoke ASA         | `configure terminal`                                                                                                                                | Spoke ASA enters global configuration mode                                       |
|   54 | Create spoke protected object or object-group             | Each Spoke ASA         | `object-group network <spoke-net-og>`                                                                                                               | Spoke protected networks are grouped                                             |
|   55 | Add spoke protected networks                              | Each Spoke ASA         | `network-object <spoke-network> <spoke-mask>`                                                                                                       | Spoke local networks are added                                                   |
|   56 | Create hub protected object or object-group               | Each Spoke ASA         | `object-group network <hub-net-og>`                                                                                                                 | Hub protected networks are grouped on spoke                                      |
|   57 | Add hub protected networks                                | Each Spoke ASA         | `network-object <hub-network> <hub-mask>`                                                                                                           | Hub networks are added                                                           |
|   58 | Configure spoke NAT exemption                             | Each Spoke ASA         | `nat (inside,outside) source static <spoke-net-og> <spoke-net-og> destination static <hub-net-og> <hub-net-og> no-proxy-arp route-lookup`           | Spoke-to-hub VPN traffic is not translated                                       |
|   59 | Create spoke crypto ACL remark                            | Each Spoke ASA         | `access-list <hub-crypto-acl> remark SPOKE to HUB IKEv2 VPN traffic`                                                                                | Crypto ACL purpose is documented                                                 |
|   60 | Create spoke crypto ACL                                   | Each Spoke ASA         | `access-list <hub-crypto-acl> extended permit ip object-group <spoke-net-og> object-group <hub-net-og>`                                             | Spoke interesting traffic to hub is defined                                      |
|   61 | Create spoke IKEv2 IPsec proposal                         | Each Spoke ASA         | `crypto ipsec ikev2 ipsec-proposal <ipsec-proposal-name>`                                                                                           | Spoke enters IPsec proposal mode                                                 |
|   62 | Set spoke ESP encryption                                  | Each Spoke ASA         | `protocol esp encryption <esp-encryption>`                                                                                                          | Spoke data-plane encryption matches hub                                          |
|   63 | Set spoke ESP integrity                                   | Each Spoke ASA         | `protocol esp integrity <esp-integrity>`                                                                                                            | Spoke data-plane integrity matches hub                                           |
|   64 | Create spoke crypto map sequence                          | Each Spoke ASA         | `crypto map <outside-crypto-map-name> <seq> match address <hub-crypto-acl>`                                                                         | Spoke crypto map references hub crypto ACL                                       |
|   65 | Set hub peer IP                                           | Each Spoke ASA         | `crypto map <outside-crypto-map-name> <seq> set peer <hub-public-ip>`                                                                               | Spoke points to hub peer                                                         |
|   66 | Attach IKEv2 IPsec proposal                               | Each Spoke ASA         | `crypto map <outside-crypto-map-name> <seq> set ikev2 ipsec-proposal <ipsec-proposal-name>`                                                         | Spoke crypto map uses intended proposal                                          |
|   67 | Enable RRI on spoke only if desired                       | Each Spoke ASA         | `crypto map <outside-crypto-map-name> <seq> set reverse-route`                                                                                      | Spoke can install routes from negotiated proxy IDs                               |
|   68 | Enable PFS only if hub requires it                        | Each Spoke ASA         | `crypto map <outside-crypto-map-name> <seq> set pfs <pfs-group>`                                                                                    | Spoke PFS matches hub                                                            |
|   69 | Apply crypto map to outside                               | Each Spoke ASA         | `crypto map <outside-crypto-map-name> interface outside`                                                                                            | Spoke crypto map is active on outside                                            |
|   70 | Enable IKEv2 on outside                                   | Each Spoke ASA         | `crypto ikev2 enable outside`                                                                                                                       | Spoke listens for IKEv2 on outside                                               |
|   71 | Create spoke IKEv2 policy                                 | Each Spoke ASA         | `crypto ikev2 policy <priority>`                                                                                                                    | Spoke enters IKEv2 policy mode                                                   |
|   72 | Set spoke IKEv2 encryption                                | Each Spoke ASA         | `encryption <ike-encryption>`                                                                                                                       | IKEv2 encryption matches hub                                                     |
|   73 | Set spoke IKEv2 integrity                                 | Each Spoke ASA         | `integrity <ike-integrity>`                                                                                                                         | IKEv2 integrity matches hub                                                      |
|   74 | Set spoke IKEv2 DH group                                  | Each Spoke ASA         | `group <dh-group-list>`                                                                                                                             | IKEv2 DH group matches hub                                                       |
|   75 | Set spoke IKEv2 PRF                                       | Each Spoke ASA         | `prf <prf-algorithm>`                                                                                                                               | IKEv2 PRF matches hub                                                            |
|   76 | Set spoke IKEv2 lifetime                                  | Each Spoke ASA         | `lifetime seconds <seconds>`                                                                                                                        | IKEv2 lifetime matches hub or is acceptable                                      |
|   77 | Create spoke group policy                                 | Each Spoke ASA         | `group-policy <group-policy-name> internal`                                                                                                         | Spoke group policy exists                                                        |
|   78 | Enter spoke group policy attributes                       | Each Spoke ASA         | `group-policy <group-policy-name> attributes`                                                                                                       | Spoke enters group policy attributes                                             |
|   79 | Restrict spoke tunnel protocol to IKEv2                   | Each Spoke ASA         | `vpn-tunnel-protocol ikev2`                                                                                                                         | Spoke group policy allows IKEv2                                                  |
|   80 | Create spoke tunnel group for hub                         | Each Spoke ASA         | `tunnel-group <hub-public-ip> type ipsec-l2l`                                                                                                       | Peer-specific hub tunnel group exists                                            |
|   81 | Apply group policy to hub tunnel group                    | Each Spoke ASA         | `tunnel-group <hub-public-ip> general-attributes` then `default-group-policy <group-policy-name>`                                                   | Hub tunnel group uses intended group policy                                      |
|   82 | Enter hub tunnel group IPsec attributes                   | Each Spoke ASA         | `tunnel-group <hub-public-ip> ipsec-attributes`                                                                                                     | Spoke enters IPsec attributes for hub peer                                       |
|   83 | Configure IKEv2 remote authentication key                 | Each Spoke ASA         | `ikev2 remote-authentication pre-shared-key <hub-key-presented-to-spoke>`                                                                           | Spoke can authenticate the hub                                                   |
|   84 | Configure IKEv2 local authentication key                  | Each Spoke ASA         | `ikev2 local-authentication pre-shared-key <spoke-key-presented-to-hub>`                                                                            | Spoke presents expected local key                                                |
|   85 | Enable NAT-T keepalive if required by design              | Hub and Spokes         | `crypto isakmp nat-traversal <seconds>`                                                                                                             | NAT-T keepalive behavior is configured                                           |
|   86 | Confirm default VPN ACL bypass behavior                   | Hub and Spokes         | `show run all sysopt`                                                                                                                               | `sysopt connection permit-vpn` is enabled unless intentionally changed           |
|   87 | Configure outside ACL only if VPN ACL bypass is disabled  | Hub and Spokes         | `access-list <outside-acl> extended permit ip <remote-vpn-network> <remote-mask> <local-vpn-network> <local-mask>`                                  | Decrypted VPN traffic is permitted if `no sysopt connection permit-vpn` is used  |
|   88 | Save spoke configuration                                  | Each Spoke ASA         | `write memory`                                                                                                                                      | Spoke configuration is saved                                                     |
|   89 | Trigger spoke-to-hub traffic                              | Spoke Host / ASA       | `ping <hub-host-ip>`                                                                                                                                | Interesting traffic triggers IKEv2 negotiation                                   |
|   90 | Verify hub IKEv2 SAs                                      | Hub ASA                | `show crypto ikev2 sa`                                                                                                                              | Hub shows one IKEv2 SA per active spoke                                          |
|   91 | Verify spoke IKEv2 SA                                     | Each Spoke ASA         | `show crypto ikev2 sa`                                                                                                                              | Spoke shows IKEv2 SA to hub                                                      |
|   92 | Verify hub IPsec SAs                                      | Hub ASA                | `show crypto ipsec sa`                                                                                                                              | Hub shows CHILD SAs and proxy identities per spoke                               |
|   93 | Verify spoke IPsec SA                                     | Each Spoke ASA         | `show crypto ipsec sa`                                                                                                                              | Spoke shows CHILD SA to hub                                                      |
|   94 | Verify encrypt/decrypt counters                           | Hub and Spokes         | `show crypto ipsec sa`                                                                                                                              | Encaps/encrypt and decaps/decrypt counters increment                             |
|   95 | Verify NAT exemption hits                                 | Hub and Spokes         | `show nat detail`                                                                                                                                   | Identity NAT hit counts increment                                                |
|   96 | Verify crypto ACL hits                                    | Hub and Spokes         | `show access-list <crypto-acl>`                                                                                                                     | Crypto ACL hit counts increment                                                  |
|   97 | Verify RRI routes if configured                           | Hub and Spokes         | `show route`                                                                                                                                        | Reverse routes appear if `set reverse-route` is configured and tunnel is active  |
|   98 | Verify VPN sessions                                       | Hub and Spokes         | `show vpn-sessiondb summary`                                                                                                                        | Active IKEv2/IPsec site-to-site sessions appear                                  |
|   99 | Check drops if traffic fails                              | Hub and Spokes         | `show asp drop`                                                                                                                                     | Drop reason points to NAT, ACL, crypto ACL, route, proposal, or inspection issue |
|  100 | Debug only when needed                                    | Hub and Spokes         | `debug crypto ikev2 protocol 2` and `debug crypto ikev2 platform 2`                                                                                 | Debug shows IKEv2 negotiation or authentication failure                          |
|  101 | Debug IPsec only when needed                              | Hub and Spokes         | `debug crypto ipsec 127`                                                                                                                            | Debug shows CHILD SA/proxy identity behavior                                     |
|  102 | Stop debugs                                               | Hub and Spokes         | `undebug all`                                                                                                                                       | Debug output stops                                                               |


```
# ASA_Site_to_Site_IKEv2_Hub_Spoke_Hub_Skeleton
configure terminal

object-group network <hub-net-og>
 network-object <hub-network-1> <hub-mask-1>
 network-object <hub-network-2> <hub-mask-2>

object-group network <spoke1-net-og>
 network-object <spoke1-network-1> <spoke1-mask-1>

nat (inside,outside) source static <hub-net-og> <hub-net-og> destination static <spoke1-net-og> <spoke1-net-og> no-proxy-arp route-lookup

access-list <spoke1-crypto-acl> remark HUB to SPOKE1 IKEv2 VPN traffic
access-list <spoke1-crypto-acl> extended permit ip object-group <hub-net-og> object-group <spoke1-net-og>

crypto ipsec ikev2 ipsec-proposal <ipsec-proposal-name>
 protocol esp encryption <esp-encryption>
 protocol esp integrity <esp-integrity>

crypto map <outside-crypto-map-name> <spoke1-seq> match address <spoke1-crypto-acl>
crypto map <outside-crypto-map-name> <spoke1-seq> set peer <spoke1-public-ip>
crypto map <outside-crypto-map-name> <spoke1-seq> set ikev2 ipsec-proposal <ipsec-proposal-name>
crypto map <outside-crypto-map-name> <spoke1-seq> set reverse-route
crypto map <outside-crypto-map-name> interface outside

crypto ikev2 enable outside
crypto ikev2 policy <priority>
 encryption <ike-encryption>
 integrity <ike-integrity>
 group <dh-group-list>
 prf <prf-algorithm>
 lifetime seconds <seconds>

crypto isakmp nat-traversal <seconds>

group-policy <group-policy-name> internal
group-policy <group-policy-name> attributes
 vpn-tunnel-protocol ikev2

tunnel-group <spoke1-public-ip> type ipsec-l2l
tunnel-group <spoke1-public-ip> general-attributes
 default-group-policy <group-policy-name>
tunnel-group <spoke1-public-ip> ipsec-attributes
 ikev2 remote-authentication pre-shared-key <spoke1-key-presented-to-hub>
 ikev2 local-authentication pre-shared-key <hub-key-presented-to-spoke1>

write memory
```

```
# ASA_Site_to_Site_IKEv2_Hub_Spoke_Additional_Spoke_On_Hub_Skeleton
configure terminal

object-group network <spokeN-net-og>
 network-object <spokeN-network-1> <spokeN-mask-1>

nat (inside,outside) source static <hub-net-og> <hub-net-og> destination static <spokeN-net-og> <spokeN-net-og> no-proxy-arp route-lookup

access-list <spokeN-crypto-acl> remark HUB to SPOKEN IKEv2 VPN traffic
access-list <spokeN-crypto-acl> extended permit ip object-group <hub-net-og> object-group <spokeN-net-og>

crypto map <outside-crypto-map-name> <spokeN-seq> match address <spokeN-crypto-acl>
crypto map <outside-crypto-map-name> <spokeN-seq> set peer <spokeN-public-ip>
crypto map <outside-crypto-map-name> <spokeN-seq> set ikev2 ipsec-proposal <ipsec-proposal-name>
crypto map <outside-crypto-map-name> <spokeN-seq> set reverse-route

tunnel-group <spokeN-public-ip> type ipsec-l2l
tunnel-group <spokeN-public-ip> general-attributes
 default-group-policy <group-policy-name>
tunnel-group <spokeN-public-ip> ipsec-attributes
 ikev2 remote-authentication pre-shared-key <spokeN-key-presented-to-hub>
 ikev2 local-authentication pre-shared-key <hub-key-presented-to-spokeN>

write memory
```


```
# ASA_Site_to_Site_IKEv2_Hub_Spoke_Spoke_Skeleton
configure terminal

object-group network <spoke-net-og>
 network-object <spoke-network-1> <spoke-mask-1>

object-group network <hub-net-og>
 network-object <hub-network-1> <hub-mask-1>
 network-object <hub-network-2> <hub-mask-2>

nat (inside,outside) source static <spoke-net-og> <spoke-net-og> destination static <hub-net-og> <hub-net-og> no-proxy-arp route-lookup

access-list <hub-crypto-acl> remark SPOKE to HUB IKEv2 VPN traffic
access-list <hub-crypto-acl> extended permit ip object-group <spoke-net-og> object-group <hub-net-og>

crypto ipsec ikev2 ipsec-proposal <ipsec-proposal-name>
 protocol esp encryption <esp-encryption>
 protocol esp integrity <esp-integrity>

crypto map <outside-crypto-map-name> <seq> match address <hub-crypto-acl>
crypto map <outside-crypto-map-name> <seq> set peer <hub-public-ip>
crypto map <outside-crypto-map-name> <seq> set ikev2 ipsec-proposal <ipsec-proposal-name>
crypto map <outside-crypto-map-name> interface outside

crypto ikev2 enable outside
crypto ikev2 policy <priority>
 encryption <ike-encryption>
 integrity <ike-integrity>
 group <dh-group-list>
 prf <prf-algorithm>
 lifetime seconds <seconds>

crypto isakmp nat-traversal <seconds>

group-policy <group-policy-name> internal
group-policy <group-policy-name> attributes
 vpn-tunnel-protocol ikev2

tunnel-group <hub-public-ip> type ipsec-l2l
tunnel-group <hub-public-ip> general-attributes
 default-group-policy <group-policy-name>
tunnel-group <hub-public-ip> ipsec-attributes
 ikev2 remote-authentication pre-shared-key <hub-key-presented-to-spoke>
 ikev2 local-authentication pre-shared-key <spoke-key-presented-to-hub>

write memory
```

```
# ASA_Site_to_Site_IKEv2_Hub_Spoke_With_PFS_Skeleton
configure terminal

crypto map <outside-crypto-map-name> <seq> set pfs <pfs-group>

write memory
```

```
# ASA_Site_to_Site_IKEv2_Hub_Spoke_With_Outside_ACL_Inspection_Skeleton
configure terminal

no sysopt connection permit-vpn

access-list <outside-acl> extended permit ip <remote-vpn-network> <remote-mask> <local-vpn-network> <local-mask>
access-group <outside-acl> in interface outside

write memory
```

# ASA_Site_to_Site_IKEv2_Hub_Spoke_Verification_Commands
| Task                              | Command                                                                                          | Expected Result                                                             |
| --------------------------------- | ------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------- |
| Verify interface state            | `show interface ip brief`                                                                        | Inside and outside interfaces are up/up                                     |
| Verify interface names            | `show nameif`                                                                                    | Interface names match NAT and crypto map references                         |
| Verify route to peer public IP    | `show route <peer-public-ip>`                                                                    | ASA has outside route to peer                                               |
| Verify hub network object-group   | `show running-config object-group network <hub-net-og>`                                          | Hub protected networks are correct                                          |
| Verify spoke network object-group | `show running-config object-group network <spoke-net-og>`                                        | Spoke protected networks are correct                                        |
| Verify NAT exemption              | `show nat`                                                                                       | Identity NAT appears before broad PAT                                       |
| Verify NAT details                | `show nat detail`                                                                                | Identity NAT shows correct source and destination objects                   |
| Verify crypto ACL                 | `show access-list <crypto-acl>`                                                                  | ACL permits local protected networks to remote protected networks           |
| Verify IKEv2 IPsec proposal       | `show running-config crypto ipsec`                                                               | Proposal has expected ESP encryption and integrity                          |
| Verify crypto map                 | `show running-config crypto map`                                                                 | Crypto map has ACL, peer, proposal, optional RRI/PFS, and outside interface |
| Verify IKEv2 enabled              | `show running-config crypto ikev2`                                                               | IKEv2 is enabled on outside and policy exists                               |
| Verify tunnel group               | `show running-config tunnel-group <peer-public-ip>`                                              | Peer tunnel group is `ipsec-l2l` and has IKEv2 local/remote keys            |
| Verify group policy               | `show running-config group-policy <group-policy-name>`                                           | Group policy permits `vpn-tunnel-protocol ikev2`                            |
| Verify NAT-T                      | `show running-config crypto isakmp`                                                              | NAT-T keepalive appears if configured                                       |
| Verify default VPN ACL bypass     | `show run all sysopt`                                                                            | `sysopt connection permit-vpn` is enabled unless intentionally disabled     |
| Simulate ICMP interesting traffic | `packet-tracer input inside icmp <local-host-ip> 8 0 <remote-host-ip> detailed`                  | Flow matches NAT exemption and crypto policy                                |
| Simulate TCP interesting traffic  | `packet-tracer input inside tcp <local-host-ip> <src-port> <remote-host-ip> <dst-port> detailed` | Flow matches NAT exemption and crypto policy                                |
| Trigger real traffic              | `ping <remote-host-ip>`                                                                          | IKEv2/IPsec negotiation starts if tunnel is idle                            |
| Verify IKEv2 SA                   | `show crypto ikev2 sa`                                                                           | IKEv2 SA is established with peer                                           |
| Verify IPsec SA                   | `show crypto ipsec sa`                                                                           | Inbound and outbound CHILD SAs exist                                        |
| Verify proxy identities           | `show crypto ipsec sa`                                                                           | Local and remote identities match crypto ACL networks                       |
| Verify outbound encryption        | `show crypto ipsec sa`                                                                           | Encaps/encrypt counters increment                                           |
| Verify inbound decryption         | `show crypto ipsec sa`                                                                           | Decaps/decrypt counters increment                                           |
| Verify RRI routes                 | `show route`                                                                                     | Reverse routes appear if `set reverse-route` is configured                  |
| Verify VPN session summary        | `show vpn-sessiondb summary`                                                                     | Active IKEv2 IPsec site-to-site sessions appear                             |
| Verify connections                | `show conn`                                                                                      | Local-to-remote VPN flows appear                                            |
| Verify translations               | `show xlate`                                                                                     | VPN traffic is not PATed to outside interface                               |
| Verify drops                      | `show asp drop`                                                                                  | No relevant NAT, ACL, crypto, route, or inspection drops increment          |
| Debug IKEv2 protocol if needed    | `debug crypto ikev2 protocol 2`                                                                  | Debug shows IKEv2 negotiation behavior                                      |
| Debug IKEv2 platform if needed    | `debug crypto ikev2 platform 2`                                                                  | Debug shows platform/authentication details                                 |
| Debug IPsec if needed             | `debug crypto ipsec 127`                                                                         | Debug shows CHILD SA and proxy identity behavior                            |
| Stop debugs                       | `undebug all`                                                                                    | Debug output stops                                                          |
# ASA_Site_to_Site_IKEv2_Hub_Spoke_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify crypto map entries | ASA | `show running-config crypto map` | Crypto map name, sequence numbers, ACLs, peers, and proposals are identified |
| 2 | Identify IKEv2 policy | ASA | `show running-config crypto ikev2` | IKEv2 policy priority and parameters are identified |
| 3 | Identify tunnel groups | ASA | `show running-config tunnel-group` | Peer tunnel groups and keys are identified |
| 4 | Identify NAT exemption rules | ASA | `show nat detail` | Identity NAT rules and objects are identified |
| 5 | Enter configuration mode | ASA | `configure terminal` | ASA enters global configuration mode |
| 6 | Remove crypto map from outside interface if removing all tunnels | ASA | `no crypto map <outside-crypto-map-name> interface outside` | Crypto map is detached from outside |
| 7 | Remove peer from crypto map sequence | ASA | `no crypto map <outside-crypto-map-name> <seq> set peer <peer-public-ip>` | Peer binding is removed |
| 8 | Remove IKEv2 proposal from crypto map sequence | ASA | `no crypto map <outside-crypto-map-name> <seq> set ikev2 ipsec-proposal <ipsec-proposal-name>` | Proposal binding is removed |
| 9 | Remove RRI if configured | ASA | `no crypto map <outside-crypto-map-name> <seq> set reverse-route` | Reverse-route behavior is removed |
| 10 | Remove PFS if configured | ASA | `no crypto map <outside-crypto-map-name> <seq> set pfs <pfs-group>` | PFS is removed |
| 11 | Remove crypto ACL match from crypto map | ASA | `no crypto map <outside-crypto-map-name> <seq> match address <crypto-acl>` | Crypto ACL binding is removed |
| 12 | Remove crypto ACL | ASA | `clear configure access-list <crypto-acl>` | Crypto ACL is removed |
| 13 | Remove NAT exemption | ASA | `no nat (inside,outside) source static <local-og> <local-og> destination static <remote-og> <remote-og> no-proxy-arp route-lookup` | Identity NAT is removed |
| 14 | Remove tunnel group | ASA | `clear configure tunnel-group <peer-public-ip>` | Peer-specific L2L tunnel group is removed |
| 15 | Remove group policy if unused | ASA | `clear configure group-policy <group-policy-name>` | Group policy is removed if not shared |
| 16 | Remove IKEv2 IPsec proposal if unused | ASA | `no crypto ipsec ikev2 ipsec-proposal <ipsec-proposal-name>` | IPsec proposal is removed |
| 17 | Remove IKEv2 policy if unused | ASA | `clear configure crypto ikev2 policy <priority>` | IKEv2 policy is removed |
| 18 | Disable IKEv2 on outside only if no other IKEv2 VPNs use it | ASA | `no crypto ikev2 enable outside` | ASA stops listening for IKEv2 on outside |
| 19 | Remove outside ACL permit if added only for this VPN | ASA | `no access-list <outside-acl> extended permit ip <remote-vpn-network> <remote-mask> <local-vpn-network> <local-mask>` | VPN-specific ACL entry is removed |
| 20 | Restore VPN ACL bypass if changed | ASA | `sysopt connection permit-vpn` | Decrypted VPN traffic bypasses interface ACLs again |
| 21 | Remove local object-group if unused | ASA | `no object-group network <local-og>` | Local object-group is removed |
| 22 | Remove remote object-group if unused | ASA | `no object-group network <remote-og>` | Remote object-group is removed |
| 23 | Clear IKEv2 SAs | ASA | `clear crypto ikev2 sa` | IKEv2 control SAs are cleared |
| 24 | Clear IPsec SAs | ASA | `clear crypto ipsec sa` | IPsec CHILD SAs are cleared |
| 25 | Clear stale translations | ASA | `clear xlate` | Old NAT translations are cleared |
| 26 | Clear stale connections | ASA | `clear conn address <local-host-ip>` | Old connection state is cleared |
| 27 | Verify rollback | ASA | `show running-config crypto` | Removed hub/spoke crypto config no longer appears |
| 28 | Save rollback state | ASA | `write memory` | Rollback is saved |
# ASA_Site_to_Site_IKEv2_Hub_Spoke_Failure_Checks
| Symptom                                                 | Command                                                    | What Usually Broke                                                                                                                                                    |
| ------------------------------------------------------- | ---------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| IKEv2 SA does not appear                                | `show crypto ikev2 sa`                                     | IKEv2 not enabled, peer unreachable, UDP/500 blocked, NAT-T issue, or wrong peer IP                                                                                   |
| IKEv2 negotiation starts but fails                      | `debug crypto ikev2 protocol 2`                            | IKEv2 policy mismatch, authentication key mismatch, or unsupported proposal                                                                                           |
| Debug shows authentication failure                      | `debug crypto ikev2 protocol 2`                            | IKEv2 local/remote preshared keys are reversed or mismatched                                                                                                          |
| Phase 1 is up but IPsec SA fails                        | `show crypto ipsec sa` and `debug crypto ipsec 127`        | IPsec proposal, PFS, crypto ACL, or proxy identity mismatch                                                                                                           |
| Proxy identities do not match                           | `show crypto ipsec sa`                                     | Hub and spoke crypto ACLs are not mirror images                                                                                                                       |
| NAT exemption has zero hits                             | `show nat detail`                                          | Wrong local object, wrong remote object, wrong interface pair, or traffic is not matching                                                                             |
| Traffic gets PATed instead of encrypted                 | `packet-tracer input inside ... detailed`                  | NAT exemption missing or placed below broad Dynamic PAT                                                                                                               |
| Crypto ACL hit count is zero                            | `show access-list <crypto-acl>`                            | No interesting traffic or crypto ACL source/destination is wrong                                                                                                      |
| One spoke works and another fails                       | `show running-config crypto map`                           | Wrong crypto map sequence, wrong tunnel group, wrong key, wrong peer IP, or wrong spoke object                                                                        |
| Static tunnel breaks after adding spoke                 | `show running-config crypto map`                           | Crypto map sequence overlap or crypto ACL overlap                                                                                                                     |
| Hub has IKEv2 SA but no data flows                      | `show crypto ipsec sa`                                     | CHILD SA not built, crypto ACL mismatch, or no interesting traffic                                                                                                    |
| Encrypt counter increments but decrypt does not         | `show crypto ipsec sa`                                     | Remote side not returning traffic, remote NAT issue, remote firewall issue, or peer ACL mismatch                                                                      |
| Decrypt counter increments but local host gets no reply | `show conn`, `show asp drop`, host firewall checks         | Local ACL, host firewall, route, or return path issue                                                                                                                 |
| Spoke-to-spoke traffic fails                            | `show crypto ipsec sa`, `show route`, `show nat detail`    | Hub-spoke design does not automatically allow spoke-to-spoke transit; crypto ACLs, routes, same-interface forwarding, and NAT exemptions must be designed for transit |
| RRI routes missing                                      | `show route` and `show crypto ipsec sa`                    | `set reverse-route` missing or tunnel not active                                                                                                                      |
| RRI installs unexpected routes                          | `show route`                                               | Crypto ACL/proxy ID too broad                                                                                                                                         |
| Outside ACL blocks decrypted VPN traffic                | `show run all sysopt` and `show access-list <outside-acl>` | `no sysopt connection permit-vpn` is configured but ACL does not permit remote-to-local traffic                                                                       |
| Packet-tracer allows but real traffic fails             | `show conn`, `show xlate`, captures, `show asp drop`       | Stale state, peer-side issue, host firewall, or asymmetric routing                                                                                                    |
| Tunnel works until rekey                                | `show crypto ikev2 sa` and `debug crypto ikev2 protocol 2` | Lifetime, PFS, proposal, or peer rekey behavior mismatch                                                                                                              |
| NAT-T tunnel fails behind NAT                           | `show crypto ikev2 sa`, captures on outside                | Upstream NAT/firewall blocks UDP/500 or UDP/4500                                                                                                                      |
| Debugs flood the terminal                               | `show debug`                                               | Debugs left enabled; run `undebug all`                                                                                                                                |
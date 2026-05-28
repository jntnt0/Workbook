
```
# Index
# Source_Basis
# ASA_Remote_Access_IKEv1_Mental_Model
# ASA_Remote_Access_IKEv1_Configuration_Checklist
# ASA_Remote_Access_IKEv1_Local_Users_Skeleton
# ASA_Remote_Access_IKEv1_RADIUS_Skeleton
# ASA_Remote_Access_IKEv1_VPN_Filter_Skeleton
# ASA_Remote_Access_IKEv1_Outside_ACL_Inspection_Skeleton
# ASA_Remote_Access_IKEv1_IPsec_Over_UDP_Skeleton
# ASA_Remote_Access_IKEv1_IPsec_Over_TCP_Skeleton
# ASA_Remote_Access_IKEv1_Verification_Commands
# ASA_Remote_Access_IKEv1_Rollback
# ASA_Remote_Access_IKEv1_Failure_Checks
```


# ASA_Remote_Access_IKEv1_Mental_Model
| Concept                        | Operational Meaning                                                                                                         |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| IKEv1 remote access            | Legacy IPsec VPN client model where remote users connect to the ASA outside interface                                       |
| Client type                    | Usually legacy Cisco VPN Client, built-in legacy Cisco IPsec client, or Easy VPN hardware client                            |
| Tunnel group                   | Connection profile users select or land on during VPN login                                                                 |
| Group policy                   | Policy bundle pushed to users after authentication                                                                          |
| IKEv1 policy                   | Phase 1 policy used to authenticate and build the secure control channel                                                    |
| Transform set                  | Phase 2/IPsec policy used to encrypt user traffic                                                                           |
| Dynamic crypto map             | Required because remote-access clients usually have dynamic public IP addresses                                             |
| Static crypto map wrapper      | Binds the dynamic crypto map to the ASA outside interface                                                                   |
| Address pool                   | VPN adapter IP range assigned to remote users                                                                               |
| User authentication            | Validates the person connecting, usually local database or RADIUS                                                           |
| Group preshared key            | Validates the VPN group before user authentication                                                                          |
| Split tunnel ACL               | Tells the client which internal networks should be encrypted into the tunnel                                                |
| NAT exemption                  | Prevents inside-to-VPN-pool traffic from being translated incorrectly                                                       |
| `sysopt connection permit-vpn` | Lets decrypted VPN traffic bypass interface ACLs by default                                                                 |
| VPN filter                     | Group-policy ACL used to restrict VPN users without disabling global VPN ACL bypass                                         |
| NAT-T                          | Encapsulates IPsec through UDP/4500 when clients sit behind NAT                                                             |
| IPsec over TCP/UDP             | Legacy workaround when ESP or UDP/500/4500 is blocked                                                                       |
| Blunt rule                     | This is a legacy VPN model. Learn it for ASA operations and labs, but do not treat it as the modern AnyConnect/IKEv2 design |
# ASA_Remote_Access_IKEv1_Configuration_Checklist
| Step | Task                                                                | Device      | Command                                                                                                                                               | Expected Result                                                                         |
| ---: | ------------------------------------------------------------------- | ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
|    1 | Confirm ASA interface status                                        | ASA         | `show interface ip brief`                                                                                                                             | Inside and outside interfaces are up/up                                                 |
|    2 | Confirm ASA logical interface names                                 | ASA         | `show nameif`                                                                                                                                         | Interfaces are named correctly, usually `inside` and `outside`                          |
|    3 | Confirm outside interface IP                                        | ASA         | `show running-config interface <outside-interface>`                                                                                                   | Outside interface has expected public or client-reachable IP                            |
|    4 | Confirm inside interface IP                                         | ASA         | `show running-config interface <inside-interface>`                                                                                                    | Inside interface has expected LAN gateway IP                                            |
|    5 | Confirm ASA route to Internet                                       | ASA         | `show route`                                                                                                                                          | ASA has a default route or outside route toward VPN clients                             |
|    6 | Confirm inside protected networks                                   | ASA / Notes | `<inside-network-list>`                                                                                                                               | Internal networks reachable by VPN users are known                                      |
|    7 | Confirm VPN pool                                                    | ASA / Notes | `<vpn-pool-start>-<vpn-pool-end> mask <vpn-pool-mask>`                                                                                                | Remote-access address pool is planned and does not overlap inside LAN                   |
|    8 | Confirm tunnel group name                                           | ASA / Notes | `<tunnel-group-name>`                                                                                                                                 | Client connection profile name is known                                                 |
|    9 | Confirm group policy name                                           | ASA / Notes | `<group-policy-name>`                                                                                                                                 | Group policy name is known                                                              |
|   10 | Confirm group preshared key                                         | ASA / Notes | `<group-pre-shared-key>`                                                                                                                              | Shared group key is known and matches the VPN client profile                            |
|   11 | Confirm user authentication method                                  | ASA / Notes | `LOCAL` or `<aaa-server-group>`                                                                                                                       | ASA knows whether to use local users or external AAA                                    |
|   12 | Confirm IKEv1 Phase 1 parameters                                    | ASA / Notes | `pre-share, aes-256, sha, group 2, lifetime 86400`                                                                                                    | Client and ASA Phase 1 parameters match                                                 |
|   13 | Confirm IPsec Phase 2 parameters                                    | ASA / Notes | `esp-aes-256 esp-sha-hmac`                                                                                                                            | Client and ASA transform set parameters match                                           |
|   14 | Review existing IKEv1 config                                        | ASA         | `show running-config crypto ikev1`                                                                                                                    | Existing IKEv1 policies and outside enablement are visible                              |
|   15 | Review existing transform sets                                      | ASA         | `show running-config crypto ipsec`                                                                                                                    | Existing IPsec transform sets are visible                                               |
|   16 | Review existing crypto maps                                         | ASA         | `show running-config crypto map`                                                                                                                      | Existing static and dynamic crypto map entries are visible                              |
|   17 | Review existing tunnel groups                                       | ASA         | `show running-config tunnel-group`                                                                                                                    | Existing remote-access and L2L tunnel groups are visible                                |
|   18 | Review existing group policies                                      | ASA         | `show running-config group-policy`                                                                                                                    | Existing VPN group policy attributes are visible                                        |
|   19 | Review existing NAT                                                 | ASA         | `show nat detail`                                                                                                                                     | Existing broad PAT and identity NAT rules are visible                                   |
|   20 | Enter configuration mode                                            | ASA         | `configure terminal`                                                                                                                                  | ASA enters global configuration mode                                                    |
|   21 | Enable IKEv1 on outside                                             | ASA         | `crypto ikev1 enable outside`                                                                                                                         | ASA listens for IKEv1 on outside                                                        |
|   22 | Create IKEv1 policy                                                 | ASA         | `crypto ikev1 policy <priority>`                                                                                                                      | ASA enters IKEv1 policy mode                                                            |
|   23 | Set IKEv1 authentication                                            | ASA         | `authentication pre-share`                                                                                                                            | IKEv1 uses preshared-key authentication                                                 |
|   24 | Set IKEv1 encryption                                                | ASA         | `encryption aes-256`                                                                                                                                  | IKEv1 encryption matches client                                                         |
|   25 | Set IKEv1 hash                                                      | ASA         | `hash sha`                                                                                                                                            | IKEv1 hash matches client                                                               |
|   26 | Set IKEv1 DH group                                                  | ASA         | `group 2`                                                                                                                                             | IKEv1 DH group matches client                                                           |
|   27 | Set IKEv1 lifetime                                                  | ASA         | `lifetime 86400`                                                                                                                                      | IKEv1 lifetime is configured                                                            |
|   28 | Create group policy                                                 | ASA         | `group-policy <group-policy-name> internal`                                                                                                           | Group policy exists                                                                     |
|   29 | Enter group policy attributes                                       | ASA         | `group-policy <group-policy-name> attributes`                                                                                                         | ASA enters group policy attribute mode                                                  |
|   30 | Restrict group policy to IKEv1                                      | ASA         | `vpn-tunnel-protocol ikev1`                                                                                                                           | Group policy permits only IKEv1 IPsec remote access                                     |
|   31 | Create VPN address pool                                             | ASA         | `ip local pool <vpn-pool-name> <start-ip>-<end-ip> mask <mask>`                                                                                       | ASA has a pool for VPN client adapter addresses                                         |
|   32 | Bind address pool to group policy                                   | ASA         | `address-pools value <vpn-pool-name>`                                                                                                                 | VPN users receive addresses from the pool                                               |
|   33 | Create split tunnel ACL for inside networks                         | ASA         | `access-list <split-acl> standard permit <inside-network> <inside-mask>`                                                                              | Split tunnel ACL lists internal networks to encrypt                                     |
|   34 | Enable split tunneling                                              | ASA         | `split-tunnel-policy tunnelspecified`                                                                                                                 | Client encrypts only listed internal networks                                           |
|   35 | Bind split tunnel ACL                                               | ASA         | `split-tunnel-network-list value <split-acl>`                                                                                                         | Split tunnel list is pushed to clients                                                  |
|   36 | Push DNS servers                                                    | ASA         | `dns-server value <dns1> <dns2>`                                                                                                                      | VPN clients receive internal DNS servers                                                |
|   37 | Push WINS servers if needed                                         | ASA         | `wins-server value <wins1> <wins2>`                                                                                                                   | VPN clients receive WINS servers if still required                                      |
|   38 | Push default domain                                                 | ASA         | `default-domain value <domain-name>`                                                                                                                  | VPN clients receive internal DNS suffix                                                 |
|   39 | Create tunnel group                                                 | ASA         | `tunnel-group <tunnel-group-name> type remote-access`                                                                                                 | Remote-access connection profile exists                                                 |
|   40 | Enter tunnel group general attributes                               | ASA         | `tunnel-group <tunnel-group-name> general-attributes`                                                                                                 | ASA enters tunnel group general attributes                                              |
|   41 | Bind group policy to tunnel group                                   | ASA         | `default-group-policy <group-policy-name>`                                                                                                            | Users landing on tunnel group inherit the intended policy                               |
|   42 | Configure local user authentication if using local database         | ASA         | `authentication-server-group LOCAL`                                                                                                                   | Tunnel group authenticates users against local ASA database                             |
|   43 | Configure AAA authentication if using RADIUS/LDAP                   | ASA         | `authentication-server-group <aaa-server-group>`                                                                                                      | Tunnel group authenticates users against external AAA                                   |
|   44 | Enter tunnel group IPsec attributes                                 | ASA         | `tunnel-group <tunnel-group-name> ipsec-attributes`                                                                                                   | ASA enters tunnel group IPsec attributes                                                |
|   45 | Configure group preshared key                                       | ASA         | `pre-shared-key <group-pre-shared-key>`                                                                                                               | VPN client group key is configured                                                      |
|   46 | Create local VPN user if using local database                       | ASA         | `username <username> password <password>`                                                                                                             | Local VPN user exists                                                                   |
|   47 | Configure AAA server group if using RADIUS                          | ASA         | `aaa-server <aaa-server-group> protocol radius`                                                                                                       | AAA server group exists                                                                 |
|   48 | Add RADIUS server if using RADIUS                                   | ASA         | `aaa-server <aaa-server-group> (<aaa-interface>) host <radius-ip>`                                                                                    | ASA points to RADIUS server                                                             |
|   49 | Configure RADIUS shared secret                                      | ASA         | `key <radius-shared-secret>`                                                                                                                          | ASA can authenticate to the RADIUS server                                               |
|   50 | Create IKEv1 transform set                                          | ASA         | `crypto ipsec transform-set <transform-set-name> esp-aes-256 esp-sha-hmac`                                                                            | IPsec Phase 2 transform set exists                                                      |
|   51 | Configure dynamic crypto map transform set                          | ASA         | `crypto dynamic-map <dynamic-map-name> <dynamic-seq> set transform-set <transform-set-name>`                                                          | Dynamic crypto map has required transform set                                           |
|   52 | Enable PFS only if required                                         | ASA         | `crypto dynamic-map <dynamic-map-name> <dynamic-seq> set pfs group2`                                                                                  | PFS is configured only when the VPN client supports/matches it                          |
|   53 | Link dynamic map to static crypto map                               | ASA         | `crypto map <outside-crypto-map-name> <static-wrapper-seq> ipsec-isakmp dynamic <dynamic-map-name>`                                                   | Static crypto map wrapper points to dynamic map                                         |
|   54 | Apply crypto map to outside                                         | ASA         | `crypto map <outside-crypto-map-name> interface outside`                                                                                              | Remote-access crypto map is active on outside                                           |
|   55 | Create inside network object for NAT exemption                      | ASA         | `object network <inside-object>`                                                                                                                      | ASA enters inside object mode                                                           |
|   56 | Define inside protected subnet                                      | ASA         | `subnet <inside-network> <inside-mask>`                                                                                                               | Inside protected network object is defined                                              |
|   57 | Create VPN pool object                                              | ASA         | `object network <vpn-pool-object>`                                                                                                                    | ASA enters VPN pool object mode                                                         |
|   58 | Define VPN pool subnet                                              | ASA         | `subnet <vpn-pool-network> <vpn-pool-mask>`                                                                                                           | VPN pool object is defined                                                              |
|   59 | Configure NAT exemption                                             | ASA         | `nat (inside,outside) source static <inside-object> <inside-object> destination static <vpn-pool-object> <vpn-pool-object> no-proxy-arp route-lookup` | Inside-to-VPN-pool traffic is not translated                                            |
|   60 | Confirm default VPN ACL bypass behavior                             | ASA         | `show run all sysopt`                                                                                                                                 | `sysopt connection permit-vpn` is enabled unless intentionally disabled                 |
|   61 | Configure group VPN filter only if restricting user access          | ASA         | `access-list <vpn-filter-acl> extended permit <protocol> <vpn-pool-network> <vpn-pool-mask> <inside-destination> <inside-mask>`                       | VPN user access filter ACL exists                                                       |
|   62 | Apply VPN filter only if needed                                     | ASA         | `group-policy <group-policy-name> attributes` then `vpn-filter value <vpn-filter-acl>`                                                                | VPN traffic is restricted by group policy                                               |
|   63 | Disable VPN ACL bypass only if interface ACL inspection is required | ASA         | `no sysopt connection permit-vpn`                                                                                                                     | Decrypted VPN traffic must pass outside ACL                                             |
|   64 | Permit decrypted VPN traffic if using outside ACL inspection        | ASA         | `access-list <outside-acl> extended permit ip <vpn-pool-network> <vpn-pool-mask> <inside-network> <inside-mask>`                                      | Outside ACL permits VPN users to inside network                                         |
|   65 | Apply outside ACL if required                                       | ASA         | `access-group <outside-acl> in interface outside`                                                                                                     | Outside ACL inspects decrypted VPN traffic                                              |
|   66 | Enable NAT-T keepalives                                             | ASA         | `crypto isakmp nat-traversal <seconds>`                                                                                                               | ASA supports UDP/4500 traversal for clients behind NAT                                  |
|   67 | Enable IPsec over UDP only if needed                                | ASA         | `group-policy <group-policy-name> attributes` then `ipsec-udp enable` and `ipsec-udp-port <port>`                                                     | ASA pushes IPsec over UDP settings to VPN clients                                       |
|   68 | Enable IPsec over TCP only if needed                                | ASA         | `isakmp ipsec-over-tcp port <tcp-port>`                                                                                                               | ASA allows IKE/IPsec over TCP for legacy blocked-network cases                          |
|   69 | Save configuration                                                  | ASA         | `write memory`                                                                                                                                        | Remote-access IKEv1 configuration is saved                                              |
|   70 | Verify IKEv1 outside enablement                                     | ASA         | `show running-config crypto ikev1`                                                                                                                    | IKEv1 is enabled on outside and policy exists                                           |
|   71 | Verify group policy                                                 | ASA         | `show running-config group-policy <group-policy-name>`                                                                                                | Group policy includes IKEv1, pool, split tunnel, DNS/WINS/domain, and optional filter   |
|   72 | Verify tunnel group                                                 | ASA         | `show running-config tunnel-group <tunnel-group-name>`                                                                                                | Tunnel group is `remote-access`, has group policy, auth server group, and preshared key |
|   73 | Verify dynamic crypto map                                           | ASA         | `show running-config crypto dynamic-map`                                                                                                              | Dynamic crypto map has transform set and optional PFS                                   |
|   74 | Verify static wrapper crypto map                                    | ASA         | `show running-config crypto map`                                                                                                                      | Static crypto map links to dynamic map and is applied to outside                        |
|   75 | Verify NAT exemption                                                | ASA         | `show nat detail`                                                                                                                                     | Identity NAT appears before broad PAT                                                   |
|   76 | Connect test VPN client                                             | Client      | `<connect using group name, group key, username, password>`                                                                                           | Client completes tunnel negotiation                                                     |
|   77 | Verify VPN session summary                                          | ASA         | `show vpn-sessiondb detail`                                                                                                                           | Active `IPSec Remote Access` session appears                                            |
|   78 | Verify remote-access session                                        | ASA         | `show vpn-sessiondb remote`                                                                                                                           | Username, assigned IP, public IP, protocol, group policy, and tunnel group appear       |
|   79 | Verify IKEv1 Phase 1                                                | ASA         | `show crypto ikev1 sa detail`                                                                                                                         | Remote-access client shows `AM_ACTIVE`                                                  |
|   80 | Verify IPsec Phase 2                                                | ASA         | `show crypto ipsec sa`                                                                                                                                | Dynamic peer, assigned client IP, proxy identities, and counters appear                 |
|   81 | Verify client traffic                                               | Client      | `ping <inside-host-ip>`                                                                                                                               | Client can reach allowed inside resources                                               |
|   82 | Verify encrypt/decrypt counters                                     | ASA         | `show crypto ipsec sa`                                                                                                                                | Encaps/encrypt and decaps/decrypt counters increment                                    |
|   83 | Verify VPN filter hits if used                                      | ASA         | `show access-list <vpn-filter-acl>`                                                                                                                   | VPN filter hit counts increment                                                         |
|   84 | Verify outside ACL hits if used                                     | ASA         | `show access-list <outside-acl>`                                                                                                                      | Outside ACL hit counts increment if `no sysopt connection permit-vpn` is configured     |
|   85 | Check drops if traffic fails                                        | ASA         | `show asp drop`                                                                                                                                       | Drop reason points to NAT, ACL, VPN filter, route, crypto, or inspection issue          |
|   86 | Debug IKEv1 only when needed                                        | ASA         | `debug crypto ikev1 127`                                                                                                                              | Debug shows group landing, IKE proposal, NAT-T, preshared key, or XAUTH failure         |
|   87 | Debug IPsec only when needed                                        | ASA         | `debug crypto ipsec 127`                                                                                                                              | Debug shows Phase 2/IPsec transform and proxy behavior                                  |
|   88 | Stop debugs                                                         | ASA         | `undebug all`                                                                                                                                         | Debug output stops                                                                      |

```
# ASA_Remote_Access_IKEv1_Local_Users_Skeleton
configure terminal

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
 address-pools value <vpn-pool-name>
 split-tunnel-policy tunnelspecified
 split-tunnel-network-list value <split-acl>
 dns-server value <dns1> <dns2>
 wins-server value <wins1> <wins2>
 default-domain value <domain-name>

ip local pool <vpn-pool-name> <start-ip>-<end-ip> mask <mask>

access-list <split-acl> standard permit <inside-network-1> <inside-mask-1>
access-list <split-acl> standard permit <inside-network-2> <inside-mask-2>
access-list <split-acl> remark Internal networks encrypted by IKEv1 remote-access clients

tunnel-group <tunnel-group-name> type remote-access
tunnel-group <tunnel-group-name> general-attributes
 default-group-policy <group-policy-name>
 authentication-server-group LOCAL
tunnel-group <tunnel-group-name> ipsec-attributes
 pre-shared-key <group-pre-shared-key>

username <username> password <password>

crypto ipsec transform-set <transform-set-name> esp-aes-256 esp-sha-hmac

crypto dynamic-map <dynamic-map-name> <dynamic-seq> set transform-set <transform-set-name>
crypto map <outside-crypto-map-name> <static-wrapper-seq> ipsec-isakmp dynamic <dynamic-map-name>
crypto map <outside-crypto-map-name> interface outside

object network <inside-object>
 subnet <inside-network> <inside-mask>

object network <vpn-pool-object>
 subnet <vpn-pool-network> <vpn-pool-mask>

nat (inside,outside) source static <inside-object> <inside-object> destination static <vpn-pool-object> <vpn-pool-object> no-proxy-arp route-lookup

crypto isakmp nat-traversal <seconds>

write memory
```


```
# ASA_Remote_Access_IKEv1_RADIUS_Skeleton
configure terminal

aaa-server <aaa-server-group> protocol radius
aaa-server <aaa-server-group> (<aaa-interface>) host <radius-ip>
 key <radius-shared-secret>
 exit

tunnel-group <tunnel-group-name> general-attributes
 authentication-server-group <aaa-server-group>

write memory
```


```
# ASA_Remote_Access_IKEv1_VPN_Filter_Skeleton
configure terminal

access-list <vpn-filter-acl> extended permit <protocol> <vpn-pool-network> <vpn-pool-mask> <inside-destination> <inside-mask>
access-list <vpn-filter-acl> extended deny ip any any

group-policy <group-policy-name> attributes
 vpn-filter value <vpn-filter-acl>

write memory
```

```
# ASA_Remote_Access_IKEv1_Outside_ACL_Inspection_Skeleton
configure terminal

no sysopt connection permit-vpn

access-list <outside-acl> extended permit ip <vpn-pool-network> <vpn-pool-mask> <inside-network> <inside-mask>
access-group <outside-acl> in interface outside

write memory
```

```
# ASA_Remote_Access_IKEv1_IPsec_Over_UDP_Skeleton
configure terminal

group-policy <group-policy-name> attributes
 ipsec-udp enable
 ipsec-udp-port <udp-port>

write memory
```


```
# ASA_Remote_Access_IKEv1_IPsec_Over_TCP_Skeleton
configure terminal

isakmp ipsec-over-tcp port <tcp-port>

write memory
```

# ASA_Remote_Access_IKEv1_Verification_Commands
| Task                                 | Command                                                                                                      | Expected Result                                                                                |                                                       |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| Verify interface status              | `show interface ip brief`                                                                                    | Inside and outside interfaces are up/up                                                        |                                                       |
| Verify logical interface names       | `show nameif`                                                                                                | Interfaces match `inside` and `outside` references                                             |                                                       |
| Verify outside reachability route    | `show route`                                                                                                 | ASA has outside default route or route toward clients                                          |                                                       |
| Verify IKEv1 policy                  | `show running-config crypto ikev1`                                                                           | IKEv1 is enabled on outside and policy exists                                                  |                                                       |
| Verify group policy                  | `show running-config group-policy <group-policy-name>`                                                       | Group policy includes IKEv1, address pool, split tunnel, DNS/WINS/domain, and optional filters |                                                       |
| Verify tunnel group                  | `show running-config tunnel-group <tunnel-group-name>`                                                       | Tunnel group is `remote-access` and has group policy, auth source, and preshared key           |                                                       |
| Verify local users                   | `show running-config username`                                                                               | Local VPN users exist if using local database                                                  |                                                       |
| Verify AAA server group              | `show running-config aaa-server`                                                                             | RADIUS/LDAP server group exists if using external authentication                               |                                                       |
| Test AAA server if supported         | `test aaa-server authentication <aaa-server-group> host <server-ip> username <username> password <password>` | AAA authentication succeeds                                                                    |                                                       |
| Verify address pool                  | `show running-config ip local pool`                                                                          | VPN client address pool exists                                                                 |                                                       |
| Verify split tunnel ACL              | `show access-list <split-acl>`                                                                               | Inside networks are listed as standard ACL entries                                             |                                                       |
| Verify transform set                 | `show running-config crypto ipsec`                                                                           | IKEv1 transform set exists                                                                     |                                                       |
| Verify dynamic crypto map            | `show running-config crypto dynamic-map`                                                                     | Dynamic map has transform set and optional PFS                                                 |                                                       |
| Verify static crypto map wrapper     | `show running-config crypto map`                                                                             | Static crypto map links to dynamic map and is applied to outside                               |                                                       |
| Verify NAT exemption                 | `show nat`                                                                                                   | Identity NAT appears before broad PAT                                                          |                                                       |
| Verify NAT details                   | `show nat detail`                                                                                            | Identity NAT shows inside network to VPN pool untranslated                                     |                                                       |
| Verify NAT-T                         | `show running-config crypto isakmp`                                                                          | NAT-T keepalive appears if configured                                                          |                                                       |
| Verify IPsec over TCP                | `show running-config isakmp`                                                                                 | IPsec over TCP port appears if configured                                                      |                                                       |
| Verify VPN session summary           | `show vpn-sessiondb detail`                                                                                  | Active `IPSec Remote Access` session appears after connection                                  |                                                       |
| Verify remote-access session         | `show vpn-sessiondb remote`                                                                                  | Username, assigned IP, public IP, protocol, group policy, and tunnel group appear              |                                                       |
| Verify IKEv1 SA                      | `show crypto ikev1 sa detail`                                                                                | Client shows `Type: user`, `Role: responder`, and `State: AM_ACTIVE`                           |                                                       |
| Verify IPsec SA                      | `show crypto ipsec sa`                                                                                       | Dynamic peer, assigned client IP, local/remote idents, and IPsec counters appear               |                                                       |
| Verify IPsec over TCP use            | `show crypto ipsec sa                                                                                        | include settings`                                                                              | Output shows `TCP-Encaps` if IPsec over TCP is active |
| Verify crypto accelerator            | `show crypto accelerator statistics`                                                                         | Crypto accelerator is active and counters increase                                             |                                                       |
| Verify IKEv1 statistics              | `show crypto protocol statistics ikev1`                                                                      | IKEv1 counters are visible and failures do not climb                                           |                                                       |
| Verify IPsec statistics              | `show crypto protocol statistics ipsec`                                                                      | IPsec counters are visible and failures do not climb                                           |                                                       |
| Verify VPN filter hits               | `show access-list <vpn-filter-acl>`                                                                          | Hit counts increment if VPN filter is configured                                               |                                                       |
| Verify outside ACL hits              | `show access-list <outside-acl>`                                                                             | Hit counts increment if `no sysopt connection permit-vpn` is used                              |                                                       |
| Verify client reachability to inside | `ping <inside-host-ip>` from VPN client                                                                      | Client reaches permitted internal host                                                         |                                                       |
| Verify ASA connection table          | `show conn`                                                                                                  | VPN client flows appear                                                                        |                                                       |
| Verify translations                  | `show xlate`                                                                                                 | VPN pool traffic is not incorrectly PATed                                                      |                                                       |
| Verify drops                         | `show asp drop`                                                                                              | No relevant NAT, ACL, VPN filter, crypto, route, or inspection drops increment                 |                                                       |
| Debug IKEv1                          | `debug crypto ikev1 127`                                                                                     | Debug shows tunnel group match, proposal, NAT-T, preshared key, and XAUTH behavior             |                                                       |
| Debug IPsec                          | `debug crypto ipsec 127`                                                                                     | Debug shows IPsec transform/proxy behavior                                                     |                                                       |
| Stop debugs                          | `undebug all`                                                                                                | Debug output stops                                                                             |                                                       |

# ASA_Remote_Access_IKEv1_Rollback
| Step | Task                                                        | Device | Command                                                                                                                                                  | Expected Result                                                  |
| ---: | ----------------------------------------------------------- | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
|    1 | Identify IKEv1 remote-access configuration                  | ASA    | `show running-config crypto ikev1`                                                                                                                       | IKEv1 outside enablement and policy are identified               |
|    2 | Identify dynamic crypto map                                 | ASA    | `show running-config crypto dynamic-map`                                                                                                                 | Dynamic crypto map name and sequence are identified              |
|    3 | Identify static wrapper crypto map                          | ASA    | `show running-config crypto map`                                                                                                                         | Static crypto map wrapper and outside application are identified |
|    4 | Identify tunnel group                                       | ASA    | `show running-config tunnel-group <tunnel-group-name>`                                                                                                   | Remote-access tunnel group settings are identified               |
|    5 | Identify group policy                                       | ASA    | `show running-config group-policy <group-policy-name>`                                                                                                   | Group policy settings are identified                             |
|    6 | Identify NAT exemption                                      | ASA    | `show nat detail`                                                                                                                                        | Identity NAT rule and object names are identified                |
|    7 | Enter configuration mode                                    | ASA    | `configure terminal`                                                                                                                                     | ASA enters global configuration mode                             |
|    8 | Remove crypto map from outside only if no other VPNs use it | ASA    | `no crypto map <outside-crypto-map-name> interface outside`                                                                                              | Crypto map is detached from outside                              |
|    9 | Remove static wrapper dynamic map entry                     | ASA    | `no crypto map <outside-crypto-map-name> <static-wrapper-seq> ipsec-isakmp dynamic <dynamic-map-name>`                                                   | Static wrapper no longer references dynamic map                  |
|   10 | Remove dynamic crypto map PFS if configured                 | ASA    | `no crypto dynamic-map <dynamic-map-name> <dynamic-seq> set pfs group2`                                                                                  | PFS is removed from dynamic map                                  |
|   11 | Remove dynamic crypto map transform set                     | ASA    | `no crypto dynamic-map <dynamic-map-name> <dynamic-seq> set transform-set <transform-set-name>`                                                          | Transform set is removed from dynamic map                        |
|   12 | Remove transform set                                        | ASA    | `no crypto ipsec transform-set <transform-set-name>`                                                                                                     | Transform set is removed                                         |
|   13 | Remove tunnel group                                         | ASA    | `clear configure tunnel-group <tunnel-group-name>`                                                                                                       | Remote-access connection profile is removed                      |
|   14 | Remove group policy if unused                               | ASA    | `clear configure group-policy <group-policy-name>`                                                                                                       | Group policy is removed if not shared                            |
|   15 | Remove local VPN user if lab-only                           | ASA    | `no username <username>`                                                                                                                                 | Local user is removed                                            |
|   16 | Remove AAA server host if lab-only                          | ASA    | `no aaa-server <aaa-server-group> (<aaa-interface>) host <radius-ip>`                                                                                    | RADIUS server host is removed                                    |
|   17 | Remove AAA server group if unused                           | ASA    | `no aaa-server <aaa-server-group> protocol radius`                                                                                                       | AAA server group is removed if unused                            |
|   18 | Remove address pool                                         | ASA    | `no ip local pool <vpn-pool-name> <start-ip>-<end-ip> mask <mask>`                                                                                       | VPN pool is removed                                              |
|   19 | Remove split tunnel ACL                                     | ASA    | `clear configure access-list <split-acl>`                                                                                                                | Split tunnel ACL is removed                                      |
|   20 | Remove VPN filter ACL                                       | ASA    | `clear configure access-list <vpn-filter-acl>`                                                                                                           | VPN filter ACL is removed                                        |
|   21 | Remove outside ACL entries if added only for VPN            | ASA    | `no access-list <outside-acl> extended permit ip <vpn-pool-network> <vpn-pool-mask> <inside-network> <inside-mask>`                                      | VPN-specific outside ACL entry is removed                        |
|   22 | Restore VPN ACL bypass if changed                           | ASA    | `sysopt connection permit-vpn`                                                                                                                           | Decrypted VPN traffic again bypasses interface ACLs              |
|   23 | Remove NAT exemption                                        | ASA    | `no nat (inside,outside) source static <inside-object> <inside-object> destination static <vpn-pool-object> <vpn-pool-object> no-proxy-arp route-lookup` | Identity NAT exemption is removed                                |
|   24 | Remove inside object if unused                              | ASA    | `no object network <inside-object>`                                                                                                                      | Inside object is removed                                         |
|   25 | Remove VPN pool object if unused                            | ASA    | `no object network <vpn-pool-object>`                                                                                                                    | VPN pool object is removed                                       |
|   26 | Remove IPsec over TCP if lab-only                           | ASA    | `no isakmp ipsec-over-tcp port <tcp-port>`                                                                                                               | IPsec over TCP setting is removed                                |
|   27 | Remove NAT-T custom setting if lab-only                     | ASA    | `no crypto isakmp nat-traversal`                                                                                                                         | Custom NAT-T setting is removed                                  |
|   28 | Remove IKEv1 policy if unused                               | ASA    | `clear configure crypto ikev1 policy <priority>`                                                                                                         | IKEv1 policy is removed                                          |
|   29 | Disable IKEv1 on outside only if no IKEv1 VPNs remain       | ASA    | `no crypto ikev1 enable outside`                                                                                                                         | ASA stops listening for IKEv1 on outside                         |
|   30 | Clear IKEv1 SAs                                             | ASA    | `clear crypto ikev1 sa`                                                                                                                                  | IKEv1 user sessions are cleared                                  |
|   31 | Clear IPsec SAs                                             | ASA    | `clear crypto ipsec sa`                                                                                                                                  | IPsec SAs are cleared                                            |
|   32 | Clear stale translations                                    | ASA    | `clear xlate`                                                                                                                                            | Old translations are cleared                                     |
|   33 | Clear stale connections                                     | ASA    | `clear conn`                                                                                                                                             | Old connection state is cleared                                  |
|   34 | Verify rollback                                             | ASA    | `show running-config crypto`                                                                                                                             | Removed IKEv1 RA crypto config no longer appears                 |
|   35 | Save rollback state                                         | ASA    | `write memory`                                                                                                                                           | Rollback is saved                                                |

# ASA_Remote_Access_IKEv1_Failure_Checks
| Symptom                                                 | Command                                                         | What Usually Broke                                                                                                                                                             |                                                                                      |
| ------------------------------------------------------- | --------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| Client cannot start tunnel                              | `show crypto ikev1 sa detail`                                   | IKEv1 not enabled on outside, wrong public IP, upstream block, or client profile wrong<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br> |                                                                                      |
| No IKEv1 SA appears                                     | `debug crypto ikev1 127`                                        | UDP/500 blocked, client not reaching ASA, wrong interface, or outside ACL/upstream firewall issue                                                                              |                                                                                      |
| IKEv1 proposal rejected                                 | `debug crypto ikev1 127`                                        | IKEv1 policy mismatch: encryption, hash, DH group, authentication, or lifetime                                                                                                 |                                                                                      |
| Tunnel group not matched                                | `debug crypto ikev1 127`                                        | Client group name does not match tunnel group name                                                                                                                             |                                                                                      |
| Group authentication fails                              | `debug crypto ikev1 127`                                        | Tunnel group preshared key mismatch                                                                                                                                            |                                                                                      |
| User authentication fails                               | `show aaa-server`, `debug crypto ikev1 127`                     | Bad username/password, wrong auth server group, RADIUS unreachable, or local user missing                                                                                      |                                                                                      |
| Client connects but gets no IP                          | `show vpn-sessiondb remote`                                     | Address pool missing, exhausted, not bound to group policy, or AAA/DHCP assignment failed                                                                                      |                                                                                      |
| Client receives wrong policy                            | `show vpn-sessiondb remote`                                     | Wrong tunnel group, wrong default group policy, or RADIUS class/OU override                                                                                                    |                                                                                      |
| Client gets full tunnel when split tunnel expected      | `show running-config group-policy <group-policy-name>`          | Missing `split-tunnel-policy tunnelspecified` or missing split tunnel ACL                                                                                                      |                                                                                      |
| Client cannot resolve internal names                    | `show running-config group-policy <group-policy-name>`          | DNS server or default domain not pushed                                                                                                                                        |                                                                                      |
| Client connects but cannot reach inside hosts           | `show nat detail`, `show access-list`, `show asp drop`          | NAT exemption missing, VPN filter blocks traffic, route missing, host firewall, or ACL issue                                                                                   |                                                                                      |
| VPN traffic is PATed                                    | `packet-tracer input inside ... detailed` and `show nat detail` | Identity NAT missing or broad PAT matched first                                                                                                                                |                                                                                      |
| IPsec SA exists but counters stay zero                  | `show crypto ipsec sa`                                          | No interesting traffic, split tunnel wrong, client route missing, or inside host unreachable                                                                                   |                                                                                      |
| Encrypt counter increments but decrypt does not         | `show crypto ipsec sa`                                          | Client not returning traffic, client firewall, NAT-T issue, or upstream filtering                                                                                              |                                                                                      |
| Decrypt counter increments but inside host gets nothing | `show conn`, `show asp drop`                                    | ASA receives VPN traffic but inside ACL, route, host firewall, or VPN filter blocks it                                                                                         |                                                                                      |
| VPN filter denies traffic                               | `show access-list <vpn-filter-acl>`                             | VPN filter is too narrow or has implicit deny after specific permits                                                                                                           |                                                                                      |
| Outside ACL denies decrypted traffic                    | `show run all sysopt` and `show access-list <outside-acl>`      | `no sysopt connection permit-vpn` is configured but outside ACL does not permit VPN pool to inside                                                                             |                                                                                      |
| NAT-T fails behind home router                          | `debug crypto ikev1 127`, captures                              | UDP/4500 blocked, NAT-T disabled, or broken NAT device                                                                                                                         |                                                                                      |
| IPsec works only on some networks                       | `show crypto ipsec sa                                           | include settings`                                                                                                                                                              | ESP/UDP blocked on client network; may need NAT-T, IPsec over TCP, or IPsec over UDP |
| Client connects but Internet breaks                     | Client route table and split tunnel config                      | Full tunnel is enabled without U-turn NAT/Internet policy, or split tunneling is missing                                                                                       |                                                                                      |
| Client-to-client traffic fails                          | `show running-config same-security-traffic`                     | IPsec hairpinning not enabled or same-interface VPN design missing                                                                                                             |                                                                                      |
| Debugs flood terminal                                   | `show debug`                                                    | Debugs left enabled; run `undebug all`                                                                                                                                         |                                                                                      |
| Legacy client cannot connect on modern workstation      | Client/version check                                            | Cisco IKEv1 VPN client is end-of-life and may not run on modern OS versions                                                                                                    |                                                                                      |

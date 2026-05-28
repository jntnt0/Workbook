

```
# Index
# Source_Basis
# ASA_Remote_Access_Native_Windows_L2TP_IPsec_Mental_Model
# ASA_Remote_Access_Native_Windows_L2TP_IPsec_Configuration_Checklist
# ASA_Remote_Access_Native_Windows_L2TP_IPsec_Full_Tunnel_Skeleton
# ASA_Remote_Access_Native_Windows_L2TP_IPsec_Split_Tunnel_Skeleton
# ASA_Remote_Access_Native_Windows_L2TP_IPsec_RADIUS_Skeleton
# ASA_Remote_Access_Native_Windows_L2TP_IPsec_VPN_Filter_Skeleton
# ASA_Remote_Access_Native_Windows_L2TP_IPsec_Outside_ACL_Inspection_Skeleton
# ASA_Remote_Access_Native_Windows_L2TP_IPsec_Windows_Client_Skeleton
# ASA_Remote_Access_Native_Windows_L2TP_IPsec_Verification_Commands
# ASA_Remote_Access_Native_Windows_L2TP_IPsec_Rollback
# ASA_Remote_Access_Native_Windows_L2TP_IPsec_Failure_Checks
```
# ASA_Remote_Access_Native_Windows_L2TP_IPsec_Mental_Mode

| Concept                   | Operational Meaning                                                                                                                |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Native Windows L2TP/IPsec | Windows built-in VPN client connects without Cisco VPN Client or AnyConnect                                                        |
| L2TP alone                | Not supported natively on ASA without IPsec protection                                                                             |
| IKEv1 only                | ASA L2TP over IPsec does not support IKEv2                                                                                         |
| Transport mode            | Required because Windows L2TP/IPsec protects UDP/1701 L2TP traffic with IPsec transport mode                                       |
| IKEv1 Phase 1             | Builds the control/security channel between Windows client and ASA                                                                 |
| IPsec Phase 2             | Encrypts L2TP traffic, usually UDP/1701                                                                                            |
| L2TP session              | Starts after IPsec is established                                                                                                  |
| PPP authentication        | User authentication happens inside the L2TP session                                                                                |
| `DefaultRAGroup`          | Required connection profile for preshared-key Windows L2TP/IPsec because the Windows client does not specify a custom tunnel group |
| Group policy              | Pushes VPN protocol, DNS, WINS, default domain, pool, and split tunnel behavior                                                    |
| Address pool              | VPN client receives an internal tunnel IP from the ASA pool                                                                        |
| Local `mschap` user       | Required when local database is used with MS-CHAPv1 or MS-CHAPv2                                                                   |
| Dynamic crypto map        | Required because native Windows clients arrive from dynamic public IPs                                                             |
| NAT exemption             | Prevents inside-to-VPN-pool traffic from being PATed                                                                               |
| NAT-T                     | Required when clients sit behind NAT/PAT devices                                                                                   |
| Split tunnel              | Requires ASA split tunnel ACL plus Windows setting to not use default gateway on remote network                                    |
| Blunt rule                | This is legacy compatibility VPN. Useful to know, but it is not the modern AnyConnect/Secure Client design                         |
|                           |                                                                                                                                    |
|                           |                                                                                                                                    |
# ASA_Remote_Access_Native_Windows_L2TP_IPsec_Configuration_Checklist
| Step | Task                                                                       | Device          | Command                                                                                                                                               | Expected Result                                                                   |
| ---: | -------------------------------------------------------------------------- | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
|    1 | Confirm ASA interface status                                               | ASA             | `show interface ip brief`                                                                                                                             | Inside and outside interfaces are up/up                                           |
|    2 | Confirm logical interface names                                            | ASA             | `show nameif`                                                                                                                                         | Interfaces are named correctly, usually `inside` and `outside`                    |
|    3 | Confirm ASA is routed mode                                                 | ASA             | `show firewall`                                                                                                                                       | Firewall mode is routed, not transparent                                          |
|    4 | Confirm outside interface IP                                               | ASA             | `show running-config interface <outside-interface>`                                                                                                   | Outside interface has expected public or NATed client-reachable IP                |
|    5 | Confirm inside interface IP                                                | ASA             | `show running-config interface <inside-interface>`                                                                                                    | Inside interface has expected LAN gateway IP                                      |
|    6 | Confirm ASA default route                                                  | ASA             | `show route`                                                                                                                                          | ASA has outside route toward client networks/Internet                             |
|    7 | Confirm client can reach ASA public IP                                     | Windows Client  | `Test-NetConnection <asa-public-ip> -Port 500 -InformationLevel Detailed`                                                                             | Basic path toward ASA exists, though UDP behavior may need packet capture         |
|    8 | Confirm upstream firewall requirements                                     | Network / Notes | `UDP/500, UDP/4500, ESP protocol 50, UDP/1701 inside IPsec`                                                                                           | Path does not block required L2TP/IPsec traffic                                   |
|    9 | Confirm VPN pool                                                           | ASA / Notes     | `<vpn-pool-start>-<vpn-pool-end> mask <vpn-pool-mask>`                                                                                                | VPN client pool is planned and does not overlap inside LAN                        |
|   10 | Confirm VPN pool network object                                            | ASA / Notes     | `<vpn-pool-network> <vpn-pool-mask>`                                                                                                                  | Pool subnet is ready for NAT exemption                                            |
|   11 | Confirm inside protected networks                                          | ASA / Notes     | `<inside-network-list>`                                                                                                                               | Internal networks Windows clients should reach are known                          |
|   12 | Confirm authentication model                                               | ASA / Notes     | `LOCAL` or `<aaa-server-group>`                                                                                                                       | ASA knows whether users authenticate locally or through AAA                       |
|   13 | Confirm PPP authentication                                                 | ASA / Notes     | `ms-chap-v2 preferred`                                                                                                                                | Windows client and ASA support matching PPP authentication                        |
|   14 | Confirm preshared key                                                      | ASA / Notes     | `<l2tp-ipsec-pre-shared-key>`                                                                                                                         | Same key will be entered on ASA and Windows client                                |
|   15 | Confirm group policy name                                                  | ASA / Notes     | `<group-policy-name>`                                                                                                                                 | Group policy name is selected                                                     |
|   16 | Confirm transform set name                                                 | ASA / Notes     | `<transform-set-name>`                                                                                                                                | IPsec transform set name is selected                                              |
|   17 | Confirm dynamic crypto map name                                            | ASA / Notes     | `<dynamic-map-name>`                                                                                                                                  | Dynamic crypto map name is selected                                               |
|   18 | Confirm outside crypto map name                                            | ASA / Notes     | `<outside-crypto-map-name>`                                                                                                                           | Static wrapper crypto map name is selected                                        |
|   19 | Review existing IKEv1 config                                               | ASA             | `show running-config crypto ikev1`                                                                                                                    | Existing IKEv1 policies and outside enablement are visible                        |
|   20 | Review existing IPsec transform sets                                       | ASA             | `show running-config crypto ipsec`                                                                                                                    | Existing transform sets and proposals are visible                                 |
|   21 | Review existing crypto maps                                                | ASA             | `show running-config crypto map`                                                                                                                      | Existing static and dynamic crypto maps are visible                               |
|   22 | Review existing group policies                                             | ASA             | `show running-config group-policy`                                                                                                                    | Existing VPN policies are visible                                                 |
|   23 | Review existing tunnel groups                                              | ASA             | `show running-config tunnel-group`                                                                                                                    | Existing `DefaultRAGroup` configuration is visible                                |
|   24 | Review existing NAT table                                                  | ASA             | `show nat detail`                                                                                                                                     | Broad PAT and existing identity NAT rules are visible                             |
|   25 | Enter configuration mode                                                   | ASA             | `configure terminal`                                                                                                                                  | ASA enters global configuration mode                                              |
|   26 | Create VPN address pool                                                    | ASA             | `ip local pool <vpn-pool-name> <start-ip>-<end-ip> mask <mask>`                                                                                       | VPN address pool exists                                                           |
|   27 | Create group policy                                                        | ASA             | `group-policy <group-policy-name> internal`                                                                                                           | Internal group policy exists                                                      |
|   28 | Enter group policy attributes                                              | ASA             | `group-policy <group-policy-name> attributes`                                                                                                         | ASA enters group-policy attribute mode                                            |
|   29 | Set L2TP/IPsec tunnel protocol                                             | ASA             | `vpn-tunnel-protocol l2tp-ipsec`                                                                                                                      | Group policy permits L2TP over IPsec                                              |
|   30 | Push DNS servers                                                           | ASA             | `dns-server value <dns1> <dns2>`                                                                                                                      | Windows client receives internal DNS servers                                      |
|   31 | Push WINS servers if needed                                                | ASA             | `wins-server value <wins1> <wins2>`                                                                                                                   | Windows client receives WINS servers if required                                  |
|   32 | Push default domain                                                        | ASA             | `default-domain value <domain-name>`                                                                                                                  | Windows client receives DNS suffix                                                |
|   33 | Configure split tunnel policy if split tunnel is desired                   | ASA             | `split-tunnel-policy tunnelspecified`                                                                                                                 | ASA will push only specified encrypted routes                                     |
|   34 | Create split tunnel ACL if split tunnel is desired                         | ASA             | `access-list <split-acl> standard permit <inside-network> <inside-mask>`                                                                              | Split tunnel list includes internal network                                       |
|   35 | Attach split tunnel ACL                                                    | ASA             | `split-tunnel-network-list value <split-acl>`                                                                                                         | Split tunnel ACL is applied to group policy                                       |
|   36 | Enable DHCP intercept for Windows classless routes if split tunnel is used | ASA             | `intercept-dhcp 255.255.255.255 enable`                                                                                                               | ASA responds to DHCPINFORM for route attributes                                   |
|   37 | Create local user if using local database                                  | ASA             | `username <username> password <password> mschap`                                                                                                      | Local user exists with MS-CHAP-compatible stored password                         |
|   38 | Configure AAA server group if using RADIUS                                 | ASA             | `aaa-server <aaa-server-group> protocol radius`                                                                                                       | AAA server group exists                                                           |
|   39 | Add RADIUS server if using RADIUS                                          | ASA             | `aaa-server <aaa-server-group> (<aaa-interface>) host <radius-ip>`                                                                                    | ASA points to RADIUS server                                                       |
|   40 | Configure RADIUS shared secret                                             | ASA             | `key <radius-shared-secret>`                                                                                                                          | ASA can authenticate to RADIUS                                                    |
|   41 | Enter `DefaultRAGroup` general attributes                                  | ASA             | `tunnel-group DefaultRAGroup general-attributes`                                                                                                      | ASA enters default remote-access tunnel group general attributes                  |
|   42 | Bind VPN address pool to `DefaultRAGroup`                                  | ASA             | `address-pool <vpn-pool-name>`                                                                                                                        | `DefaultRAGroup` assigns VPN client addresses                                     |
|   43 | Bind group policy to `DefaultRAGroup`                                      | ASA             | `default-group-policy <group-policy-name>`                                                                                                            | L2TP users inherit intended group policy                                          |
|   44 | Configure local authentication                                             | ASA             | `authentication-server-group LOCAL`                                                                                                                   | L2TP users authenticate against ASA local database                                |
|   45 | Configure AAA authentication if using RADIUS                               | ASA             | `authentication-server-group <aaa-server-group> LOCAL`                                                                                                | ASA uses AAA and can fall back to local if configured                             |
|   46 | Enter `DefaultRAGroup` IPsec attributes                                    | ASA             | `tunnel-group DefaultRAGroup ipsec-attributes`                                                                                                        | ASA enters IPsec attributes mode                                                  |
|   47 | Configure IKEv1 preshared key                                              | ASA             | `ikev1 pre-shared-key <l2tp-ipsec-pre-shared-key>`                                                                                                    | ASA expects same PSK configured on Windows client                                 |
|   48 | Enter `DefaultRAGroup` PPP attributes                                      | ASA             | `tunnel-group DefaultRAGroup ppp-attributes`                                                                                                          | ASA enters PPP attributes mode                                                    |
|   49 | Disable default CHAP if using local database                               | ASA             | `no authentication chap`                                                                                                                              | Unsupported local CHAP path is avoided                                            |
|   50 | Enable MS-CHAPv2                                                           | ASA             | `authentication ms-chap-v2`                                                                                                                           | Windows client can authenticate with MS-CHAPv2                                    |
|   51 | Enable MS-CHAPv1 only if legacy client requires it                         | ASA             | `authentication ms-chap-v1`                                                                                                                           | Legacy MS-CHAPv1 is allowed only if required                                      |
|   52 | Avoid PAP unless specifically required                                     | ASA             | `authentication pap`                                                                                                                                  | PAP is only enabled if your design accepts weak PPP authentication                |
|   53 | Create IKEv1 policy                                                        | ASA             | `crypto ikev1 policy <priority>`                                                                                                                      | ASA enters IKEv1 policy mode                                                      |
|   54 | Set IKEv1 authentication                                                   | ASA             | `authentication pre-share`                                                                                                                            | IKEv1 uses preshared key authentication                                           |
|   55 | Set IKEv1 encryption                                                       | ASA             | `encryption aes-256`                                                                                                                                  | IKEv1 encryption uses AES-256                                                     |
|   56 | Set IKEv1 hash                                                             | ASA             | `hash sha`                                                                                                                                            | IKEv1 integrity uses SHA-1                                                        |
|   57 | Set IKEv1 DH group                                                         | ASA             | `group 14`                                                                                                                                            | IKEv1 uses DH group 14                                                            |
|   58 | Set IKEv1 lifetime                                                         | ASA             | `lifetime 86400`                                                                                                                                      | IKEv1 SA lifetime is 24 hours                                                     |
|   59 | Create IKEv1 transform set                                                 | ASA             | `crypto ipsec ikev1 transform-set <transform-set-name> esp-aes-256 esp-sha-hmac`                                                                      | IPsec Phase 2 transform set exists                                                |
|   60 | Set transport mode                                                         | ASA             | `crypto ipsec ikev1 transform-set <transform-set-name> mode transport`                                                                                | Transform set uses transport mode required by Windows L2TP/IPsec                  |
|   61 | Create dynamic crypto map                                                  | ASA             | `crypto dynamic-map <dynamic-map-name> <dynamic-seq> set ikev1 transform-set <transform-set-name>`                                                    | Dynamic map accepts Windows clients with dynamic public IPs                       |
|   62 | Do not enable PFS unless Windows client compatibility is confirmed         | ASA / Notes     | `No PFS by default`                                                                                                                                   | Avoids client compatibility failures                                              |
|   63 | Link dynamic map to static wrapper                                         | ASA             | `crypto map <outside-crypto-map-name> <static-wrapper-seq> ipsec-isakmp dynamic <dynamic-map-name>`                                                   | Static crypto map wrapper references dynamic map                                  |
|   64 | Apply crypto map to outside                                                | ASA             | `crypto map <outside-crypto-map-name> interface outside`                                                                                              | Crypto map is active on outside                                                   |
|   65 | Enable IKEv1 on outside                                                    | ASA             | `crypto ikev1 enable outside`                                                                                                                         | ASA listens for IKEv1 on outside                                                  |
|   66 | Enable NAT-T                                                               | ASA             | `crypto isakmp nat-traversal <seconds>`                                                                                                               | Clients behind NAT can use UDP/4500                                               |
|   67 | Configure L2TP hello interval if needed                                    | ASA             | `l2tp tunnel hello <seconds>`                                                                                                                         | L2TP hello timer is set                                                           |
|   68 | Create inside network object                                               | ASA             | `object network <inside-object>`                                                                                                                      | ASA enters inside object mode                                                     |
|   69 | Define inside protected network                                            | ASA             | `subnet <inside-network> <inside-mask>`                                                                                                               | Inside protected network object is defined                                        |
|   70 | Create VPN pool object                                                     | ASA             | `object network <vpn-pool-object>`                                                                                                                    | ASA enters VPN pool object mode                                                   |
|   71 | Define VPN pool subnet                                                     | ASA             | `subnet <vpn-pool-network> <vpn-pool-mask>`                                                                                                           | VPN pool object is defined                                                        |
|   72 | Configure NAT exemption                                                    | ASA             | `nat (inside,outside) source static <inside-object> <inside-object> destination static <vpn-pool-object> <vpn-pool-object> no-proxy-arp route-lookup` | Inside-to-VPN-pool traffic is not translated                                      |
|   73 | Confirm default VPN ACL bypass behavior                                    | ASA             | `show run all sysopt`                                                                                                                                 | `sysopt connection permit-vpn` is enabled unless intentionally disabled           |
|   74 | Configure VPN filter only if restricting users                             | ASA             | `vpn-filter value <vpn-filter-acl>` under group-policy attributes                                                                                     | L2TP users are restricted by VPN filter ACL                                       |
|   75 | Configure outside ACL inspection only if required                          | ASA             | `no sysopt connection permit-vpn`                                                                                                                     | Decrypted VPN traffic must pass outside ACL                                       |
|   76 | Permit VPN pool to inside if outside ACL inspection is used                | ASA             | `access-list <outside-acl> extended permit ip <vpn-pool-network> <vpn-pool-mask> <inside-network> <inside-mask>`                                      | Outside ACL permits decrypted L2TP VPN traffic                                    |
|   77 | Apply outside ACL if required                                              | ASA             | `access-group <outside-acl> in interface outside`                                                                                                     | Outside ACL inspects decrypted VPN traffic                                        |
|   78 | Save configuration                                                         | ASA             | `write memory`                                                                                                                                        | ASA L2TP/IPsec configuration is saved                                             |
|   79 | Create Windows VPN profile                                                 | Windows Client  | Settings > Network & Internet > VPN > Add VPN                                                                                                         | Windows VPN profile exists                                                        |
|   80 | Set VPN provider                                                           | Windows Client  | `Windows built-in`                                                                                                                                    | Native Windows client is used                                                     |
|   81 | Set server name/address                                                    | Windows Client  | `<asa-public-ip-or-fqdn>`                                                                                                                             | Client points to ASA outside address                                              |
|   82 | Set VPN type                                                               | Windows Client  | `L2TP/IPsec with pre-shared key`                                                                                                                      | Windows uses L2TP/IPsec                                                           |
|   83 | Enter preshared key                                                        | Windows Client  | `<l2tp-ipsec-pre-shared-key>`                                                                                                                         | Windows PSK matches ASA `DefaultRAGroup` PSK                                      |
|   84 | Set sign-in type                                                           | Windows Client  | `User name and password`                                                                                                                              | Windows prompts for ASA/AAA credentials                                           |
|   85 | Set adapter security options                                               | Windows Client  | Adapter Properties > Security > Allow these protocols > MS-CHAP v2                                                                                    | Windows PPP auth matches ASA                                                      |
|   86 | Configure split tunnel behavior if needed                                  | Windows Client  | Uncheck `Use default gateway on remote network`                                                                                                       | Windows does not send all traffic through VPN if split tunnel is intended         |
|   87 | Connect test client                                                        | Windows Client  | Connect VPN profile                                                                                                                                   | Windows establishes L2TP/IPsec session                                            |
|   88 | Verify IKEv1 SA                                                            | ASA             | `show crypto ikev1 sa`                                                                                                                                | Client appears with active IKEv1 SA                                               |
|   89 | Verify IPsec SA                                                            | ASA             | `show crypto ipsec sa`                                                                                                                                | Local/remote idents show UDP/1701 and counters increment                          |
|   90 | Verify L2TP session                                                        | ASA             | `show vpdn session`                                                                                                                                   | L2TP/PPP session appears                                                          |
|   91 | Verify L2TP tunnel                                                         | ASA             | `show vpdn tunnel`                                                                                                                                    | Active L2TP tunnel appears                                                        |
|   92 | Verify VPN session database                                                | ASA             | `show vpn-sessiondb remote`                                                                                                                           | Username, assigned IP, tunnel group, and protocol appear                          |
|   93 | Verify client assigned address                                             | Windows Client  | `ipconfig /all`                                                                                                                                       | VPN adapter receives ASA pool IP                                                  |
|   94 | Test inside reachability                                                   | Windows Client  | `ping <inside-host-ip>` or service test                                                                                                               | Client reaches permitted inside resource                                          |
|   95 | Verify NAT exemption hits                                                  | ASA             | `show nat detail`                                                                                                                                     | Identity NAT hit count increments                                                 |
|   96 | Verify IPsec counters                                                      | ASA             | `show crypto ipsec sa`                                                                                                                                | Encaps/encrypt and decaps/decrypt counters increment                              |
|   97 | Check drops if traffic fails                                               | ASA             | `show asp drop`                                                                                                                                       | Drop reason points to NAT, ACL, VPN filter, route, PPP, L2TP, IKE, or IPsec issue |
|   98 | Debug IKEv1 only when needed                                               | ASA             | `debug crypto ikev1 127`                                                                                                                              | Debug shows IKEv1 negotiation and PSK/proposal behavior                           |
|   99 | Debug IPsec only when needed                                               | ASA             | `debug crypto ipsec 127`                                                                                                                              | Debug shows Phase 2/UDP 1701 protection behavior                                  |
|  100 | Debug L2TP/PPP only when needed                                            | ASA             | `debug vpdn event` and `debug ppp authentication`                                                                                                     | Debug shows L2TP and PPP authentication behavior                                  |
|  101 | Stop debugs                                                                | ASA             | `undebug all`                                                                                                                                         | Debug output stops                                                                |
|      |                                                                            |                 |                                                                                                                                                       |                                                                                   |

```
# ASA_Remote_Access_Native_Windows_L2TP_IPsec_Full_Tunnel_Skeleton
configure terminal

ip local pool <vpn-pool-name> <start-ip>-<end-ip> mask <mask>

group-policy <group-policy-name> internal
group-policy <group-policy-name> attributes
 vpn-tunnel-protocol l2tp-ipsec
 dns-server value <dns1> <dns2>
 wins-server value <wins1> <wins2>
 default-domain value <domain-name>

username <username> password <password> mschap

tunnel-group DefaultRAGroup general-attributes
 address-pool <vpn-pool-name>
 default-group-policy <group-policy-name>
 authentication-server-group LOCAL

tunnel-group DefaultRAGroup ipsec-attributes
 ikev1 pre-shared-key <l2tp-ipsec-pre-shared-key>

tunnel-group DefaultRAGroup ppp-attributes
 no authentication chap
 authentication ms-chap-v2

crypto ikev1 policy <priority>
 authentication pre-share
 encryption aes-256
 hash sha
 group 14
 lifetime 86400

crypto ipsec ikev1 transform-set <transform-set-name> esp-aes-256 esp-sha-hmac
crypto ipsec ikev1 transform-set <transform-set-name> mode transport

crypto dynamic-map <dynamic-map-name> <dynamic-seq> set ikev1 transform-set <transform-set-name>
crypto map <outside-crypto-map-name> <static-wrapper-seq> ipsec-isakmp dynamic <dynamic-map-name>
crypto map <outside-crypto-map-name> interface outside
crypto ikev1 enable outside

crypto isakmp nat-traversal <seconds>
l2tp tunnel hello <seconds>

object network <inside-object>
 subnet <inside-network> <inside-mask>

object network <vpn-pool-object>
 subnet <vpn-pool-network> <vpn-pool-mask>

nat (inside,outside) source static <inside-object> <inside-object> destination static <vpn-pool-object> <vpn-pool-object> no-proxy-arp route-lookup

write memory
```

```
# ASA_Remote_Access_Native_Windows_L2TP_IPsec_Split_Tunnel_Skeleton
configure terminal

access-list <split-acl> standard permit <inside-network-1> <inside-mask-1>
access-list <split-acl> standard permit <inside-network-2> <inside-mask-2>

group-policy <group-policy-name> attributes
 split-tunnel-policy tunnelspecified
 split-tunnel-network-list value <split-acl>
 intercept-dhcp 255.255.255.255 enable

write memory
```


```
# ASA_Remote_Access_Native_Windows_L2TP_IPsec_RADIUS_Skeleton
configure terminal

aaa-server <aaa-server-group> protocol radius
aaa-server <aaa-server-group> (<aaa-interface>) host <radius-ip>
 key <radius-shared-secret>
 exit

tunnel-group DefaultRAGroup general-attributes
 authentication-server-group <aaa-server-group> LOCAL

tunnel-group DefaultRAGroup ppp-attributes
 no authentication chap
 authentication ms-chap-v2

write memory
```


```
# ASA_Remote_Access_Native_Windows_L2TP_IPsec_VPN_Filter_Skeleton
configure terminal

access-list <vpn-filter-acl> remark Filter for native Windows L2TP/IPsec users
access-list <vpn-filter-acl> extended permit tcp <vpn-pool-network> <vpn-pool-mask> host <inside-host-ip> eq <port>
access-list <vpn-filter-acl> extended permit icmp <vpn-pool-network> <vpn-pool-mask> <inside-network> <inside-mask>
access-list <vpn-filter-acl> extended deny ip any any log

group-policy <group-policy-name> attributes
 vpn-filter value <vpn-filter-acl>

write memory
```

```
# ASA_Remote_Access_Native_Windows_L2TP_IPsec_Outside_ACL_Inspection_Skeleton
configure terminal

no sysopt connection permit-vpn

access-list <outside-acl> extended permit ip <vpn-pool-network> <vpn-pool-mask> <inside-network> <inside-mask>
access-group <outside-acl> in interface outside

write memory
```


```
# ASA_Remote_Access_Native_Windows_L2TP_IPsec_Windows_Client_Skeleton
Windows Settings:
1. Open Settings.
2. Go to Network & Internet.
3. Open VPN.
4. Add VPN.
5. VPN provider: Windows built-in.
6. Connection name: <connection-name>.
7. Server name or address: <asa-public-ip-or-fqdn>.
8. VPN type: L2TP/IPsec with pre-shared key.
9. Pre-shared key: <l2tp-ipsec-pre-shared-key>.
10. Type of sign-in info: User name and password.
11. Save the profile.

Adapter Properties:
1. Open adapter options for the VPN profile.
2. Security tab.
3. Type of VPN: Layer 2 Tunneling Protocol with IPsec.
4. Advanced settings: Use pre-shared key for authentication.
5. Enter the same preshared key used on ASA.
6. Authentication: Allow these protocols.
7. Select Microsoft CHAP Version 2.
8. Clear unsupported protocols unless required.
9. Networking tab.
10. IPv4 Properties > Advanced.
11. For split tunnel: uncheck Use default gateway on remote network.
12. For full tunnel: leave Use default gateway on remote network checked.
13. Connect with <username> and <password>.
```

# ASA_Remote_Access_Native_Windows_L2TP_IPsec_Verification_Commands
| Task                                         | Command                                                | Expected Result                                                                 |
| -------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------- |
| Verify interface state                       | `show interface ip brief`                              | Inside and outside interfaces are up/up                                         |
| Verify logical names                         | `show nameif`                                          | Interfaces match `inside` and `outside` references                              |
| Verify firewall mode                         | `show firewall`                                        | ASA is in routed mode                                                           |
| Verify outside route                         | `show route`                                           | ASA has route toward client/Internet                                            |
| Verify group policy                          | `show running-config group-policy <group-policy-name>` | Policy includes `vpn-tunnel-protocol l2tp-ipsec`                                |
| Verify split tunnel ACL                      | `show access-list <split-acl>`                         | Split tunnel destinations are correct if split tunnel is used                   |
| Verify address pool                          | `show running-config ip local pool`                    | VPN pool exists with correct range and mask                                     |
| Verify local MS-CHAP user                    | `show running-config username <username>`              | User exists and includes `mschap` if local database is used                     |
| Verify `DefaultRAGroup` general attributes   | `show running-config tunnel-group DefaultRAGroup`      | Address pool, group policy, and authentication source are configured            |
| Verify `DefaultRAGroup` IPsec attributes     | `show running-config tunnel-group DefaultRAGroup`      | IKEv1 preshared key is configured                                               |
| Verify PPP attributes                        | `show running-config tunnel-group DefaultRAGroup`      | MS-CHAPv2 is enabled and unsupported CHAP is disabled when using local database |
| Verify IKEv1 policy                          | `show running-config crypto ikev1`                     | IKEv1 policy and `crypto ikev1 enable outside` are present                      |
| Verify transport transform set               | `show running-config crypto ipsec`                     | Transform set uses ESP encryption/hash and `mode transport`                     |
| Verify dynamic crypto map                    | `show running-config crypto dynamic-map`               | Dynamic map has IKEv1 transform set                                             |
| Verify static crypto map wrapper             | `show running-config crypto map`                       | Static map references dynamic map and is applied to outside                     |
| Verify NAT-T                                 | `show running-config crypto isakmp`                    | NAT-T keepalive is configured                                                   |
| Verify L2TP hello                            | `show running-config l2tp`                             | L2TP hello interval appears if configured                                       |
| Verify NAT exemption                         | `show nat`                                             | Identity NAT appears before broad PAT                                           |
| Verify NAT exemption detail                  | `show nat detail`                                      | Identity NAT shows inside network to VPN pool untranslated                      |
| Verify Windows VPN IP                        | `ipconfig /all`                                        | Windows VPN adapter receives ASA pool IP                                        |
| Verify Windows routes                        | `route print`                                          | Full tunnel or split tunnel routes match design                                 |
| Verify IKEv1 SA                              | `show crypto ikev1 sa`                                 | Client appears as active IKEv1 peer                                             |
| Verify IKEv1 SA detail                       | `show crypto ikev1 sa detail`                          | Type is user and state is active                                                |
| Verify IPsec SA                              | `show crypto ipsec sa`                                 | Local and remote idents show UDP/1701 selectors                                 |
| Verify IPsec counters                        | `show crypto ipsec sa`                                 | Encaps/encrypt and decaps/decrypt counters increment                            |
| Verify L2TP session                          | `show vpdn session`                                    | Active L2TP session appears                                                     |
| Verify L2TP tunnel                           | `show vpdn tunnel`                                     | Active L2TP tunnel appears                                                      |
| Verify VPN session database                  | `show vpn-sessiondb remote`                            | User, assigned IP, public IP, tunnel group, and group policy appear             |
| Verify connection table                      | `show conn address <vpn-client-assigned-ip>`           | Client flows appear                                                             |
| Verify translations                          | `show xlate`                                           | VPN pool traffic is not incorrectly PATed                                       |
| Verify VPN filter hits                       | `show access-list <vpn-filter-acl>`                    | Filter hits increment if VPN filter is configured                               |
| Verify outside ACL hits if `sysopt` disabled | `show access-list <outside-acl>`                       | Outside ACL hits increment if interface ACL inspection is used                  |
| Verify drops                                 | `show asp drop`                                        | No relevant NAT, ACL, PPP, L2TP, crypto, route, or inspection drops increment   |
| Debug IKEv1 if needed                        | `debug crypto ikev1 127`                               | Debug shows Phase 1 and preshared-key/proposal behavior                         |
| Debug IPsec if needed                        | `debug crypto ipsec 127`                               | Debug shows transport mode and UDP/1701 protection behavior                     |
| Debug L2TP if needed                         | `debug vpdn event`                                     | Debug shows L2TP tunnel/session behavior                                        |
| Debug PPP auth if needed                     | `debug ppp authentication`                             | Debug shows MS-CHAP/PAP authentication behavior                                 |
| Stop debugs                                  | `undebug all`                                          | Debug output stops                                                              |

# ASA_Remote_Access_Native_Windows_L2TP_IPsec_Rollback
| Step | Task                                                        | Device         | Command                                                                                                                                                  | Expected Result                                          |
| ---: | ----------------------------------------------------------- | -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
|    1 | Identify L2TP group policy                                  | ASA            | `show running-config group-policy <group-policy-name>`                                                                                                   | L2TP group policy settings are identified                |
|    2 | Identify `DefaultRAGroup` config                            | ASA            | `show running-config tunnel-group DefaultRAGroup`                                                                                                        | Pool, policy, auth, PSK, and PPP settings are identified |
|    3 | Identify dynamic crypto map                                 | ASA            | `show running-config crypto dynamic-map`                                                                                                                 | Dynamic map name and sequence are identified             |
|    4 | Identify static crypto map wrapper                          | ASA            | `show running-config crypto map`                                                                                                                         | Wrapper map and outside interface binding are identified |
|    5 | Identify NAT exemption                                      | ASA            | `show nat detail`                                                                                                                                        | Identity NAT rule and object names are identified        |
|    6 | Enter configuration mode                                    | ASA            | `configure terminal`                                                                                                                                     | ASA enters global configuration mode                     |
|    7 | Remove crypto map from outside only if no other VPNs use it | ASA            | `no crypto map <outside-crypto-map-name> interface outside`                                                                                              | Crypto map is detached from outside                      |
|    8 | Remove static wrapper dynamic map entry                     | ASA            | `no crypto map <outside-crypto-map-name> <static-wrapper-seq> ipsec-isakmp dynamic <dynamic-map-name>`                                                   | Static wrapper no longer references dynamic map          |
|    9 | Remove dynamic crypto map transform set                     | ASA            | `no crypto dynamic-map <dynamic-map-name> <dynamic-seq> set ikev1 transform-set <transform-set-name>`                                                    | Dynamic map no longer references transform set           |
|   10 | Remove transform set                                        | ASA            | `no crypto ipsec ikev1 transform-set <transform-set-name>`                                                                                               | IKEv1 transform set is removed                           |
|   11 | Remove `DefaultRAGroup` address pool binding                | ASA            | `tunnel-group DefaultRAGroup general-attributes` then `no address-pool <vpn-pool-name>`                                                                  | Pool binding is removed                                  |
|   12 | Remove `DefaultRAGroup` group policy binding                | ASA            | `tunnel-group DefaultRAGroup general-attributes` then `no default-group-policy <group-policy-name>`                                                      | Group policy binding is removed                          |
|   13 | Remove `DefaultRAGroup` authentication server binding       | ASA            | `tunnel-group DefaultRAGroup general-attributes` then `no authentication-server-group <auth-source>`                                                     | Authentication binding is removed                        |
|   14 | Remove `DefaultRAGroup` preshared key                       | ASA            | `tunnel-group DefaultRAGroup ipsec-attributes` then `no ikev1 pre-shared-key <l2tp-ipsec-pre-shared-key>`                                                | PSK is removed                                           |
|   15 | Remove PPP authentication settings                          | ASA            | `tunnel-group DefaultRAGroup ppp-attributes` then `no authentication ms-chap-v2`                                                                         | MS-CHAPv2 setting is removed                             |
|   16 | Remove group policy if unused                               | ASA            | `clear configure group-policy <group-policy-name>`                                                                                                       | L2TP group policy is removed if not shared               |
|   17 | Remove local user if lab-only                               | ASA            | `no username <username>`                                                                                                                                 | Local VPN user is removed                                |
|   18 | Remove AAA server host if lab-only                          | ASA            | `no aaa-server <aaa-server-group> (<aaa-interface>) host <radius-ip>`                                                                                    | RADIUS host is removed                                   |
|   19 | Remove AAA server group if unused                           | ASA            | `no aaa-server <aaa-server-group> protocol radius`                                                                                                       | AAA server group is removed if unused                    |
|   20 | Remove VPN address pool                                     | ASA            | `no ip local pool <vpn-pool-name> <start-ip>-<end-ip> mask <mask>`                                                                                       | VPN pool is removed                                      |
|   21 | Remove split tunnel ACL if unused                           | ASA            | `clear configure access-list <split-acl>`                                                                                                                | Split tunnel ACL is removed                              |
|   22 | Remove VPN filter ACL if unused                             | ASA            | `clear configure access-list <vpn-filter-acl>`                                                                                                           | VPN filter ACL is removed                                |
|   23 | Remove outside ACL entry if lab-only                        | ASA            | `no access-list <outside-acl> extended permit ip <vpn-pool-network> <vpn-pool-mask> <inside-network> <inside-mask>`                                      | VPN-specific outside ACL entry is removed                |
|   24 | Restore VPN ACL bypass if changed                           | ASA            | `sysopt connection permit-vpn`                                                                                                                           | Decrypted VPN traffic bypasses interface ACLs again      |
|   25 | Remove NAT exemption                                        | ASA            | `no nat (inside,outside) source static <inside-object> <inside-object> destination static <vpn-pool-object> <vpn-pool-object> no-proxy-arp route-lookup` | Identity NAT exemption is removed                        |
|   26 | Remove inside object if unused                              | ASA            | `no object network <inside-object>`                                                                                                                      | Inside object is removed                                 |
|   27 | Remove VPN pool object if unused                            | ASA            | `no object network <vpn-pool-object>`                                                                                                                    | VPN pool object is removed                               |
|   28 | Remove L2TP hello custom setting if lab-only                | ASA            | `no l2tp tunnel hello <seconds>`                                                                                                                         | Custom L2TP hello setting is removed                     |
|   29 | Remove NAT-T custom setting if lab-only                     | ASA            | `no crypto isakmp nat-traversal`                                                                                                                         | NAT-T custom setting is removed                          |
|   30 | Remove IKEv1 policy if unused                               | ASA            | `clear configure crypto ikev1 policy <priority>`                                                                                                         | IKEv1 policy is removed                                  |
|   31 | Disable IKEv1 on outside only if no IKEv1 VPNs remain       | ASA            | `no crypto ikev1 enable outside`                                                                                                                         | ASA stops listening for IKEv1 on outside                 |
|   32 | Clear IKEv1 SAs                                             | ASA            | `clear crypto ikev1 sa`                                                                                                                                  | IKEv1 SAs are cleared                                    |
|   33 | Clear IPsec SAs                                             | ASA            | `clear crypto ipsec sa`                                                                                                                                  | IPsec SAs are cleared                                    |
|   34 | Clear stale translations                                    | ASA            | `clear xlate`                                                                                                                                            | Old translations are cleared                             |
|   35 | Clear stale connections                                     | ASA            | `clear conn`                                                                                                                                             | Old connection state is cleared                          |
|   36 | Delete Windows VPN profile                                  | Windows Client | Settings > Network & Internet > VPN > Remove profile                                                                                                     | Client no longer has the L2TP profile                    |
|   37 | Verify rollback                                             | ASA            | `show running-config crypto`                                                                                                                             | Removed L2TP/IPsec crypto config no longer appears       |
|   38 | Save rollback state                                         | ASA            | `write memory`                                                                                                                                           | Rollback is saved                                        |

# ASA_Remote_Access_Native_Windows_L2TP_IPsec_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| Windows cannot connect at all | `show crypto ikev1 sa` | Client traffic not reaching ASA, IKEv1 not enabled, wrong ASA public IP, or UDP/500 blocked |
| No IKEv1 SA appears | `debug crypto ikev1 127` | Upstream firewall/NAT problem, wrong Windows server address, or IKEv1 disabled on outside |
| IKEv1 fails proposal negotiation | `debug crypto ikev1 127` | IKEv1 policy mismatch, weak/deprecated client proposal, wrong DH group, or encryption mismatch |
| IKEv1 fails authentication | `debug crypto ikev1 127` | Windows preshared key does not match `DefaultRAGroup` |
| IPsec Phase 2 fails | `debug crypto ipsec 127` | Transform set mismatch, missing transport mode, dynamic crypto map issue, or PFS mismatch |
| IPsec SA shows wrong mode | `show running-config crypto ipsec` | Transform set missing `mode transport` |
| IPsec SA is up but L2TP fails | `show vpdn session`, `debug vpdn event` | L2TP/UDP 1701 issue, PPP authentication issue, or tunnel group PPP attributes mismatch |
| User authentication fails | `debug ppp authentication` | Missing `username ... mschap`, wrong password, wrong PPP auth method, or AAA issue |
| Local MS-CHAP user fails | `show running-config username <username>` | Username was created without `mschap` keyword |
| Client gets no VPN IP | `show vpn-sessiondb remote` | Address pool missing, exhausted, or not bound to `DefaultRAGroup` |
| Client connects but cannot reach inside | `show nat detail`, `show asp drop`, `show conn` | NAT exemption missing, route missing, VPN filter, outside ACL inspection, or host firewall |
| VPN traffic gets PATed | `show xlate` and `show nat detail` | Missing inside-to-VPN-pool identity NAT or broad PAT matched first |
| Split tunnel routes missing on Windows | `route print` | Split tunnel ACL missing, Windows default gateway setting wrong, or missing DHCP intercept for classless routes |
| Full tunnel breaks Internet | Windows route table and ASA NAT config | Client sends Internet through ASA but ASA lacks U-turn/outside PAT policy for VPN pool |
| DNS fails after connection | `show running-config group-policy <group-policy-name>` | DNS server/default domain not pushed or DNS blocked by VPN filter |
| Ping fails but TCP works | `show access-list <vpn-filter-acl>` | ICMP not permitted or inside host firewall blocks ping |
| TCP fails but ping works | `show access-list <vpn-filter-acl>` | Service port blocked, service not listening, host firewall, or VPN filter too narrow |
| Multiple clients behind same home NAT fail | `show crypto ikev1 sa`, `show crypto ipsec sa` | NAT-T missing or upstream device mishandles IPsec passthrough |
| ASA outside ACL blocks decrypted traffic | `show run all sysopt` and `show access-list <outside-acl>` | `no sysopt connection permit-vpn` is configured and outside ACL does not permit VPN pool to inside |
| VPN filter blocks traffic | `show access-list <vpn-filter-acl>` | VPN filter has missing permit or explicit deny catches traffic |
| IPsec counters encrypt but do not decrypt | `show crypto ipsec sa` | Client/firewall/NAT path not returning traffic |
| IPsec counters decrypt but inside host gets nothing | `show conn`, `show asp drop`, inside host firewall check | ASA receives traffic but route, ACL, host firewall, or return path fails |
| Windows says policy match failed | Windows event logs and ASA `debug crypto ikev1 127` | IKE policy, transform set, PSK, or transport mode mismatch |
| Windows connects but shows no internal access | `ipconfig /all`, `route print`, `show vpn-sessiondb remote` | Client got IP but lacks route/DNS, NAT exemption, or allowed access |
| Debugs flood terminal | `show debug` | Debugs left enabled; run `undebug all` |

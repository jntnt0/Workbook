

```
# Index
# Source_Basis
# ASA_Remote_Access_IKEv2_Mental_Model
# ASA_Remote_Access_IKEv2_Configuration_Checklist
# ASA_Remote_Access_IKEv2_AnyConnect_Local_Users_Skeleton
# ASA_Remote_Access_IKEv2_RADIUS_Skeleton
# ASA_Remote_Access_IKEv2_PSK_Lab_Skeleton
# ASA_Remote_Access_IKEv2_Standards_Based_EAP_Skeleton
# ASA_Remote_Access_IKEv2_VPN_Filter_Skeleton
# ASA_Remote_Access_IKEv2_Outside_ACL_Inspection_Skeleton
# ASA_Remote_Access_IKEv2_Verification_Commands
# ASA_Remote_Access_IKEv2_Rollback
# ASA_Remote_Access_IKEv2_Failure_Checks
```


# ASA_Remote_Access_IKEv2_Mental_Model
| Concept                                 | Operational Meaning                                                                                                                                                             |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| IKEv2 remote access                     | Remote users connect to ASA using IPsec/IKEv2 instead of legacy IKEv1                                                                                                           |
| AnyConnect / Secure Client              | Primary Cisco client used for ASA IKEv2 remote access                                                                                                                           |
| ASA device certificate                  | Required for AnyConnect IKEv2 server identity                                                                                                                                   |
| Trustpoint                              | ASA PKI object used by IKEv2 and SSL/client services                                                                                                                            |
| `crypto ikev2 remote-access trustpoint` | Tells ASA which certificate trustpoint to use for IKEv2 remote-access authentication                                                                                            |
| Client services                         | HTTPS service used for AnyConnect software/profile/file downloads for IPsec IKEv2 clients                                                                                       |
| `webvpn` section                        | Still used for AnyConnect image, profile, group alias, and client service delivery                                                                                              |
| IKEv2 policy                            | Controls Phase 1 parameters: encryption, integrity, PRF, DH group, and lifetime                                                                                                 |
| IKEv2 IPsec proposal                    | Controls Phase 2 data protection: ESP encryption and ESP integrity                                                                                                              |
| Dynamic crypto map                      | Required because remote-access clients arrive from unknown or changing public IPs                                                                                               |
| Static crypto map wrapper               | Applies the dynamic crypto map to the outside interface                                                                                                                         |
| Tunnel group                            | Connection profile selected by the user or profile                                                                                                                              |
| Group policy                            | Defines user policy such as protocol, pool, DNS, split tunnel, VPN filter, and AnyConnect profile                                                                               |
| Address pool                            | VPN adapter IP range assigned to connected clients                                                                                                                              |
| Split tunnel ACL                        | Defines internal networks that the client should send through the VPN                                                                                                           |
| NAT exemption                           | Prevents inside-to-VPN-pool traffic from being translated incorrectly                                                                                                           |
| Local authentication                    | Can work for simple lab AnyConnect IKEv2 cases using local usernames                                                                                                            |
| EAP / standards-based IKEv2             | Usually requires external AAA and certificate authentication behavior                                                                                                           |
| Mobike                                  | Allows IKEv2 remote-access sessions to survive client IP changes if negotiated                                                                                                  |
| Blunt rule                              | IKEv2 remote access is not just IKEv1 with a different policy. The certificate, trustpoint, AnyConnect client services, dynamic map, pool, and group policy all have to line up |


# ASA_Remote_Access_IKEv2_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm ASA interface status | ASA | `show interface ip brief` | Inside and outside interfaces are up/up |
| 2 | Confirm ASA logical interface names | ASA | `show nameif` | Interfaces are named correctly, usually `inside` and `outside` |
| 3 | Confirm outside interface IP | ASA | `show running-config interface <outside-interface>` | Outside interface has expected public or client-reachable IP |
| 4 | Confirm inside interface IP | ASA | `show running-config interface <inside-interface>` | Inside interface has expected LAN gateway IP |
| 5 | Confirm ASA route to Internet | ASA | `show route` | ASA has default route or outside route toward VPN clients |
| 6 | Confirm routed firewall mode | ASA | `show firewall` | ASA is in routed mode |
| 7 | Confirm inside protected networks | ASA / Notes | `<inside-network-list>` | Internal networks reachable by VPN users are known |
| 8 | Confirm VPN pool | ASA / Notes | `<vpn-pool-start>-<vpn-pool-end> mask <vpn-pool-mask>` | VPN client address pool is planned and does not overlap inside LAN |
| 9 | Confirm tunnel group name | ASA / Notes | `<tunnel-group-name>` | AnyConnect connection profile name is known |
| 10 | Confirm group policy name | ASA / Notes | `<group-policy-name>` | Group policy name is known |
| 11 | Confirm ASA FQDN or certificate CN/SAN | ASA / Notes | `<vpn-fqdn>` | AnyConnect profile host address matches ASA certificate identity |
| 12 | Confirm trustpoint name | ASA / Notes | `<trustpoint-name>` | ASA trustpoint for IKEv2 remote access is known |
| 13 | Confirm AnyConnect package path | ASA / Notes | `disk0:/<anyconnect-package>.pkg` | AnyConnect package exists or will be uploaded |
| 14 | Confirm AnyConnect profile path | ASA / Notes | `disk0:/<anyconnect-profile>.xml` | AnyConnect profile exists or will be uploaded |
| 15 | Confirm user authentication method | ASA / Notes | `LOCAL` or `<aaa-server-group>` | ASA knows whether users authenticate locally or through AAA |
| 16 | Confirm IKEv2 Phase 1 parameters | ASA / Notes | `encryption <ike-encryption>, integrity <ike-integrity>, group <dh-group>, prf <prf>, lifetime seconds <seconds>` | ASA and clients support matching IKEv2 policy |
| 17 | Confirm IPsec Phase 2 parameters | ASA / Notes | `protocol esp encryption <esp-encryption>, protocol esp integrity <esp-integrity>` | ASA and clients support matching IPsec proposal |
| 18 | Review existing certificates | ASA | `show crypto ca certificates` | Existing ASA identity and CA certificates are visible |
| 19 | Review existing SSL trustpoint binding | ASA | `show running-config ssl` | Outside SSL trustpoint is known |
| 20 | Review existing WebVPN / AnyConnect config | ASA | `show running-config webvpn` | Existing AnyConnect images, profiles, and enablement are visible |
| 21 | Review existing IKEv2 config | ASA | `show running-config crypto ikev2` | Existing IKEv2 policies, enablement, and trustpoint are visible |
| 22 | Review existing IPsec proposals | ASA | `show running-config crypto ipsec` | Existing IKEv2 proposals are visible |
| 23 | Review existing crypto maps | ASA | `show running-config crypto map` | Existing static and dynamic crypto map entries are visible |
| 24 | Review existing tunnel groups | ASA | `show running-config tunnel-group` | Existing remote-access and L2L tunnel groups are visible |
| 25 | Review existing group policies | ASA | `show running-config group-policy` | Existing VPN group policy attributes are visible |
| 26 | Review existing NAT | ASA | `show nat detail` | Existing broad PAT and identity NAT rules are visible |
| 27 | Enter configuration mode | ASA | `configure terminal` | ASA enters global configuration mode |
| 28 | Configure hostname if certificate identity requires it | ASA | `hostname <asa-hostname>` | ASA hostname is set |
| 29 | Configure domain name if certificate identity requires it | ASA | `domain-name <domain-name>` | ASA domain name is set |
| 30 | Generate RSA key pair if trustpoint does not exist | ASA | `crypto key generate rsa label <rsa-keypair-name> modulus <rsa-modulus>` | RSA key pair exists for the ASA certificate |
| 31 | Create trustpoint if not already present | ASA | `crypto ca trustpoint <trustpoint-name>` | ASA enters trustpoint configuration mode |
| 32 | Bind key pair to trustpoint if using named RSA key | ASA | `keypair <rsa-keypair-name>` | Trustpoint uses intended key pair |
| 33 | Configure enrollment method | ASA | `enrollment terminal` or `enrollment url <scep-url>` | Trustpoint has manual or SCEP enrollment method |
| 34 | Configure FQDN for certificate request | ASA | `fqdn <vpn-fqdn>` | Certificate request includes ASA FQDN |
| 35 | Configure subject name | ASA | `subject-name <subject-name>` | Certificate request includes expected subject DN |
| 36 | Configure CRL behavior | ASA | `crl optional` | ASA uses practical revocation behavior for lab or production policy |
| 37 | Authenticate CA certificate if using CA-signed identity cert | ASA | `crypto ca authenticate <trustpoint-name>` | CA certificate is imported and trusted |
| 38 | Enroll identity certificate if using CA-signed cert | ASA | `crypto ca enroll <trustpoint-name>` | ASA receives identity certificate |
| 39 | Verify certificate availability | ASA | `show crypto ca certificates <trustpoint-name>` | Identity certificate shows available and valid |
| 40 | Bind certificate to SSL on outside | ASA | `ssl trust-point <trustpoint-name> outside` | ASA uses the certificate for HTTPS/client services on outside |
| 41 | Enter WebVPN configuration | ASA | `webvpn` | ASA enters WebVPN configuration mode |
| 42 | Enable WebVPN/AnyConnect services on outside | ASA | `enable outside` | AnyConnect web/client service is enabled on outside |
| 43 | Add AnyConnect package | ASA | `anyconnect image disk0:/<anyconnect-package>.pkg <sequence>` | ASA can deploy the AnyConnect/Secure Client package |
| 44 | Add AnyConnect profile | ASA | `anyconnect profiles <profile-name> disk0:/<anyconnect-profile>.xml` | ASA can offer the IKEv2 client profile |
| 45 | Enable AnyConnect | ASA | `anyconnect enable` | AnyConnect client functionality is enabled |
| 46 | Enable tunnel group list if users should choose profiles | ASA | `tunnel-group-list enable` | Login page can show connection profiles |
| 47 | Exit WebVPN mode | ASA | `exit` | ASA returns to global configuration mode |
| 48 | Enable IKEv2 with client services | ASA | `crypto ikev2 enable outside client-services port 443` | ASA listens for IKEv2 and provides client services on outside |
| 49 | Bind IKEv2 remote-access trustpoint | ASA | `crypto ikev2 remote-access trustpoint <trustpoint-name>` | IKEv2 RA uses the intended ASA certificate |
| 50 | Create IKEv2 policy | ASA | `crypto ikev2 policy <priority>` | ASA enters IKEv2 policy mode |
| 51 | Set IKEv2 encryption | ASA | `encryption <ike-encryption>` | IKEv2 encryption is configured |
| 52 | Set IKEv2 integrity | ASA | `integrity <ike-integrity>` | IKEv2 integrity is configured |
| 53 | Set IKEv2 DH group | ASA | `group <dh-group-list>` | IKEv2 DH group list is configured |
| 54 | Set IKEv2 PRF | ASA | `prf <prf-algorithm>` | IKEv2 PRF is configured |
| 55 | Set IKEv2 lifetime | ASA | `lifetime seconds <seconds>` | IKEv2 lifetime is configured |
| 56 | Create IKEv2 IPsec proposal | ASA | `crypto ipsec ikev2 ipsec-proposal <ipsec-proposal-name>` | ASA enters IPsec proposal mode |
| 57 | Set ESP encryption | ASA | `protocol esp encryption <esp-encryption>` | Data-plane encryption is configured |
| 58 | Set ESP integrity | ASA | `protocol esp integrity <esp-integrity>` | Data-plane integrity is configured |
| 59 | Create VPN address pool | ASA | `ip local pool <vpn-pool-name> <start-ip>-<end-ip> mask <mask>` | ASA has a pool for VPN client adapter addresses |
| 60 | Create local user if using local authentication | ASA | `username <username> password <password>` | Local VPN user exists |
| 61 | Configure AAA server group if using RADIUS | ASA | `aaa-server <aaa-server-group> protocol radius` | AAA server group exists |
| 62 | Add RADIUS server if using RADIUS | ASA | `aaa-server <aaa-server-group> (<aaa-interface>) host <radius-ip>` | ASA points to RADIUS server |
| 63 | Configure RADIUS shared secret | ASA | `key <radius-shared-secret>` | ASA can authenticate to RADIUS |
| 64 | Create group policy | ASA | `group-policy <group-policy-name> internal` | Group policy exists |
| 65 | Enter group policy attributes | ASA | `group-policy <group-policy-name> attributes` | ASA enters group-policy attribute mode |
| 66 | Permit IKEv2 tunnel protocol | ASA | `vpn-tunnel-protocol ikev2` | Group policy permits IKEv2 remote access |
| 67 | Include SSL client only if profile/download behavior requires it | ASA | `vpn-tunnel-protocol ikev2 ssl-client` | Group policy permits both IKEv2 and SSL client behavior |
| 68 | Bind address pool to group policy | ASA | `address-pools value <vpn-pool-name>` | VPN users receive addresses from the pool |
| 69 | Create split tunnel ACL | ASA | `access-list <split-acl> standard permit <inside-network> <inside-mask>` | Split tunnel ACL lists internal protected networks |
| 70 | Enable split tunnel | ASA | `split-tunnel-policy tunnelspecified` | Client sends only listed networks through VPN |
| 71 | Bind split tunnel ACL | ASA | `split-tunnel-network-list value <split-acl>` | Split tunnel list is pushed to client |
| 72 | Push DNS servers | ASA | `dns-server value <dns1> <dns2>` | VPN clients receive DNS servers |
| 73 | Push WINS behavior | ASA | `wins-server none` or `wins-server value <wins1> <wins2>` | WINS is disabled or explicitly pushed |
| 74 | Push default domain | ASA | `default-domain value <domain-name>` | VPN clients receive DNS suffix |
| 75 | Enter group-policy WebVPN attributes | ASA | `webvpn` | ASA enters group-policy WebVPN submode |
| 76 | Bind AnyConnect profile | ASA | `anyconnect profiles value <profile-name> type user` | Users receive the intended AnyConnect IKEv2 profile |
| 77 | Create remote-access tunnel group | ASA | `tunnel-group <tunnel-group-name> type remote-access` | Connection profile exists |
| 78 | Enter tunnel group general attributes | ASA | `tunnel-group <tunnel-group-name> general-attributes` | ASA enters tunnel group general attributes |
| 79 | Bind group policy to tunnel group | ASA | `default-group-policy <group-policy-name>` | Users landing on tunnel group inherit intended policy |
| 80 | Bind address pool to tunnel group if not using group-policy pool | ASA | `address-pool <vpn-pool-name>` | Tunnel group assigns VPN addresses |
| 81 | Configure local authentication if using local database | ASA | `authentication-server-group LOCAL` | Tunnel group authenticates against local ASA users |
| 82 | Configure AAA authentication if using RADIUS/LDAP | ASA | `authentication-server-group <aaa-server-group>` | Tunnel group authenticates against external AAA |
| 83 | Enter tunnel group IPsec attributes | ASA | `tunnel-group <tunnel-group-name> ipsec-attributes` | ASA enters tunnel group IPsec attributes |
| 84 | Configure IKEv2 local preshared key if using PSK lab model | ASA | `ikev2 local-authentication pre-shared-key <local-key>` | ASA presents local key |
| 85 | Configure IKEv2 remote preshared key if using PSK lab model | ASA | `ikev2 remote-authentication pre-shared-key <remote-key>` | ASA expects remote key |
| 86 | Configure certificate local auth for standards-based EAP model | ASA | `ikev2 local-authentication certificate <trustpoint-name>` | ASA authenticates itself with certificate |
| 87 | Configure EAP remote auth for standards-based model | ASA | `ikev2 remote-authentication eap query-identity` | ASA queries user identity for EAP |
| 88 | Enter tunnel group WebVPN attributes | ASA | `tunnel-group <tunnel-group-name> webvpn-attributes` | ASA enters tunnel group WebVPN attributes |
| 89 | Enable group alias | ASA | `group-alias <tunnel-group-name> enable` | Users can select the connection profile alias |
| 90 | Create dynamic crypto map entry | ASA | `crypto dynamic-map <dynamic-map-name> <dynamic-seq> set ikev2 ipsec-proposal <ipsec-proposal-name>` | Dynamic map has IKEv2 IPsec proposal |
| 91 | Enable reverse route injection | ASA | `crypto dynamic-map <dynamic-map-name> <dynamic-seq> set reverse-route` | ASA installs routes for connected clients if supported by design |
| 92 | Link dynamic map to static crypto map | ASA | `crypto map <outside-crypto-map-name> <static-wrapper-seq> ipsec-isakmp dynamic <dynamic-map-name>` | Static wrapper references dynamic crypto map |
| 93 | Apply crypto map to outside | ASA | `crypto map <outside-crypto-map-name> interface outside` | Remote-access crypto map is active on outside |
| 94 | Create inside network object for NAT exemption | ASA | `object network <inside-object>` | ASA enters inside object mode |
| 95 | Define inside protected subnet | ASA | `subnet <inside-network> <inside-mask>` | Inside protected network object is defined |
| 96 | Create VPN pool object | ASA | `object network <vpn-pool-object>` | ASA enters VPN pool object mode |
| 97 | Define VPN pool subnet | ASA | `subnet <vpn-pool-network> <vpn-pool-mask>` | VPN pool object is defined |
| 98 | Configure NAT exemption | ASA | `nat (inside,outside) source static <inside-object> <inside-object> destination static <vpn-pool-object> <vpn-pool-object> no-proxy-arp route-lookup` | Inside-to-VPN-pool traffic is not translated |
| 99 | Configure VPN filter only if restricting VPN users | ASA | `vpn-filter value <vpn-filter-acl>` under group-policy attributes | VPN users are restricted by group-policy ACL |
| 100 | Confirm default VPN ACL bypass behavior | ASA | `show run all sysopt` | `sysopt connection permit-vpn` is enabled unless intentionally disabled |
| 101 | Configure outside ACL inspection only if required | ASA | `no sysopt connection permit-vpn` | Decrypted VPN traffic must pass outside ACL |
| 102 | Permit decrypted VPN traffic if using outside ACL inspection | ASA | `access-list <outside-acl> extended permit ip <vpn-pool-network> <vpn-pool-mask> <inside-network> <inside-mask>` | Outside ACL permits VPN pool to inside |
| 103 | Apply outside ACL if required | ASA | `access-group <outside-acl> in interface outside` | Outside ACL inspects decrypted VPN traffic |
| 104 | Save configuration | ASA | `write memory` | IKEv2 remote-access configuration is saved |
| 105 | Verify certificate binding | ASA | `show crypto ca certificates <trustpoint-name>` | Certificate is present, valid, and not expired |
| 106 | Verify SSL trustpoint | ASA | `show running-config ssl` | Outside interface uses intended trustpoint |
| 107 | Verify AnyConnect WebVPN config | ASA | `show running-config webvpn` | AnyConnect image, profile, enablement, and tunnel-group-list are present |
| 108 | Verify IKEv2 config | ASA | `show running-config crypto ikev2` | IKEv2 is enabled on outside with client services and remote-access trustpoint |
| 109 | Verify group policy | ASA | `show running-config group-policy <group-policy-name>` | Group policy includes IKEv2, pool, split tunnel, DNS, and profile settings |
| 110 | Verify tunnel group | ASA | `show running-config tunnel-group <tunnel-group-name>` | Tunnel group has policy, pool/authentication, alias, and IPsec attributes |
| 111 | Verify dynamic crypto map | ASA | `show running-config crypto dynamic-map` | Dynamic crypto map has IKEv2 proposal and optional reverse-route |
| 112 | Verify crypto map wrapper | ASA | `show running-config crypto map` | Static crypto map references dynamic map and is applied to outside |
| 113 | Verify NAT exemption | ASA | `show nat detail` | Identity NAT appears before broad PAT |
| 114 | Connect test AnyConnect client | Client | `<connect to ASA FQDN using IKEv2 profile>` | Client authenticates and receives VPN IP |
| 115 | Verify VPN session | ASA | `show vpn-sessiondb detail anyconnect` | Active AnyConnect IKEv2 session appears |
| 116 | Verify session protocol and policy | ASA | `show vpn-sessiondb anyconnect` | Username, assigned IP, tunnel group, group policy, and protocol are visible |
| 117 | Verify IKEv2 SA | ASA | `show crypto ikev2 sa detail` | IKEv2 SA is established for the client |
| 118 | Verify IPsec SA | ASA | `show crypto ipsec sa` | Client IP/proxy identities and IPsec counters appear |
| 119 | Test inside reachability | Client | `ping <inside-host-ip>` or service test | Client reaches permitted inside resource |
| 120 | Verify encrypt/decrypt counters | ASA | `show crypto ipsec sa` | Encaps/encrypt and decaps/decrypt counters increment |
| 121 | Check drops if traffic fails | ASA | `show asp drop` | Drop reason points to NAT, ACL, VPN filter, route, crypto, certificate, or inspection issue |
| 122 | Debug IKEv2 only when needed | ASA | `debug crypto ikev2 protocol 2` and `debug crypto ikev2 platform 2` | Debug shows IKEv2 negotiation or authentication issue |
| 123 | Debug IPsec only when needed | ASA | `debug crypto ipsec 127` | Debug shows IPsec proposal or data-plane issue |
| 124 | Stop debugs | ASA | `undebug all` | Debug output stops |
```
# ASA_Remote_Access_IKEv2_AnyConnect_Local_Users_Skeleton
configure terminal

hostname <asa-hostname>
domain-name <domain-name>

crypto key generate rsa label <rsa-keypair-name> modulus <rsa-modulus>

crypto ca trustpoint <trustpoint-name>
 keypair <rsa-keypair-name>
 enrollment terminal
 fqdn <vpn-fqdn>
 subject-name <subject-name>
 crl optional
exit

crypto ca authenticate <trustpoint-name>
crypto ca enroll <trustpoint-name>

ssl trust-point <trustpoint-name> outside

webvpn
 enable outside
 anyconnect image disk0:/<anyconnect-package>.pkg <sequence>
 anyconnect profiles <profile-name> disk0:/<anyconnect-profile>.xml
 anyconnect enable
 tunnel-group-list enable
 exit

crypto ikev2 enable outside client-services port 443
crypto ikev2 remote-access trustpoint <trustpoint-name>

crypto ikev2 policy <priority>
 encryption <ike-encryption>
 integrity <ike-integrity>
 group <dh-group-list>
 prf <prf-algorithm>
 lifetime seconds <seconds>

crypto ipsec ikev2 ipsec-proposal <ipsec-proposal-name>
 protocol esp encryption <esp-encryption>
 protocol esp integrity <esp-integrity>

ip local pool <vpn-pool-name> <start-ip>-<end-ip> mask <mask>

username <username> password <password>

access-list <split-acl> standard permit <inside-network> <inside-mask>

group-policy <group-policy-name> internal
group-policy <group-policy-name> attributes
 vpn-tunnel-protocol ikev2 ssl-client
 address-pools value <vpn-pool-name>
 split-tunnel-policy tunnelspecified
 split-tunnel-network-list value <split-acl>
 dns-server value <dns1> <dns2>
 wins-server none
 default-domain value <domain-name>
 webvpn
  anyconnect profiles value <profile-name> type user

tunnel-group <tunnel-group-name> type remote-access
tunnel-group <tunnel-group-name> general-attributes
 default-group-policy <group-policy-name>
 address-pool <vpn-pool-name>
 authentication-server-group LOCAL
tunnel-group <tunnel-group-name> webvpn-attributes
 group-alias <tunnel-group-name> enable

crypto dynamic-map <dynamic-map-name> <dynamic-seq> set ikev2 ipsec-proposal <ipsec-proposal-name>
crypto dynamic-map <dynamic-map-name> <dynamic-seq> set reverse-route
crypto map <outside-crypto-map-name> <static-wrapper-seq> ipsec-isakmp dynamic <dynamic-map-name>
crypto map <outside-crypto-map-name> interface outside

object network <inside-object>
 subnet <inside-network> <inside-mask>

object network <vpn-pool-object>
 subnet <vpn-pool-network> <vpn-pool-mask>

nat (inside,outside) source static <inside-object> <inside-object> destination static <vpn-pool-object> <vpn-pool-object> no-proxy-arp route-lookup

write memory
```

```
# ASA_Remote_Access_IKEv2_RADIUS_Skeleton
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
# ASA_Remote_Access_IKEv2_PSK_Lab_Skeleton
configure terminal

crypto ikev2 policy <priority>
 encryption <ike-encryption>
 integrity <ike-integrity>
 group <dh-group-list>
 prf <prf-algorithm>
 lifetime seconds <seconds>

crypto ikev2 enable outside

ip local pool <vpn-pool-name> <start-ip>-<end-ip> mask <mask>

username <username> password <password>

crypto ipsec ikev2 ipsec-proposal <ipsec-proposal-name>
 protocol esp encryption <esp-encryption>
 protocol esp integrity <esp-integrity>

tunnel-group <tunnel-group-name> type remote-access
tunnel-group <tunnel-group-name> general-attributes
 address-pool <vpn-pool-name>
 authentication-server-group LOCAL
tunnel-group <tunnel-group-name> ipsec-attributes
 ikev2 local-authentication pre-shared-key <local-key>
 ikev2 remote-authentication pre-shared-key <remote-key>

crypto dynamic-map <dynamic-map-name> <dynamic-seq> set ikev2 ipsec-proposal <ipsec-proposal-name>
crypto dynamic-map <dynamic-map-name> <dynamic-seq> set reverse-route
crypto map <outside-crypto-map-name> <static-wrapper-seq> ipsec-isakmp dynamic <dynamic-map-name>
crypto map <outside-crypto-map-name> interface outside

write memory
```

```
# ASA_Remote_Access_IKEv2_Standards_Based_EAP_Skeleton
configure terminal

crypto ikev2 enable outside
crypto ikev2 remote-access trustpoint <trustpoint-name>

aaa-server <aaa-server-group> protocol radius
aaa-server <aaa-server-group> (<aaa-interface>) host <radius-ip>
 key <radius-shared-secret>
 exit

ip local pool <vpn-pool-name> <start-ip>-<end-ip> mask <mask>

group-policy <group-policy-name> internal
group-policy <group-policy-name> attributes
 vpn-tunnel-protocol ikev2

crypto dynamic-map <dynamic-map-name> <dynamic-seq> set ikev2 ipsec-proposal <ipsec-proposal-name>
crypto map <outside-crypto-map-name> <static-wrapper-seq> ipsec-isakmp dynamic <dynamic-map-name>
crypto map <outside-crypto-map-name> interface outside

tunnel-group DefaultRAGroup type remote-access
tunnel-group DefaultRAGroup general-attributes
 default-group-policy <group-policy-name>
 address-pool <vpn-pool-name>
 authentication-server-group <aaa-server-group>
tunnel-group DefaultRAGroup ipsec-attributes
 ikev2 remote-authentication eap query-identity
 ikev2 local-authentication certificate <trustpoint-name>

write memory
```

```
# ASA_Remote_Access_IKEv2_VPN_Filter_Skeleton
configure terminal

access-list <vpn-filter-acl> remark Filter for <group-policy-name> IKEv2 remote-access users
access-list <vpn-filter-acl> extended permit tcp <vpn-pool-network> <vpn-pool-mask> host <inside-host-ip> eq <port>
access-list <vpn-filter-acl> extended permit icmp <vpn-pool-network> <vpn-pool-mask> <inside-network> <inside-mask>
access-list <vpn-filter-acl> extended deny ip any any log

group-policy <group-policy-name> attributes
 vpn-filter value <vpn-filter-acl>

write memory
```

```
# ASA_Remote_Access_IKEv2_Outside_ACL_Inspection_Skeleton
configure terminal

no sysopt connection permit-vpn

access-list <outside-acl> extended permit ip <vpn-pool-network> <vpn-pool-mask> <inside-network> <inside-mask>
access-group <outside-acl> in interface outside

write memory
```

# ASA_Remote_Access_IKEv2_Verification_Commands
| Task                                         | Command                                                                                                      | Expected Result                                                                                  |
| -------------------------------------------- | ------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------ |
| Verify interface state                       | `show interface ip brief`                                                                                    | Inside and outside interfaces are up/up                                                          |
| Verify interface names                       | `show nameif`                                                                                                | Interface names match `inside` and `outside` references                                          |
| Verify ASA clock                             | `show clock`                                                                                                 | ASA time falls inside certificate validity window                                                |
| Verify certificate                           | `show crypto ca certificates <trustpoint-name>`                                                              | Identity certificate is available and valid                                                      |
| Verify SSL trustpoint                        | `show running-config ssl`                                                                                    | Outside interface uses intended trustpoint                                                       |
| Verify WebVPN / AnyConnect config            | `show running-config webvpn`                                                                                 | AnyConnect image, profile, enablement, and tunnel-group-list are present                         |
| Verify IKEv2 remote-access trustpoint        | `show running-config crypto ikev2`                                                                           | Output includes `crypto ikev2 remote-access trustpoint <trustpoint-name>`                        |
| Verify IKEv2 client services                 | `show running-config crypto ikev2`                                                                           | Output includes `crypto ikev2 enable outside client-services port 443`                           |
| Verify IKEv2 policy                          | `show running-config crypto ikev2`                                                                           | IKEv2 policy has expected encryption, integrity, group, PRF, and lifetime                        |
| Verify IPsec proposal                        | `show running-config crypto ipsec`                                                                           | IKEv2 IPsec proposal has expected ESP encryption and integrity                                   |
| Verify address pool                          | `show running-config ip local pool`                                                                          | VPN pool exists with expected range and mask                                                     |
| Verify local users                           | `show running-config username`                                                                               | Local users exist if LOCAL authentication is used                                                |
| Verify AAA server group                      | `show running-config aaa-server`                                                                             | AAA server group exists if RADIUS/LDAP is used                                                   |
| Test AAA server if supported                 | `test aaa-server authentication <aaa-server-group> host <server-ip> username <username> password <password>` | AAA authentication succeeds                                                                      |
| Verify group policy                          | `show running-config group-policy <group-policy-name>`                                                       | Group policy includes IKEv2, pool, DNS, split tunnel, filter, and AnyConnect profile as expected |
| Verify split tunnel ACL                      | `show access-list <split-acl>`                                                                               | Inside networks are listed as standard ACL entries                                               |
| Verify tunnel group                          | `show running-config tunnel-group <tunnel-group-name>`                                                       | Tunnel group is remote-access and has group policy, pool, auth, alias, and IPsec attributes      |
| Verify dynamic crypto map                    | `show running-config crypto dynamic-map`                                                                     | Dynamic map has IKEv2 proposal and optional reverse-route                                        |
| Verify static crypto map wrapper             | `show running-config crypto map`                                                                             | Static crypto map links to dynamic map and is applied to outside                                 |
| Verify NAT exemption                         | `show nat`                                                                                                   | Identity NAT appears before broad PAT                                                            |
| Verify NAT details                           | `show nat detail`                                                                                            | Identity NAT shows inside network to VPN pool untranslated                                       |
| Verify VPN session summary                   | `show vpn-sessiondb summary`                                                                                 | Active AnyConnect or IKEv2 remote-access session appears                                         |
| Verify AnyConnect session                    | `show vpn-sessiondb anyconnect`                                                                              | Username, assigned IP, public IP, tunnel group, and group policy appear                          |
| Verify detailed AnyConnect session           | `show vpn-sessiondb detail anyconnect`                                                                       | Protocol and policy details are visible                                                          |
| Verify IKEv2 SA                              | `show crypto ikev2 sa detail`                                                                                | IKEv2 SA is established for client                                                               |
| Verify Mobike state                          | `show crypto ikev2 sa detail`                                                                                | Mobike support is visible if negotiated                                                          |
| Verify IPsec SA                              | `show crypto ipsec sa`                                                                                       | Client IP, local/remote idents, and IPsec counters appear                                        |
| Verify outbound traffic                      | `show crypto ipsec sa`                                                                                       | Encaps/encrypt counters increment                                                                |
| Verify inbound traffic                       | `show crypto ipsec sa`                                                                                       | Decaps/decrypt counters increment                                                                |
| Verify VPN filter hits                       | `show access-list <vpn-filter-acl>`                                                                          | VPN filter permit/deny hits increment if configured                                              |
| Verify outside ACL hits if `sysopt` disabled | `show access-list <outside-acl>`                                                                             | Outside ACL hit counts increment for decrypted VPN traffic                                       |
| Verify ASA connection table                  | `show conn address <vpn-client-assigned-ip>`                                                                 | VPN client flows appear                                                                          |
| Verify translations                          | `show xlate`                                                                                                 | VPN pool traffic is not incorrectly PATed                                                        |
| Verify drops                                 | `show asp drop`                                                                                              | No relevant NAT, ACL, VPN filter, certificate, route, crypto, or inspection drops increment      |
| Debug IKEv2 protocol if needed               | `debug crypto ikev2 protocol 2`                                                                              | Debug shows IKEv2 negotiation behavior                                                           |
| Debug IKEv2 platform if needed               | `debug crypto ikev2 platform 2`                                                                              | Debug shows authentication/platform behavior                                                     |
| Debug IPsec if needed                        | `debug crypto ipsec 127`                                                                                     | Debug shows IPsec proposal/data-plane behavior                                                   |
| Stop debugs                                  | `undebug all`                                                                                                | Debug output stops                                                                               |

# ASA_Remote_Access_IKEv2_Rollback
| Step | Task                                                         | Device | Command                                                                                                                                                  | Expected Result                                                             |
| ---: | ------------------------------------------------------------ | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
|    1 | Identify IKEv2 config                                        | ASA    | `show running-config crypto ikev2`                                                                                                                       | IKEv2 policies, enablement, and trustpoint are identified                   |
|    2 | Identify WebVPN / AnyConnect config                          | ASA    | `show running-config webvpn`                                                                                                                             | AnyConnect image, profile, enablement, and tunnel-group-list are identified |
|    3 | Identify crypto map config                                   | ASA    | `show running-config crypto map`                                                                                                                         | Dynamic map and static wrapper are identified                               |
|    4 | Identify tunnel group                                        | ASA    | `show running-config tunnel-group <tunnel-group-name>`                                                                                                   | Remote-access tunnel group settings are identified                          |
|    5 | Identify group policy                                        | ASA    | `show running-config group-policy <group-policy-name>`                                                                                                   | Group policy settings are identified                                        |
|    6 | Identify NAT exemption                                       | ASA    | `show nat detail`                                                                                                                                        | Identity NAT rule and objects are identified                                |
|    7 | Enter configuration mode                                     | ASA    | `configure terminal`                                                                                                                                     | ASA enters global configuration mode                                        |
|    8 | Remove crypto map from outside only if no other VPNs use it  | ASA    | `no crypto map <outside-crypto-map-name> interface outside`                                                                                              | Crypto map is detached from outside                                         |
|    9 | Remove static wrapper dynamic map entry                      | ASA    | `no crypto map <outside-crypto-map-name> <static-wrapper-seq> ipsec-isakmp dynamic <dynamic-map-name>`                                                   | Static wrapper no longer references dynamic map                             |
|   10 | Remove dynamic crypto map reverse-route                      | ASA    | `no crypto dynamic-map <dynamic-map-name> <dynamic-seq> set reverse-route`                                                                               | Reverse-route is removed                                                    |
|   11 | Remove dynamic crypto map proposal                           | ASA    | `no crypto dynamic-map <dynamic-map-name> <dynamic-seq> set ikev2 ipsec-proposal <ipsec-proposal-name>`                                                  | IKEv2 proposal is removed from dynamic map                                  |
|   12 | Remove IPsec proposal if unused                              | ASA    | `no crypto ipsec ikev2 ipsec-proposal <ipsec-proposal-name>`                                                                                             | IKEv2 IPsec proposal is removed                                             |
|   13 | Remove tunnel group                                          | ASA    | `clear configure tunnel-group <tunnel-group-name>`                                                                                                       | Remote-access connection profile is removed                                 |
|   14 | Remove DefaultRAGroup edits if used only for lab             | ASA    | `clear configure tunnel-group DefaultRAGroup`                                                                                                            | Default remote-access group is cleared if safe                              |
|   15 | Remove group policy if unused                                | ASA    | `clear configure group-policy <group-policy-name>`                                                                                                       | Group policy is removed if not shared                                       |
|   16 | Remove local VPN user if lab-only                            | ASA    | `no username <username>`                                                                                                                                 | Local user is removed                                                       |
|   17 | Remove AAA server host if lab-only                           | ASA    | `no aaa-server <aaa-server-group> (<aaa-interface>) host <radius-ip>`                                                                                    | RADIUS host is removed                                                      |
|   18 | Remove AAA server group if unused                            | ASA    | `no aaa-server <aaa-server-group> protocol radius`                                                                                                       | AAA server group is removed if unused                                       |
|   19 | Remove address pool                                          | ASA    | `no ip local pool <vpn-pool-name> <start-ip>-<end-ip> mask <mask>`                                                                                       | VPN pool is removed                                                         |
|   20 | Remove split tunnel ACL                                      | ASA    | `clear configure access-list <split-acl>`                                                                                                                | Split tunnel ACL is removed                                                 |
|   21 | Remove VPN filter ACL                                        | ASA    | `clear configure access-list <vpn-filter-acl>`                                                                                                           | VPN filter ACL is removed                                                   |
|   22 | Remove outside ACL entry if added only for this VPN          | ASA    | `no access-list <outside-acl> extended permit ip <vpn-pool-network> <vpn-pool-mask> <inside-network> <inside-mask>`                                      | VPN-specific outside ACL entry is removed                                   |
|   23 | Restore VPN ACL bypass if changed                            | ASA    | `sysopt connection permit-vpn`                                                                                                                           | Decrypted VPN traffic again bypasses interface ACLs                         |
|   24 | Remove NAT exemption                                         | ASA    | `no nat (inside,outside) source static <inside-object> <inside-object> destination static <vpn-pool-object> <vpn-pool-object> no-proxy-arp route-lookup` | Identity NAT exemption is removed                                           |
|   25 | Remove inside object if unused                               | ASA    | `no object network <inside-object>`                                                                                                                      | Inside object is removed                                                    |
|   26 | Remove VPN pool object if unused                             | ASA    | `no object network <vpn-pool-object>`                                                                                                                    | VPN pool object is removed                                                  |
|   27 | Remove AnyConnect profile binding                            | ASA    | `webvpn` then `no anyconnect profiles <profile-name> disk0:/<anyconnect-profile>.xml`                                                                    | AnyConnect profile reference is removed                                     |
|   28 | Remove AnyConnect image reference if lab-only                | ASA    | `webvpn` then `no anyconnect image disk0:/<anyconnect-package>.pkg <sequence>`                                                                           | AnyConnect image reference is removed                                       |
|   29 | Disable AnyConnect only if no AnyConnect services remain     | ASA    | `webvpn` then `no anyconnect enable`                                                                                                                     | AnyConnect client functionality is disabled                                 |
|   30 | Disable WebVPN outside only if no SSL/client services remain | ASA    | `webvpn` then `no enable outside`                                                                                                                        | WebVPN service is disabled on outside                                       |
|   31 | Remove IKEv2 remote-access trustpoint                        | ASA    | `no crypto ikev2 remote-access trustpoint <trustpoint-name>`                                                                                             | IKEv2 RA trustpoint binding is removed                                      |
|   32 | Disable IKEv2 on outside only if no IKEv2 VPNs remain        | ASA    | `no crypto ikev2 enable outside`                                                                                                                         | ASA stops listening for IKEv2 on outside                                    |
|   33 | Remove IKEv2 policy if unused                                | ASA    | `clear configure crypto ikev2 policy <priority>`                                                                                                         | IKEv2 policy is removed                                                     |
|   34 | Remove SSL trustpoint only if unused                         | ASA    | `no ssl trust-point <trustpoint-name> outside`                                                                                                           | Outside SSL certificate binding is removed                                  |
|   35 | Remove trustpoint only if unused                             | ASA    | `no crypto ca trustpoint <trustpoint-name>`                                                                                                              | Trustpoint is removed                                                       |
|   36 | Remove RSA key pair only if unused                           | ASA    | `crypto key zeroize rsa label <rsa-keypair-name>`                                                                                                        | RSA key pair is removed                                                     |
|   37 | Clear IKEv2 SAs                                              | ASA    | `clear crypto ikev2 sa`                                                                                                                                  | IKEv2 SAs are cleared                                                       |
|   38 | Clear IPsec SAs                                              | ASA    | `clear crypto ipsec sa`                                                                                                                                  | IPsec SAs are cleared                                                       |
|   39 | Clear stale translations                                     | ASA    | `clear xlate`                                                                                                                                            | Old translations are cleared                                                |
|   40 | Clear stale connections                                      | ASA    | `clear conn`                                                                                                                                             | Old connection state is cleared                                             |
|   41 | Verify rollback                                              | ASA    | `show running-config crypto`                                                                                                                             | Removed IKEv2 RA crypto config no longer appears                            |
|   42 | Save rollback state                                          | ASA    | `write memory`                                                                                                                                           | Rollback is saved                                                           |

# ASA_Remote_Access_IKEv2_Failure_Checks
| Symptom | Command | What Usually Broke |
|---|---|---|
| AnyConnect cannot download profile or package | `show running-config webvpn` | Missing `webvpn enable outside`, missing AnyConnect image/profile, missing `anyconnect enable`, or client services not enabled |
| Client services fail | `show running-config crypto ikev2` | Missing `crypto ikev2 enable outside client-services port 443` |
| Certificate warning or connection refusal | `show crypto ca certificates <trustpoint-name>` | Certificate expired, wrong CN/SAN, untrusted CA, or profile host does not match certificate |
| IKEv2 RA does not use expected certificate | `show running-config crypto ikev2` and `show running-config ssl` | Missing `crypto ikev2 remote-access trustpoint` or wrong `ssl trust-point` |
| No IKEv2 SA appears | `show crypto ikev2 sa detail` | IKEv2 not enabled, client not reaching outside IP, UDP/500 or UDP/4500 blocked, or wrong profile |
| IKEv2 negotiation starts but fails | `debug crypto ikev2 protocol 2` | IKEv2 policy mismatch, authentication mismatch, bad certificate, or unsupported proposal |
| Authentication fails | `debug crypto ikev2 platform 2` | Wrong local user, AAA failure, RADIUS unreachable, EAP/certificate mismatch, or wrong tunnel group |
| User lands on wrong group policy | `show vpn-sessiondb anyconnect` | Wrong tunnel group, wrong group alias, profile UserGroup mismatch, or AAA policy override |
| Client connects but receives no IP | `show vpn-sessiondb detail anyconnect` | Pool missing, exhausted, not bound to group policy/tunnel group, or AAA assignment issue |
| Client connects but cannot reach inside | `show nat detail`, `show access-list`, `show asp drop` | NAT exemption missing, split tunnel missing, VPN filter, route, ACL, or host firewall issue |
| VPN pool traffic gets PATed | `show xlate` and `show nat detail` | Missing inside-to-VPN-pool identity NAT or broad PAT matched first |
| Split tunnel routes missing | `show running-config group-policy <group-policy-name>` | Missing `split-tunnel-policy tunnelspecified` or wrong split tunnel ACL |
| DNS fails after VPN connects | `show running-config group-policy <group-policy-name>` | DNS server/default domain not pushed or VPN filter blocks DNS |
| IPsec SA exists but counters stay zero | `show crypto ipsec sa` | No interesting traffic, split tunnel route missing, or client route table issue |
| Encrypt counter increments but decrypt does not | `show crypto ipsec sa` | Client not returning traffic, endpoint firewall, upstream NAT/firewall, or NAT-T issue |
| Decrypt counter increments but inside host gets no reply | `show conn`, `show asp drop`, host firewall check | ASA receives VPN traffic but inside route, host firewall, ACL, VPN filter, or return path breaks |
| VPN filter denies traffic | `show access-list <vpn-filter-acl>` | VPN filter is too narrow or missing required protocol/port |
| Outside ACL blocks decrypted VPN traffic | `show run all sysopt` and `show access-list <outside-acl>` | `no sysopt connection permit-vpn` is configured but outside ACL does not permit VPN pool to inside |
| Mobike does not preserve roaming session | `show crypto ikev2 sa detail` | Client did not negotiate Mobike, client/network blocks update, or session rekey broke |
| Web launch works but IKEv2 fails | `show running-config crypto ikev2` | WebVPN/AnyConnect service exists, but IKEv2 policy, trustpoint, dynamic map, or tunnel group is wrong |
| IKEv2 works but SSL fallback is used | AnyConnect client statistics and `show vpn-sessiondb anyconnect` | Client profile prefers SSL, IKEv2 profile missing, or IPsec blocked |
| Debugs flood terminal | `show debug` | Debugs left enabled; run `undebug all` |

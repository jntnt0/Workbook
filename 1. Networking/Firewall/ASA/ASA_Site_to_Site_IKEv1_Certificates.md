
```
# Index
# Source_Basis
# ASA_Site_to_Site_IKEv1_Certificates_Mental_Model
# ASA_Site_to_Site_IKEv1_Certificates_Configuration_Checklist
# ASA_Site_to_Site_IKEv1_Certificates_Skeleton
# ASA_Site_to_Site_IKEv1_Certificates_Manual_Enrollment_Skeleton
# ASA_Site_to_Site_IKEv1_Certificates_Verification_Commands
# ASA_Site_to_Site_IKEv1_Certificates_Rollback
# ASA_Site_to_Site_IKEv1_Certificates_Failure_Checks
```

# ASA_Site_to_Site_IKEv1_Certificates_Mental_Model
| Concept                          | Operational Meaning                                                                                                                                                           |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Certificate-based IKEv1          | The peers authenticate using RSA signatures instead of a preshared key                                                                                                        |
| Trustpoint                       | ASA object that ties the CA, enrollment method, identity certificate, and certificate validation behavior together                                                            |
| Identity certificate             | The ASA certificate used to prove its identity to the peer                                                                                                                    |
| CA certificate                   | The trusted root or issuing CA certificate used to validate peer certificates                                                                                                 |
| RSA key pair                     | Local public/private key material used for certificate enrollment and RSA signature authentication                                                                            |
| `authentication rsa-sig`         | IKEv1 Phase 1 authentication method for certificate-based VPN                                                                                                                 |
| `isakmp identity auto`           | Lets ASA choose identity behavior automatically, useful when some tunnels use PSK and others use certificates                                                                 |
| `crypto map ... set trustpoint`  | Binds the certificate trustpoint to the crypto map used for the VPN                                                                                                           |
| `peer-id-validate cert`          | Requires peer identity validation through the peer certificate                                                                                                                |
| `chain`                          | Sends the full certificate chain to the peer during negotiation                                                                                                               |
| `trust-point` under tunnel group | Associates the tunnel group with the local certificate trustpoint                                                                                                             |
| Crypto ACL                       | Defines the local and remote protected networks that should be encrypted                                                                                                      |
| NAT exemption                    | Prevents VPN traffic from being translated before encryption                                                                                                                  |
| CRL checking                     | Validates whether the peer certificate has been revoked                                                                                                                       |
| Time synchronization             | Required because certificate validity depends on clock accuracy                                                                                                               |
| Blunt rule                       | Certificate VPNs fail from clock problems, missing trustpoints, bad certificate chains, wrong peer identity validation, CRL reachability, or ordinary crypto ACL/NAT mismatch |
# ASA_Site_to_Site_IKEv1_Certificates_Configuration_Checklist
| Step | Task                                                   | Device              | Command                                                                                                                                         | Expected Result                                                                      |
| ---: | ------------------------------------------------------ | ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
|    1 | Confirm ASA interface status                           | ASA                 | `show interface ip brief`                                                                                                                       | Inside and outside interfaces are up/up                                              |
|    2 | Confirm logical interface names                        | ASA                 | `show nameif`                                                                                                                                   | Interfaces are named correctly, usually `inside` and `outside`                       |
|    3 | Confirm outside interface IP                           | ASA                 | `show running-config interface <outside-interface>`                                                                                             | Outside interface has expected VPN termination IP                                    |
|    4 | Confirm inside interface IP                            | ASA                 | `show running-config interface <inside-interface>`                                                                                              | Inside interface has expected LAN gateway IP                                         |
|    5 | Confirm local protected network                        | ASA / Notes         | `<local-network> <local-mask>`                                                                                                                  | Local VPN-protected network is known                                                 |
|    6 | Confirm remote protected network                       | ASA / Notes         | `<remote-network> <remote-mask>`                                                                                                                | Remote VPN-protected network is known                                                |
|    7 | Confirm local peer public IP                           | ASA / Notes         | `<local-peer-public-ip>`                                                                                                                        | Local ASA VPN peer IP is known                                                       |
|    8 | Confirm remote peer public IP                          | ASA / Notes         | `<remote-peer-public-ip>`                                                                                                                       | Remote ASA VPN peer IP is known                                                      |
|    9 | Confirm CA server reachability requirement             | ASA / Notes         | `<ca-server-ip-or-fqdn>`                                                                                                                        | CA enrollment path is known                                                          |
|   10 | Confirm CRL retrieval requirement                      | ASA / Notes         | `<crl-url-or-policy>`                                                                                                                           | CRL behavior is known: required, optional, or disabled for lab only                  |
|   11 | Verify ASA clock                                       | ASA                 | `show clock`                                                                                                                                    | ASA clock is accurate before certificate enrollment                                  |
|   12 | Configure NTP if available                             | ASA                 | `ntp server <ntp-server-ip>`                                                                                                                    | ASA has a stable time source                                                         |
|   13 | Verify route to CA server                              | ASA                 | `show route <ca-server-ip>`                                                                                                                     | ASA can route to the CA server                                                       |
|   14 | Verify route to remote peer                            | ASA                 | `show route <remote-peer-public-ip>`                                                                                                            | ASA can route to the VPN peer public IP                                              |
|   15 | Review current NAT                                     | ASA                 | `show nat detail`                                                                                                                               | Existing NAT rules and hit counts are visible                                        |
|   16 | Review current VPN crypto config                       | ASA                 | `show running-config crypto`                                                                                                                    | Existing IKE/IPsec/crypto map config is visible                                      |
|   17 | Review current tunnel groups                           | ASA                 | `show running-config tunnel-group`                                                                                                              | Existing L2L tunnel groups are identified                                            |
|   18 | Review current certificates                            | ASA                 | `show crypto ca certificates`                                                                                                                   | Existing identity and CA certificates are visible                                    |
|   19 | Enter configuration mode                               | ASA                 | `configure terminal`                                                                                                                            | ASA enters global configuration mode                                                 |
|   20 | Set hostname if not already correct                    | ASA                 | `hostname <asa-hostname>`                                                                                                                       | ASA hostname matches certificate identity plan                                       |
|   21 | Set domain name if not already correct                 | ASA                 | `domain-name <domain-name>`                                                                                                                     | ASA FQDN can be used in certificate enrollment                                       |
|   22 | Generate RSA key pair                                  | ASA                 | `crypto key generate rsa label <rsa-keypair-name> modulus <rsa-modulus>`                                                                        | RSA key pair exists for certificate enrollment                                       |
|   23 | Verify RSA key pair                                    | ASA                 | `show crypto key mypubkey rsa`                                                                                                                  | RSA key pair is present and has expected label/modulus                               |
|   24 | Create trustpoint                                      | ASA                 | `crypto ca trustpoint <trustpoint-name>`                                                                                                        | ASA enters trustpoint configuration mode                                             |
|   25 | Bind RSA key pair to trustpoint if using a named key   | ASA                 | `keypair <rsa-keypair-name>`                                                                                                                    | Trustpoint uses the intended RSA key pair                                            |
|   26 | Configure SCEP enrollment URL                          | ASA                 | `enrollment url <scep-url>`                                                                                                                     | Trustpoint knows where to enroll                                                     |
|   27 | Configure enrollment retry count                       | ASA                 | `enrollment retry count <retry-count>`                                                                                                          | ASA retries enrollment if CA does not respond                                        |
|   28 | Configure enrollment retry period                      | ASA                 | `enrollment retry period <minutes>`                                                                                                             | ASA waits expected time between enrollment retries                                   |
|   29 | Configure certificate FQDN                             | ASA                 | `fqdn <asa-fqdn>`                                                                                                                               | Certificate request includes intended FQDN                                           |
|   30 | Configure certificate subject name                     | ASA                 | `subject-name <subject-name>`                                                                                                                   | Certificate request includes intended DN fields                                      |
|   31 | Configure CRL behavior for production                  | ASA                 | `crl required` or `crl optional`                                                                                                                | ASA validates revocation according to policy                                         |
|   32 | Configure lab-only CRL bypass if needed                | ASA                 | `crl nocheck`                                                                                                                                   | ASA skips CRL checking for lab use only                                              |
|   33 | Exit trustpoint mode                                   | ASA                 | `exit`                                                                                                                                          | ASA returns to global configuration mode                                             |
|   34 | Authenticate CA certificate                            | ASA                 | `crypto ca authenticate <trustpoint-name>`                                                                                                      | ASA imports and trusts the CA certificate after fingerprint acceptance               |
|   35 | Enroll for identity certificate                        | ASA                 | `crypto ca enroll <trustpoint-name>`                                                                                                            | ASA requests and installs its identity certificate                                   |
|   36 | Verify certificate installation                        | ASA                 | `show crypto ca certificates <trustpoint-name>`                                                                                                 | CA certificate and identity certificate show `Status: Available`                     |
|   37 | Create local protected network object                  | ASA                 | `object network <local-object>`                                                                                                                 | ASA enters local object mode                                                         |
|   38 | Define local protected network                         | ASA                 | `subnet <local-network> <local-mask>`                                                                                                           | Local protected network object is defined                                            |
|   39 | Create remote protected network object                 | ASA                 | `object network <remote-object>`                                                                                                                | ASA enters remote object mode                                                        |
|   40 | Define remote protected network                        | ASA                 | `subnet <remote-network> <remote-mask>`                                                                                                         | Remote protected network object is defined                                           |
|   41 | Configure NAT exemption                                | ASA                 | `nat (inside,outside) source static <local-object> <local-object> destination static <remote-object> <remote-object> no-proxy-arp route-lookup` | VPN traffic keeps real source and destination addresses                              |
|   42 | Create crypto ACL remark                               | ASA                 | `access-list <crypto-acl> remark CERT VPN interesting traffic: <local-network> to <remote-network>`                                             | Crypto ACL purpose is documented                                                     |
|   43 | Create crypto ACL permit                               | ASA                 | `access-list <crypto-acl> extended permit ip <local-network> <local-mask> <remote-network> <remote-mask>`                                       | ASA defines interesting traffic for encryption                                       |
|   44 | Configure IKEv1 IPsec transform set                    | ASA                 | `crypto ipsec ikev1 transform-set <transform-set-name> esp-aes-256 esp-sha-hmac`                                                                | Phase 2 transform set exists                                                         |
|   45 | Create crypto map match statement                      | ASA                 | `crypto map <crypto-map-name> <seq> match address <crypto-acl>`                                                                                 | Crypto map references interesting traffic ACL                                        |
|   46 | Set VPN peer                                           | ASA                 | `crypto map <crypto-map-name> <seq> set peer <remote-peer-public-ip>`                                                                           | Crypto map points to remote VPN peer                                                 |
|   47 | Attach IKEv1 transform set                             | ASA                 | `crypto map <crypto-map-name> <seq> set ikev1 transform-set <transform-set-name>`                                                               | Crypto map uses the intended IKEv1 transform set                                     |
|   48 | Bind trustpoint to crypto map                          | ASA                 | `crypto map <crypto-map-name> <seq> set trustpoint <trustpoint-name>`                                                                           | Crypto map uses local ASA certificate for IKE authentication                         |
|   49 | Enable PFS only if both peers require it               | ASA                 | `crypto map <crypto-map-name> <seq> set pfs group2`                                                                                             | PFS is configured only when both sides agree                                         |
|   50 | Apply crypto map to outside interface                  | ASA                 | `crypto map <crypto-map-name> interface outside`                                                                                                | Crypto map is active on the VPN termination interface                                |
|   51 | Configure ISAKMP identity behavior                     | ASA                 | `isakmp identity auto`                                                                                                                          | ASA automatically selects identity behavior                                          |
|   52 | Enable IKEv1 on outside                                | ASA                 | `crypto ikev1 enable outside`                                                                                                                   | ASA listens for IKEv1 on outside                                                     |
|   53 | Create IKEv1 policy                                    | ASA                 | `crypto ikev1 policy <priority>`                                                                                                                | ASA enters IKEv1 policy mode                                                         |
|   54 | Set certificate authentication                         | ASA                 | `authentication rsa-sig`                                                                                                                        | IKEv1 uses RSA signatures instead of preshared keys                                  |
|   55 | Set IKEv1 encryption                                   | ASA                 | `encryption aes-256`                                                                                                                            | Phase 1 encryption matches peer                                                      |
|   56 | Set IKEv1 hash                                         | ASA                 | `hash sha`                                                                                                                                      | Phase 1 hash matches peer                                                            |
|   57 | Set IKEv1 DH group                                     | ASA                 | `group 2`                                                                                                                                       | Phase 1 DH group matches peer                                                        |
|   58 | Set IKEv1 lifetime                                     | ASA                 | `lifetime 86400`                                                                                                                                | Phase 1 lifetime matches peer or acceptable negotiation range                        |
|   59 | Create group policy                                    | ASA                 | `group-policy <group-policy-name> internal`                                                                                                     | Internal group policy exists                                                         |
|   60 | Enter group policy attributes                          | ASA                 | `group-policy <group-policy-name> attributes`                                                                                                   | ASA enters group policy attribute mode                                               |
|   61 | Restrict tunnel protocol to IKEv1                      | ASA                 | `vpn-tunnel-protocol ikev1`                                                                                                                     | Group policy allows IKEv1                                                            |
|   62 | Create L2L tunnel group                                | ASA                 | `tunnel-group <remote-peer-public-ip> type ipsec-l2l`                                                                                           | Peer-specific L2L tunnel group exists                                                |
|   63 | Enter tunnel group general attributes                  | ASA                 | `tunnel-group <remote-peer-public-ip> general-attributes`                                                                                       | ASA enters tunnel group general attributes                                           |
|   64 | Apply default group policy                             | ASA                 | `default-group-policy <group-policy-name>`                                                                                                      | Tunnel group uses intended group policy                                              |
|   65 | Enter IPsec attributes                                 | ASA                 | `tunnel-group <remote-peer-public-ip> ipsec-attributes`                                                                                         | ASA enters tunnel group IPsec attributes                                             |
|   66 | Validate peer identity by certificate                  | ASA                 | `peer-id-validate cert`                                                                                                                         | ASA validates peer identity using the peer certificate                               |
|   67 | Send certificate chain                                 | ASA                 | `chain`                                                                                                                                         | ASA sends the certificate chain to the peer                                          |
|   68 | Apply trustpoint to tunnel group                       | ASA                 | `trust-point <trustpoint-name>`                                                                                                                 | Tunnel group uses intended local certificate trustpoint                              |
|   69 | Confirm no IKEv1 preshared key remains for this tunnel | ASA                 | `show running-config tunnel-group <remote-peer-public-ip>`                                                                                      | Tunnel group uses certificate commands, not `ikev1 pre-shared-key`                   |
|   70 | Save configuration                                     | ASA                 | `write memory`                                                                                                                                  | Configuration is saved                                                               |
|   71 | Verify trustpoint and certificates                     | ASA                 | `show crypto ca certificates <trustpoint-name>`                                                                                                 | CA and identity certificates are available and valid                                 |
|   72 | Verify CRL status if enabled                           | ASA                 | `show crypto ca crls`                                                                                                                           | CRL exists and is current if CRL checking is required                                |
|   73 | Verify NAT exemption                                   | ASA                 | `show nat detail`                                                                                                                               | Identity NAT appears before broad PAT and has correct objects                        |
|   74 | Verify crypto ACL                                      | ASA                 | `show access-list <crypto-acl>`                                                                                                                 | Crypto ACL matches real local and remote networks                                    |
|   75 | Verify crypto map                                      | ASA                 | `show running-config crypto map`                                                                                                                | Crypto map has ACL, peer, transform set, trustpoint, and outside interface           |
|   76 | Verify IKEv1 policy                                    | ASA                 | `show running-config crypto ikev1`                                                                                                              | Policy uses `authentication rsa-sig`                                                 |
|   77 | Verify tunnel group certificate settings               | ASA                 | `show running-config tunnel-group <remote-peer-public-ip>`                                                                                      | Tunnel group has `peer-id-validate cert`, `chain`, and `trust-point`                 |
|   78 | Simulate interesting traffic                           | ASA                 | `packet-tracer input inside icmp <local-host-ip> 8 0 <remote-host-ip> detailed`                                                                 | Packet-tracer matches NAT exemption and crypto policy path                           |
|   79 | Generate real interesting traffic                      | Local Host / Router | `ping <remote-host-ip>`                                                                                                                         | Traffic triggers IKEv1/IPsec negotiation                                             |
|   80 | Verify IKEv1 Phase 1                                   | ASA                 | `show crypto ikev1 sa detail`                                                                                                                   | IKEv1 SA reaches established state, typically `MM_ACTIVE`                            |
|   81 | Verify IPsec Phase 2                                   | ASA                 | `show crypto ipsec sa`                                                                                                                          | Local/remote proxy IDs match expected crypto ACL networks                            |
|   82 | Verify encrypt/decrypt counters                        | ASA                 | `show crypto ipsec sa`                                                                                                                          | Encaps/encrypt and decaps/decrypt counters increment                                 |
|   83 | Verify VPN session summary                             | ASA                 | `show vpn-sessiondb summary`                                                                                                                    | Active IKEv1 site-to-site IPsec session appears                                      |
|   84 | Check drops if traffic fails                           | ASA                 | `show asp drop`                                                                                                                                 | Drop reason points to NAT, ACL, crypto, route, certificate, CRL, or inspection issue |
|   85 | Debug IKEv1 only when needed                           | ASA                 | `debug crypto ikev1 127`                                                                                                                        | Debug shows certificate, Phase 1, identity, or proposal failure                      |
|   86 | Debug PKI only when needed                             | ASA                 | `debug crypto ca transactions` and `debug crypto ca messages`                                                                                   | Debug shows CA, SCEP, enrollment, or CRL retrieval failure                           |
|   87 | Stop debugs                                            | ASA                 | `undebug all`                                                                                                                                   | Debug output stops                                                                   |
```
# ASA_Site_to_Site_IKEv1_Certificates_Skeleton
configure terminal

hostname <asa-hostname>
domain-name <domain-name>

crypto key generate rsa label <rsa-keypair-name> modulus <rsa-modulus>

crypto ca trustpoint <trustpoint-name>
 keypair <rsa-keypair-name>
 enrollment url <scep-url>
 enrollment retry count <retry-count>
 enrollment retry period <minutes>
 fqdn <asa-fqdn>
 subject-name <subject-name>
 crl optional
exit

crypto ca authenticate <trustpoint-name>
crypto ca enroll <trustpoint-name>

object network <local-object>
 subnet <local-network> <local-mask>

object network <remote-object>
 subnet <remote-network> <remote-mask>

nat (inside,outside) source static <local-object> <local-object> destination static <remote-object> <remote-object> no-proxy-arp route-lookup

access-list <crypto-acl> remark CERT VPN interesting traffic from <local-network> to <remote-network>
access-list <crypto-acl> extended permit ip <local-network> <local-mask> <remote-network> <remote-mask>

crypto ipsec ikev1 transform-set <transform-set-name> esp-aes-256 esp-sha-hmac

crypto map <crypto-map-name> <seq> match address <crypto-acl>
crypto map <crypto-map-name> <seq> set peer <remote-peer-public-ip>
crypto map <crypto-map-name> <seq> set ikev1 transform-set <transform-set-name>
crypto map <crypto-map-name> <seq> set trustpoint <trustpoint-name>
crypto map <crypto-map-name> interface outside

isakmp identity auto
crypto ikev1 enable outside
crypto ikev1 policy <priority>
 authentication rsa-sig
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
 peer-id-validate cert
 chain
 trust-point <trustpoint-name>

write memory
```

```
# ASA_Site_to_Site_IKEv1_Certificates_Manual_Enrollment_Skeleton
configure terminal

hostname <asa-hostname>
domain-name <domain-name>

crypto key generate rsa label <rsa-keypair-name> modulus <rsa-modulus>

crypto ca trustpoint <trustpoint-name>
 keypair <rsa-keypair-name>
 enrollment terminal
 fqdn <asa-fqdn>
 subject-name <subject-name>
 crl optional
exit

crypto ca authenticate <trustpoint-name>
<PASTE_BASE64_CA_CERTIFICATE>
quit

crypto ca enroll <trustpoint-name>
<ANSWER_PROMPTS_AND_COPY_CSR_TO_CA>

crypto ca import <trustpoint-name> certificate
<PASTE_BASE64_IDENTITY_CERTIFICATE>
quit

write memory
```

# ASA_Site_to_Site_IKEv1_Certificates_Verification_Commands
| Task                                      | Command                                                                                          | Expected Result                                                                  |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------- |
| Verify ASA clock                          | `show clock`                                                                                     | ASA time falls within certificate validity window                                |
| Verify NTP                                | `show ntp status`                                                                                | ASA has synchronized time if NTP is configured                                   |
| Verify RSA keys                           | `show crypto key mypubkey rsa`                                                                   | RSA key pair exists with expected label and modulus                              |
| Verify CA and identity certificates       | `show crypto ca certificates <trustpoint-name>`                                                  | CA certificate and identity certificate show `Status: Available`                 |
| Verify certificate validity dates         | `show crypto ca certificates <trustpoint-name>`                                                  | Current ASA time is between certificate start and end dates                      |
| Verify certificate trustpoint association | `show crypto ca certificates <trustpoint-name>`                                                  | Certificate output lists expected associated trustpoint                          |
| Verify CRL cache                          | `show crypto ca crls`                                                                            | CRL is present and current if CRL checking is enabled                            |
| Verify trustpoint config                  | `show running-config crypto ca trustpoint <trustpoint-name>`                                     | Enrollment, FQDN, subject name, keypair, and CRL policy are correct              |
| Verify route to CA server                 | `show route <ca-server-ip>`                                                                      | ASA has route to CA/SCEP server                                                  |
| Verify route to peer public IP            | `show route <remote-peer-public-ip>`                                                             | ASA has route to remote VPN peer                                                 |
| Verify NAT exemption                      | `show nat`                                                                                       | Identity NAT appears before broad Dynamic PAT                                    |
| Verify NAT details                        | `show nat detail`                                                                                | Rule shows local-to-local and remote-to-remote static mapping                    |
| Verify crypto ACL                         | `show access-list <crypto-acl>`                                                                  | ACL permits local protected network to remote protected network                  |
| Verify transform set                      | `show running-config crypto ipsec`                                                               | IKEv1 transform set exists                                                       |
| Verify crypto map                         | `show running-config crypto map`                                                                 | Crypto map has ACL, peer, IKEv1 transform set, trustpoint, and outside interface |
| Verify IKEv1 RSA policy                   | `show running-config crypto ikev1`                                                               | IKEv1 policy includes `authentication rsa-sig`                                   |
| Verify tunnel group certificate settings  | `show running-config tunnel-group <remote-peer-public-ip>`                                       | Tunnel group has `peer-id-validate cert`, `chain`, and `trust-point`             |
| Confirm preshared key is not used         | `show running-config tunnel-group <remote-peer-public-ip>`                                       | No `ikev1 pre-shared-key` is used for this tunnel                                |
| Simulate ICMP interesting traffic         | `packet-tracer input inside icmp <local-host-ip> 8 0 <remote-host-ip> detailed`                  | Flow matches NAT exemption and crypto policy                                     |
| Simulate TCP interesting traffic          | `packet-tracer input inside tcp <local-host-ip> <src-port> <remote-host-ip> <dst-port> detailed` | Flow matches NAT exemption and crypto policy                                     |
| Generate real interesting traffic         | `ping <remote-host-ip>`                                                                          | Tunnel negotiation is triggered                                                  |
| Verify IKEv1 Phase 1                      | `show crypto ikev1 sa detail`                                                                    | IKEv1 SA reaches established state, typically `MM_ACTIVE`                        |
| Verify IPsec Phase 2                      | `show crypto ipsec sa`                                                                           | Inbound and outbound ESP SAs exist                                               |
| Verify proxy identities                   | `show crypto ipsec sa`                                                                           | Local and remote identities match crypto ACL networks                            |
| Verify outbound traffic                   | `show crypto ipsec sa`                                                                           | Encaps/encrypt counters increment                                                |
| Verify inbound traffic                    | `show crypto ipsec sa`                                                                           | Decaps/decrypt counters increment                                                |
| Verify VPN session count                  | `show vpn-sessiondb summary`                                                                     | Active site-to-site IKEv1/IPsec session appears                                  |
| Verify connection table                   | `show conn`                                                                                      | Local-to-remote connections appear                                               |
| Verify translations                       | `show xlate`                                                                                     | VPN traffic is not PATed to the outside interface                                |
| Verify drops                              | `show asp drop`                                                                                  | No relevant NAT, ACL, crypto, certificate, CRL, or route drops increment         |
| Debug IKEv1 certificate negotiation       | `debug crypto ikev1 127`                                                                         | Debug shows Phase 1 and certificate authentication behavior                      |
| Debug PKI enrollment or CRL               | `debug crypto ca transactions` and `debug crypto ca messages`                                    | Debug shows CA, SCEP, enrollment, or CRL retrieval behavior                      |
| Stop debugs                               | `undebug all`                                                                                    | Debug output stops                                                               |
# ASA_Site_to_Site_IKEv1_Certificates_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify trustpoint | ASA | `show running-config crypto ca trustpoint <trustpoint-name>` | Trustpoint configuration is identified |
| 2 | Identify installed certificates | ASA | `show crypto ca certificates <trustpoint-name>` | CA and identity certificates are identified |
| 3 | Identify crypto map | ASA | `show running-config crypto map` | Crypto map name, sequence, peer, transform set, ACL, and trustpoint are identified |
| 4 | Identify tunnel group | ASA | `show running-config tunnel-group <remote-peer-public-ip>` | Certificate-based tunnel group settings are identified |
| 5 | Identify NAT exemption | ASA | `show nat detail` | Identity NAT rule and objects are identified |
| 6 | Enter configuration mode | ASA | `configure terminal` | ASA enters global configuration mode |
| 7 | Remove crypto map from outside interface | ASA | `no crypto map <crypto-map-name> interface outside` | Crypto map is detached from outside |
| 8 | Remove crypto map trustpoint binding | ASA | `no crypto map <crypto-map-name> <seq> set trustpoint <trustpoint-name>` | Trustpoint is removed from crypto map |
| 9 | Remove crypto map transform set binding | ASA | `no crypto map <crypto-map-name> <seq> set ikev1 transform-set <transform-set-name>` | Transform set is removed from crypto map |
| 10 | Remove crypto map peer | ASA | `no crypto map <crypto-map-name> <seq> set peer <remote-peer-public-ip>` | Peer is removed from crypto map |
| 11 | Remove crypto map ACL match | ASA | `no crypto map <crypto-map-name> <seq> match address <crypto-acl>` | Crypto ACL match is removed |
| 12 | Remove crypto ACL | ASA | `clear configure access-list <crypto-acl>` | Crypto ACL is removed |
| 13 | Remove NAT exemption | ASA | `no nat (inside,outside) source static <local-object> <local-object> destination static <remote-object> <remote-object> no-proxy-arp route-lookup` | Identity NAT is removed |
| 14 | Remove transform set | ASA | `no crypto ipsec ikev1 transform-set <transform-set-name>` | IKEv1 transform set is removed |
| 15 | Remove tunnel group | ASA | `clear configure tunnel-group <remote-peer-public-ip>` | L2L tunnel group is removed |
| 16 | Remove group policy if unused | ASA | `clear configure group-policy <group-policy-name>` | Group policy is removed if not shared |
| 17 | Remove IKEv1 policy | ASA | `clear configure crypto ikev1 policy <priority>` | IKEv1 RSA signature policy is removed |
| 18 | Disable IKEv1 on outside only if no other IKEv1 tunnels use it | ASA | `no crypto ikev1 enable outside` | ASA stops listening for IKEv1 on outside |
| 19 | Remove identity certificate if full PKI rollback is intended | ASA | `crypto ca certificate chain <trustpoint-name>` then remove certificate block carefully | Certificate chain is removed only if no other service uses it |
| 20 | Remove trustpoint if unused | ASA | `no crypto ca trustpoint <trustpoint-name>` | Trustpoint is removed |
| 21 | Remove RSA key pair only if unused | ASA | `crypto key zeroize rsa label <rsa-keypair-name>` | RSA key pair is removed |
| 22 | Remove local object if unused | ASA | `no object network <local-object>` | Local object is removed |
| 23 | Remove remote object if unused | ASA | `no object network <remote-object>` | Remote object is removed |
| 24 | Clear IKEv1 SAs | ASA | `clear crypto ikev1 sa` | IKEv1 Phase 1 state is cleared |
| 25 | Clear IPsec SAs | ASA | `clear crypto ipsec sa` | IPsec Phase 2 state is cleared |
| 26 | Clear stale translations | ASA | `clear xlate` | Old NAT translations are cleared |
| 27 | Clear stale connections | ASA | `clear conn address <local-host-ip>` | Old connection state is cleared |
| 28 | Verify rollback | ASA | `show running-config crypto` | Removed certificate VPN crypto config no longer appears |
| 29 | Save rollback state | ASA | `write memory` | Rollback is saved |
# ASA_Site_to_Site_IKEv1_Certificates_Failure_Checks
| Symptom                                                 | Command                                                       | What Usually Broke                                                                                          |
| ------------------------------------------------------- | ------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| Certificate enrollment fails immediately                | `show route <ca-server-ip>`                                   | ASA cannot reach the CA server                                                                              |
| SCEP enrollment fails                                   | `debug crypto ca transactions` and `debug crypto ca messages` | TCP/80 blocked, CA URL wrong, routing broken, or CA server not responding                                   |
| Manual enrollment certificate import fails              | `show crypto ca certificates`                                 | Wrong certificate format, DER instead of Base64, malformed paste, or wrong trustpoint                       |
| Certificate says not valid yet                          | `show clock` and `show crypto ca certificates`                | ASA clock is wrong or CA issued certificate with future start date                                          |
| Certificate is expired                                  | `show crypto ca certificates <trustpoint-name>`               | Identity or CA certificate validity period has ended                                                        |
| Peer certificate authentication fails                   | `debug crypto ikev1 127`                                      | Wrong CA trust, missing CA certificate, expired certificate, bad chain, wrong peer identity, or CRL problem |
| IKEv1 Phase 1 never reaches established state           | `show crypto ikev1 sa detail`                                 | RSA-signature policy mismatch, peer unreachable, certificate validation failure, or proposal mismatch       |
| Debug shows proposal mismatch                           | `debug crypto ikev1 127`                                      | Phase 1 encryption, hash, DH group, lifetime, or authentication method mismatch                             |
| Debug shows certificate processing then failure         | `debug crypto ikev1 127` and `debug crypto ca messages`       | Peer certificate validation, chain, CRL, or time problem                                                    |
| Tunnel still expects preshared key                      | `show running-config tunnel-group <remote-peer-public-ip>`    | Old `ikev1 pre-shared-key` remains or IKE policy still uses `authentication pre-share`                      |
| Crypto map does not use certificate                     | `show running-config crypto map`                              | Missing `crypto map <map> <seq> set trustpoint <trustpoint-name>`                                           |
| Tunnel group does not use certificate trustpoint        | `show running-config tunnel-group <remote-peer-public-ip>`    | Missing `trust-point <trustpoint-name>` under IPsec attributes                                              |
| Peer identity validation fails                          | `show running-config tunnel-group <remote-peer-public-ip>`    | `peer-id-validate cert` enabled but certificate subject/SAN does not match expectation                      |
| Peer does not receive full chain                        | `show running-config tunnel-group <remote-peer-public-ip>`    | Missing `chain` command or incomplete certificate chain                                                     |
| CRL required but tunnel fails                           | `show crypto ca crls`                                         | ASA cannot retrieve CRL or CRL is expired/unavailable                                                       |
| CRL server unreachable                                  | `show route <crl-server-ip>`                                  | Routing issue, HTTP blocked, LDAP TCP/389 blocked, or wrong CRL URL                                         |
| Phase 1 is up but Phase 2 fails                         | `show crypto ipsec sa` and `debug crypto ipsec 127`           | Transform set, PFS, or crypto ACL/proxy identity mismatch                                                   |
| Encrypt counter increments but decrypt does not         | `show crypto ipsec sa`                                        | Remote side is not returning traffic, peer crypto ACL mismatch, remote NAT issue, or remote firewall issue  |
| Decrypt counter increments but local host gets no reply | `show conn`, `show asp drop`, host firewall checks            | Local ACL, route, host firewall, or return path issue                                                       |
| Encaps/encrypt counters stay zero                       | `show crypto ipsec sa`                                        | No interesting traffic, crypto ACL mismatch, route issue, or traffic is NATed before encryption             |
| NAT exemption has zero hits                             | `show nat detail`                                             | Wrong local object, wrong remote object, wrong interface pair, or broad PAT matched first                   |
| Traffic gets PATed before encryption                    | `packet-tracer input inside ... detailed`                     | Missing NAT exemption or identity NAT is below broad PAT                                                    |
| Crypto ACL hit count is zero                            | `show access-list <crypto-acl>`                               | Traffic does not match interesting traffic ACL                                                              |
| Packet-tracer allows but real traffic fails             | `show conn`, `show xlate`, captures, `show asp drop`          | Stale state, peer-side issue, host firewall, or asymmetric routing                                          |
| Debugs flood the terminal                               | `show debug`                                                  | Debugs left enabled; run `undebug all`                                                                      |

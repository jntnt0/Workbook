
DMVPN_IPsec_Tunnel_Protection.md

# DMVPN_IPsec_Tunnel_Protection_Configuration_Checklist

# Source_Basis

| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | Book 6, Chapter 5, DMVPN | Points DMVPN Phase 1, Phase 2, Phase 3, NHRP, mGRE, and spoke-to-spoke behavior to `All_combined_part3.md` |
| `_KB_INDEX.md` | Book 6, Chapter 13, IPSec VPN | Supports IPsec VPN, IKEv1, crypto maps, GRE over IPsec, VTI, transform sets, and tunnel protection context |
| `All_combined_part3.md` | Book 6, Chapter 5, DMVPN | Supports the requirement that mGRE, NHRP, tunnel IP to NBMA mapping, and overlay routing are stable before adding IPsec |
| `All_combined_part3.md` | Book 6, Chapter 13, IPsec VPN | Supports IPsec transform set, IPsec profile, transport mode, and tunnel protection concepts |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Chapter 20, Securing DMVPN Tunnels | Supports IKEv2 keyring, IKEv2 profile, IPsec transform set, IPsec profile, `tunnel protection ipsec profile`, anti-replay, DPD, NAT keepalive, and verification with `show dmvpn detail` and `show crypto ipsec sa` |

# DMVPN_IPsec_Tunnel_Protection_Mental_Model

| Concept | Operational Meaning |
|---|---|
| IPsec tunnel protection | Applies IPsec directly to the DMVPN tunnel interface |
| DMVPN first | Underlay, tunnel, NHRP, and routing must work before crypto is added |
| Crypto second | IPsec protects a working tunnel; it does not fix a broken DMVPN control plane |
| IKEv2 keyring | Stores pre-shared keys used to authenticate remote DMVPN peers |
| IKEv2 profile | Matches peers, defines authentication, references the keyring, and optionally matches FVRF |
| Transform set | Defines ESP encryption, ESP authentication, and transport mode |
| Transport mode | Preferred for DMVPN because GRE already provides tunnel encapsulation |
| IPsec profile | Binds the transform set and IKEv2 profile together |
| Tunnel protection | Applies the IPsec profile to the tunnel with `tunnel protection ipsec profile` |
| Shared keyword | Used when multiple encrypted DMVPN tunnels terminate on the same transport interface |
| Tunnel key with shared | Required so decrypted traffic maps back to the correct DMVPN tunnel |
| Anti-replay | Protects against duplicate or old encrypted packets |
| DPD | Detects dead IPsec peers, usually placed on spokes |
| NAT keepalive | Keeps NAT translations alive for spokes behind NAT |
| Crypto counters | Encaps, encrypts, decaps, decrypts, and verifies should increase with protected traffic |
| Hard rule | Apply tunnel protection consistently to all routers in the same protected DMVPN cloud |

# DMVPN_IPsec_Tunnel_Protection_Configuration_Checklist

| Step | Task                                                           | Device      | Command                                           | Expected Result                                                                     |
| ---: | -------------------------------------------------------------- | ----------- | ------------------------------------------------- | ----------------------------------------------------------------------------------- |
|    1 | Confirm NBMA reachability first                                | Hub, Spokes | `ping <remote-nbma-ip>`                           | Underlay transport works before IPsec is configured                                 |
|    2 | Confirm tunnel interface state                                 | Hub, Spokes | `show ip interface brief`                         | Tunnel interfaces show `up/up`                                                      |
|    3 | Confirm DMVPN peer state before crypto                         | Hub, Spokes | `show dmvpn`                                      | DMVPN peers show up before encryption                                               |
|    4 | Confirm NHRP mappings before crypto                            | Hub, Spokes | `show ip nhrp`                                    | Hub has spokes and spokes have hub mapping                                          |
|    5 | Confirm overlay routing works before crypto                    | Hub, Spokes | `show ip route`                                   | Remote overlay prefixes are reachable                                               |
|    6 | Test LAN reachability before crypto                            | Hub, Spokes | `ping <remote-lan-ip>`                            | Traffic works before encryption is layered on                                       |
|    7 | Confirm tunnel source and NBMA identity                        | Hub, Spokes | `show running-config interface Tunnel<id>`        | Tunnel source matches expected transport interface or IP                            |
|    8 | Confirm whether Front Door VRF is used                         | Hub, Spokes | `show vrf`                                        | FVRF requirement is known before IKEv2 profile is built                             |
|    9 | Confirm whether `shared` is required                           | Hub, Spokes | `show running-config \| section interface Tunnel` | Design identifies if multiple encrypted DMVPN tunnels share one transport interface |
|   10 | Configure IKEv2 keyring on hub                                 | Hub         | See `Hub_IKEv2_Keyring_Config`                    | Hub has a PSK keyring for DMVPN peers                                               |
|   11 | Configure IKEv2 profile on hub                                 | Hub         | See `Hub_IKEv2_Profile_Config`                    | Hub matches peers and references keyring                                            |
|   12 | Configure IPsec transform set on hub                           | Hub         | See `IPsec_Transform_Set_Config`                  | Hub has ESP transform set in transport mode                                         |
|   13 | Configure IPsec profile on hub                                 | Hub         | See `IPsec_Profile_Config`                        | Hub binds transform set to IKEv2 profile                                            |
|   14 | Configure anti-replay window on hub if required                | Hub         | See `IPsec_Replay_Window_Config`                  | Hub tolerates expected packet reordering                                            |
|   15 | Configure IKEv2 keyring on spokes                              | Spokes      | See `Spoke_IKEv2_Keyring_Config`                  | Spokes have matching PSK keyring                                                    |
|   16 | Configure IKEv2 profile on spokes                              | Spokes      | See `Spoke_IKEv2_Profile_Config`                  | Spokes match peers and reference keyring                                            |
|   17 | Configure DPD on spokes                                        | Spokes      | See `Spoke_IKEv2_Profile_Config`                  | Spokes can detect dead IPsec peers                                                  |
|   18 | Configure IPsec transform set on spokes                        | Spokes      | See `IPsec_Transform_Set_Config`                  | Spokes have matching ESP transform set in transport mode                            |
|   19 | Configure IPsec profile on spokes                              | Spokes      | See `IPsec_Profile_Config`                        | Spokes bind transform set to IKEv2 profile                                          |
|   20 | Configure anti-replay window on spokes if required             | Spokes      | See `IPsec_Replay_Window_Config`                  | Spokes tolerate expected packet reordering                                          |
|   21 | Configure NAT keepalive on spokes behind NAT                   | Spokes      | See `Spoke_NAT_Keepalive_Config`                  | NAT translation is kept alive                                                       |
|   22 | Configure tunnel key before shared tunnel protection if needed | Hub, Spokes | See `Tunnel_Key_For_Shared_Protection`            | Tunnel can be uniquely identified when SADB is shared                               |
|   23 | Apply IPsec tunnel protection on hub                           | Hub         | See `Apply_Tunnel_Protection_Standard`            | Hub tunnel is encrypted                                                             |
|   24 | Apply shared tunnel protection on hub if required              | Hub         | See `Apply_Tunnel_Protection_Shared`              | Hub uses shared SADB safely for multiple encrypted tunnels                          |
|   25 | Apply IPsec tunnel protection on spokes                        | Spokes      | See `Apply_Tunnel_Protection_Standard`            | Spoke tunnels are encrypted                                                         |
|   26 | Apply shared tunnel protection on spokes if required           | Spokes      | See `Apply_Tunnel_Protection_Shared`              | Spokes use shared SADB safely when required                                         |
|   27 | Expect temporary adjacency loss during rollout                 | Hub, Spokes | Observation                                       | Tunnel/routing may drop until both sides have matching crypto                       |
|   28 | Verify IKEv2 profile                                           | Hub, Spokes | `show crypto ikev2 profile`                       | Profile shows keyring, authentication, FVRF if used, and DPD if configured          |
|   29 | Verify transform set                                           | Hub, Spokes | `show crypto ipsec transform-set`                 | Transform set shows transport mode                                                  |
|   30 | Verify IPsec profile                                           | Hub, Spokes | `show crypto ipsec profile`                       | IPsec profile references correct transform set and IKEv2 profile                    |
|   31 | Verify tunnel protection attachment                            | Hub, Spokes | `show running-config interface Tunnel<id>`        | Tunnel has expected `tunnel protection ipsec profile` command                       |
|   32 | Verify IKEv2 SAs                                               | Hub, Spokes | `show crypto ikev2 sa`                            | IKEv2 security associations are active                                              |
|   33 | Verify IPsec SAs                                               | Hub, Spokes | `show crypto ipsec sa`                            | Inbound and outbound IPsec SAs exist                                                |
|   34 | Verify DMVPN crypto detail                                     | Hub, Spokes | `show dmvpn detail`                               | Crypto session status shows active state                                            |
|   35 | Verify DMVPN peer recovery                                     | Hub, Spokes | `show dmvpn`                                      | DMVPN peers return to up state                                                      |
|   36 | Verify NHRP recovery                                           | Hub, Spokes | `show ip nhrp`                                    | NHRP mappings remain present                                                        |
|   37 | Verify routing neighbor recovery                               | Hub, Spokes | Routing neighbor show command                     | EIGRP, OSPF, BGP, or RIP recovers after crypto                                      |
|   38 | Generate protected traffic                                     | Hub, Spokes | `ping <remote-lan-ip>`                            | Traffic crosses encrypted DMVPN                                                     |
|   39 | Verify crypto counters increment                               | Hub, Spokes | `show crypto ipsec sa`                            | Encaps, encrypts, decaps, decrypts, and verifies increase                           |
|   40 | Check for crypto drops                                         | Hub, Spokes | `show crypto ipsec sa`                            | Drop, failed, and replay counters are not increasing                                |
|   41 | Verify MTU after encryption                                    | Hub, Spokes | `ping <remote-lan-ip> size <size> df-bit`         | No unexpected fragmentation or black-hole behavior                                  |
|   42 | Save working protected DMVPN state                             | Hub, Spokes | `copy running-config startup-config`              | Configuration is saved                                                              |

# DMVPN_IPsec_Tunnel_Protection_Skeleton

```text
# Hub_IKEv2_Keyring_Config
conf t
crypto ikev2 keyring <keyring-name>
 peer ANY
  address 0.0.0.0 0.0.0.0
  pre-shared-key <strong-key>
end
```

```text
# Spoke_IKEv2_Keyring_Config
conf t
crypto ikev2 keyring <keyring-name>
 peer ANY
  address 0.0.0.0 0.0.0.0
  pre-shared-key <strong-key>
end
```

```text
# Hub_IKEv2_Profile_Config
conf t
crypto ikev2 profile <ikev2-profile-name>
 match identity remote address 0.0.0.0
 authentication remote pre-share
 authentication local pre-share
 keyring local <keyring-name>
end
```

```text
# Hub_IKEv2_Profile_With_FVRF_Config
conf t
crypto ikev2 profile <ikev2-profile-name>
 match fvrf <front-door-vrf-name>
 match identity remote address 0.0.0.0
 authentication remote pre-share
 authentication local pre-share
 keyring local <keyring-name>
end
```

```text
# Spoke_IKEv2_Profile_Config
conf t
crypto ikev2 profile <ikev2-profile-name>
 match identity remote address 0.0.0.0
 authentication remote pre-share
 authentication local pre-share
 keyring local <keyring-name>
 dpd 40 5 on-demand
end
```

```text
# Spoke_IKEv2_Profile_With_FVRF_Config
conf t
crypto ikev2 profile <ikev2-profile-name>
 match fvrf <front-door-vrf-name>
 match identity remote address 0.0.0.0
 authentication remote pre-share
 authentication local pre-share
 keyring local <keyring-name>
 dpd 40 5 on-demand
end
```

```text
# IPsec_Transform_Set_Config
conf t
crypto ipsec transform-set <transform-set-name> esp-aes 256 esp-sha-hmac
 mode transport
end
```

```text
# IPsec_Profile_Config
conf t
crypto ipsec profile <ipsec-profile-name>
 set transform-set <transform-set-name>
 set ikev2-profile <ikev2-profile-name>
end
```

```text
# Apply_Tunnel_Protection_Standard
conf t
interface Tunnel<id>
 tunnel protection ipsec profile <ipsec-profile-name>
end
```

```text
# Tunnel_Key_For_Shared_Protection
conf t
interface Tunnel<id>
 tunnel key <unique-key-id>
end
```

```text
# Apply_Tunnel_Protection_Shared
conf t
interface Tunnel<id>
 tunnel protection ipsec profile <ipsec-profile-name> shared
end
```

```text
# IPsec_Replay_Window_Config
conf t
crypto ipsec security-association replay window-size 1024
end
```

```text
# Spoke_NAT_Keepalive_Config
conf t
crypto isakmp nat keepalive 20
end
```

```text
# Full_Hub_Example
conf t
crypto ikev2 keyring DMVPN-KEYRING-INET
 peer ANY
  address 0.0.0.0 0.0.0.0
  pre-shared-key <strong-key>
!
crypto ikev2 profile DMVPN-IKE-PROFILE-INET
 match fvrf <front-door-vrf-name>
 match identity remote address 0.0.0.0
 authentication remote pre-share
 authentication local pre-share
 keyring local DMVPN-KEYRING-INET
!
crypto ipsec transform-set AES256/SHA/TRANSPORT esp-aes 256 esp-sha-hmac
 mode transport
!
crypto ipsec profile DMVPN-IPSEC-PROFILE-INET
 set transform-set AES256/SHA/TRANSPORT
 set ikev2-profile DMVPN-IKE-PROFILE-INET
!
interface Tunnel<id>
 tunnel protection ipsec profile DMVPN-IPSEC-PROFILE-INET
!
crypto ipsec security-association replay window-size 1024
end
```

```text
# Full_Spoke_Example
conf t
crypto ikev2 keyring DMVPN-KEYRING-INET
 peer ANY
  address 0.0.0.0 0.0.0.0
  pre-shared-key <strong-key>
!
crypto ikev2 profile DMVPN-IKE-PROFILE-INET
 match fvrf <front-door-vrf-name>
 match identity remote address 0.0.0.0
 authentication remote pre-share
 authentication local pre-share
 keyring local DMVPN-KEYRING-INET
 dpd 40 5 on-demand
!
crypto ipsec transform-set AES256/SHA/TRANSPORT esp-aes 256 esp-sha-hmac
 mode transport
!
crypto ipsec profile DMVPN-IPSEC-PROFILE-INET
 set transform-set AES256/SHA/TRANSPORT
 set ikev2-profile DMVPN-IKE-PROFILE-INET
!
interface Tunnel<id>
 tunnel protection ipsec profile DMVPN-IPSEC-PROFILE-INET
!
crypto ipsec security-association replay window-size 1024
!
crypto isakmp nat keepalive 20
end
```

```text
# Shared_Tunnel_Protection_Reminder
! Use shared only when multiple encrypted DMVPN tunnels terminate on the same transport interface.
! Each protected tunnel must have a unique tunnel key.
! Without unique tunnel keys, decrypted traffic can map to the wrong tunnel.
```

```text
# Crypto_Order_Reminder
! Correct order:
! 1. Prove underlay NBMA reachability.
! 2. Prove DMVPN mGRE and NHRP.
! 3. Prove overlay routing.
! 4. Add IPsec tunnel protection.
! 5. Verify IKEv2 SA, IPsec SA, DMVPN detail, and crypto counters.
```

# DMVPN_IPsec_Tunnel_Protection_Verification_Commands

| Check | Device | Command | Good Output |
|---|---|---|---|
| Underlay reachability | Hub, Spokes | `ping <remote-nbma-ip>` | NBMA pings succeed before crypto |
| DMVPN before crypto | Hub, Spokes | `show dmvpn` | Peers are up before crypto |
| NHRP before crypto | Hub, Spokes | `show ip nhrp` | Expected mappings exist before crypto |
| Overlay routes before crypto | Hub, Spokes | `show ip route` | Remote prefixes are reachable |
| IKEv2 profile | Hub, Spokes | `show crypto ikev2 profile` | Profile shows keyring and authentication |
| Transform set | Hub, Spokes | `show crypto ipsec transform-set` | Transform set uses transport mode |
| IPsec profile | Hub, Spokes | `show crypto ipsec profile` | Profile references correct transform set and IKEv2 profile |
| Tunnel protection config | Hub, Spokes | `show running-config interface Tunnel<id>` | Tunnel protection profile is applied |
| IKEv2 SA | Hub, Spokes | `show crypto ikev2 sa` | IKEv2 SA is active |
| IPsec SA | Hub, Spokes | `show crypto ipsec sa` | ESP SAs exist for protected tunnel traffic |
| DMVPN crypto detail | Hub, Spokes | `show dmvpn detail` | Crypto session status is active |
| Crypto counters | Hub, Spokes | `show crypto ipsec sa` | Encaps, encrypts, decaps, decrypts, and verifies increment |
| Crypto drops | Hub, Spokes | `show crypto ipsec sa` | Drops, replay drops, and verification failures are not increasing |
| DMVPN after crypto | Hub, Spokes | `show dmvpn` | Peers return to up state |
| NHRP after crypto | Hub, Spokes | `show ip nhrp` | NHRP mappings remain present |
| EIGRP recovery if used | Hub, Spokes | `show ip eigrp neighbors` | EIGRP neighbors recover |
| OSPF recovery if used | Hub, Spokes | `show ip ospf neighbor` | OSPF neighbors recover |
| BGP recovery if used | Hub, Spokes | `show ip bgp summary` | BGP peers recover |
| RIP recovery if used | Hub, Spokes | `show ip route rip` | RIP routes remain present |
| Protected traffic test | Hub, Spokes | `ping <remote-lan-ip>` | Ping succeeds and crypto counters increment |
| MTU validation | Hub, Spokes | `ping <remote-lan-ip> size <size> df-bit` | Confirms whether overhead is handled |
| Shared keyword validation | Hub, Spokes | `show running-config interface Tunnel<id>` | Shared tunnel protection has unique tunnel key if used |

# DMVPN_IPsec_Tunnel_Protection_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Remove tunnel protection from spokes first | Spokes | See `Rollback_Tunnel_Protection_Standard` | Spoke tunnel is no longer encrypted |
| 2 | Remove shared tunnel protection from spokes if used | Spokes | See `Rollback_Tunnel_Protection_Shared` | Shared tunnel protection is removed |
| 3 | Remove tunnel protection from hub | Hub | See `Rollback_Tunnel_Protection_Standard` | Hub tunnel is no longer encrypted |
| 4 | Remove shared tunnel protection from hub if used | Hub | See `Rollback_Tunnel_Protection_Shared` | Shared tunnel protection is removed |
| 5 | Clear IKEv2 SAs | Hub, Spokes | `clear crypto ikev2 sa` | IKEv2 security associations are cleared |
| 6 | Clear IPsec SAs | Hub, Spokes | `clear crypto sa` | IPsec security associations are cleared |
| 7 | Remove IPsec profile if no longer used | Hub, Spokes | See `Rollback_IPsec_Profile` | IPsec profile is deleted |
| 8 | Remove transform set if no longer used | Hub, Spokes | See `Rollback_Transform_Set` | Transform set is deleted |
| 9 | Remove IKEv2 profile if no longer used | Hub, Spokes | See `Rollback_IKEv2_Profile` | IKEv2 profile is deleted |
| 10 | Remove IKEv2 keyring if no longer used | Hub, Spokes | See `Rollback_IKEv2_Keyring` | IKEv2 keyring is deleted |
| 11 | Remove NAT keepalive if no longer needed | Spokes | `no crypto isakmp nat keepalive` | NAT keepalive is disabled |
| 12 | Restore default replay window if required | Hub, Spokes | `crypto ipsec security-association replay window-size 64` | Replay window returns to common default behavior |
| 13 | Remove tunnel key only if not required by DMVPN | Hub, Spokes | See `Rollback_Tunnel_Key` | Tunnel key is removed only when safe |
| 14 | Verify crypto SAs are gone | Hub, Spokes | `show crypto ikev2 sa` and `show crypto ipsec sa` | No stale crypto SAs remain |
| 15 | Verify DMVPN still works unencrypted if expected | Hub, Spokes | `show dmvpn` and `show ip nhrp` | DMVPN core remains stable |

```text
# Rollback_Tunnel_Protection_Standard
conf t
interface Tunnel<id>
 no tunnel protection ipsec profile <ipsec-profile-name>
end
```

```text
# Rollback_Tunnel_Protection_Shared
conf t
interface Tunnel<id>
 no tunnel protection ipsec profile <ipsec-profile-name> shared
end
```

```text
# Rollback_IPsec_Profile
conf t
no crypto ipsec profile <ipsec-profile-name>
end
```

```text
# Rollback_Transform_Set
conf t
no crypto ipsec transform-set <transform-set-name>
end
```

```text
# Rollback_IKEv2_Profile
conf t
no crypto ikev2 profile <ikev2-profile-name>
end
```

```text
# Rollback_IKEv2_Keyring
conf t
no crypto ikev2 keyring <keyring-name>
end
```

```text
# Rollback_Tunnel_Key
conf t
interface Tunnel<id>
 no tunnel key <unique-key-id>
end
```

```text
# Clear_Crypto_State
clear crypto ikev2 sa
clear crypto sa
```

# DMVPN_IPsec_Tunnel_Protection_Failure_Checks

| Symptom | Likely Cause | Check | Fix |
|---|---|---|---|
| DMVPN worked before crypto and fails after crypto | Tunnel protection applied on only one side | `show running-config interface Tunnel<id>` | Apply matching tunnel protection to all peers |
| IKEv2 SA does not form | PSK, keyring, profile, FVRF, or peer match problem | `show crypto ikev2 profile` and `show crypto ikev2 sa` | Fix keyring, PSK, match identity, or FVRF |
| IPsec SA does not form but IKEv2 is active | Transform set or IPsec profile mismatch | `show crypto ipsec profile` and `show crypto ipsec transform-set` | Match transform set and IPsec profile |
| Crypto counters do not increment | Traffic is not using the protected tunnel | `show ip route <destination>` and `show crypto ipsec sa` | Fix routing so traffic crosses DMVPN |
| Encaps increase but decaps do not | Return path, peer crypto, ACL, NAT, or transport issue | `show crypto ipsec sa` | Fix reverse path and peer crypto state |
| Decrypt or verify failures increase | Transform, key, replay, or packet corruption issue | `show crypto ipsec sa` | Match crypto settings and inspect replay counters |
| Replay drops increase | Packets arrive out of order, often due to QoS | `show crypto ipsec sa` | Increase replay window to 1024 if platform supports it |
| Spoke behind NAT fails intermittently | NAT mapping expires | `show crypto ikev2 sa` | Configure NAT keepalive on spokes |
| Multiple encrypted tunnels fail on same transport | Missing `shared` or missing unique tunnel key | `show running-config \| section interface Tunnel` | Use `shared` and unique tunnel keys |
| FVRF tunnel fails | IKEv2 profile does not match FVRF | `show crypto ikev2 profile` | Add or correct `match fvrf <vrf-name>` |
| Routing adjacency stays down after crypto | Crypto SA failure or MTU problem | `show crypto ikev2 sa` and `show crypto ipsec sa` | Fix crypto first, then MTU/MSS |
| Large packets fail after encryption | GRE plus IPsec overhead causes fragmentation | `ping <destination> size <size> df-bit` | Tune tunnel MTU and TCP MSS |
| Phase 3 shortcuts fail after crypto | Dynamic spoke-to-spoke IPsec SAs are not forming | `show dmvpn detail` and `show crypto ipsec sa` | Ensure all spokes use matching tunnel protection |
| BGP session fails after crypto | Tunnel source, MTU, or crypto mismatch | `show ip bgp summary` and `show crypto ipsec sa` | Fix crypto and MTU before BGP policy |
| OSPF stuck after crypto | MTU or encrypted path instability | `show ip ospf neighbor` and `show crypto ipsec sa` | Fix crypto counters and MTU |
| EIGRP flaps after crypto | Tunnel instability, crypto SA resets, or MTU | `show ip eigrp neighbors` and `show crypto ikev2 sa` | Stabilize crypto before tuning EIGRP |
| Engineer blames IPsec too early | DMVPN was not stable before encryption | `show dmvpn` and `show ip nhrp` | Remove IPsec, fix DMVPN, then reapply protection |

##### Source_Basis
# DMVPN_IPsec_Tunnel_Protection_Mental_Model
# DMVPN_IPsec_Tunnel_Protection_Configuration_Checklist
# DMVPN_IPsec_Tunnel_Protection_Skeleton
# DMVPN_IPsec_Tunnel_Protection_Verification_Commands
# DMVPN_IPsec_Tunnel_Protection_Rollback
# DMVPN_IPsec_Tunnel_Protection_Failure_Checks
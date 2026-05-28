
QinQ_8021Q_Tunneling.md
# QinQ_8021Q_Tunneling

# QinQ_8021Q_Tunneling_Index

- QinQ_8021Q_Tunneling_Mental_Model
- QinQ_8021Q_Tunneling_Configuration_Checklist
- QinQ_8021Q_Tunneling_Skeleton
- QinQ_8021Q_Tunneling_Verification_Commands
- QinQ_8021Q_Tunneling_Rollback
- QinQ_8021Q_Tunneling_Failure_Checks

# QinQ_8021Q_Tunneling_Mental_Model

| Concept | Operational Meaning |
|---|---|
| QinQ | 802.1Q tunneling where a provider switch adds an outer service-provider VLAN tag around a customer VLAN-tagged frame |
| C-VLAN | Customer VLAN tag carried inside the provider network |
| S-VLAN | Provider service VLAN tag added by the PE tunnel port |
| Double-tagged frame | Frame carries outer S-VLAN tag and inner C-VLAN tag |
| Tunnel port | Customer-facing provider edge port configured with `switchport mode dot1q-tunnel` |
| Access VLAN on tunnel port | The access VLAN on the tunnel port becomes the provider S-VLAN |
| Provider core | Provider switches forward based only on the outer S-VLAN |
| Customer transparency | Customer VLAN tags remain intact across the provider Layer 2 transport |
| PE switch | Provider edge switch that adds or removes the S-VLAN tag |
| P switch | Provider internal switch that transports the S-VLAN like a normal VLAN |
| CE switch | Customer switch that sends and receives normal 802.1Q trunks |
| MTU overhead | QinQ adds an extra 4-byte tag, so provider links may need a larger system MTU |
| Provider STP scope | Provider STP sees the S-VLAN, not every customer C-VLAN |
| Customer STP tunneling | Optional L2 protocol tunneling can carry customer STP, CDP, and VTP across the provider network |
| Native VLAN risk | Untagged customer traffic can be dangerous; provider designs should avoid accidental native VLAN leakage |
| S-VLAN scaling | Each customer or service can be assigned a separate S-VLAN |
| Verification rule | Verify CE trunks, PE tunnel ports, provider S-VLAN transport, MAC learning in the S-VLAN, MTU, and customer VLAN reachability |

# QinQ_8021Q_Tunneling_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm platform supports dot1q tunneling | PE1/PE2 | `show running-config interface <interface>` | Platform accepts `switchport mode dot1q-tunnel` syntax |
| 2 | Confirm physical links are up | CE1/PE1/P/PE2/CE2 | `show interfaces status` | CE-facing and provider-facing links show `connected` |
| 3 | Confirm existing VLAN state before changes | CE1/PE1/P/PE2/CE2 | `show vlan brief` | Existing VLANs are known before configuring QinQ |
| 4 | Confirm existing trunk state before changes | CE1/PE1/P/PE2/CE2 | `show interfaces trunk` | Current trunking state is documented |
| 5 | Confirm current MTU before adding double tags | PE1/P/PE2 | `show system mtu` | Provider switch MTU is known |
| 6 | Increase system MTU if the platform requires baby giant support | PE1/P/PE2 | `configure terminal` then `system mtu 1504` then `end` | Switch is configured to support one extra 802.1Q tag where required |
| 7 | Reload if required by platform for system MTU change | PE1/P/PE2 | `reload` | New system MTU takes effect after reload if required |
| 8 | Verify system MTU after reload if reload was required | PE1/P/PE2 | `show system mtu` | System MTU reflects the intended value |
| 9 | Enter configuration mode on CE1 | CE1 | `configure terminal` | Prompt changes to global configuration mode |
| 10 | Create customer VLAN 10 | CE1 | `vlan 10` | VLAN 10 exists locally |
| 11 | Name customer VLAN 10 | CE1 | `name CUSTOMER_USERS` | VLAN has a readable name |
| 12 | Create customer VLAN 20 | CE1 | `vlan 20` | VLAN 20 exists locally |
| 13 | Name customer VLAN 20 | CE1 | `name CUSTOMER_SERVERS` | VLAN has a readable name |
| 14 | Configure CE1 trunk toward PE1 | CE1 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 15 | Force CE1 port to trunk mode | CE1 | `switchport mode trunk` | CE1 sends 802.1Q tagged customer VLANs |
| 16 | Allow customer VLANs on CE1 trunk | CE1 | `switchport trunk allowed vlan 10,20` | CE1 sends only intended C-VLANs toward PE1 |
| 17 | Bring up CE1 trunk | CE1 | `no shutdown` | Interface is administratively up |
| 18 | Exit CE1 configuration mode | CE1 | `end` | Prompt returns to privileged EXEC mode |
| 19 | Enter configuration mode on CE2 | CE2 | `configure terminal` | Prompt changes to global configuration mode |
| 20 | Create customer VLAN 10 | CE2 | `vlan 10` | VLAN 10 exists locally |
| 21 | Name customer VLAN 10 | CE2 | `name CUSTOMER_USERS` | VLAN has a readable name |
| 22 | Create customer VLAN 20 | CE2 | `vlan 20` | VLAN 20 exists locally |
| 23 | Name customer VLAN 20 | CE2 | `name CUSTOMER_SERVERS` | VLAN has a readable name |
| 24 | Configure CE2 trunk toward PE2 | CE2 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 25 | Force CE2 port to trunk mode | CE2 | `switchport mode trunk` | CE2 sends 802.1Q tagged customer VLANs |
| 26 | Allow customer VLANs on CE2 trunk | CE2 | `switchport trunk allowed vlan 10,20` | CE2 sends only intended C-VLANs toward PE2 |
| 27 | Bring up CE2 trunk | CE2 | `no shutdown` | Interface is administratively up |
| 28 | Exit CE2 configuration mode | CE2 | `end` | Prompt returns to privileged EXEC mode |
| 29 | Enter configuration mode on PE1 | PE1 | `configure terminal` | Prompt changes to global configuration mode |
| 30 | Create provider service VLAN | PE1 | `vlan 100` | S-VLAN 100 exists locally |
| 31 | Name provider service VLAN | PE1 | `name CUSTOMER_A_QINQ_SERVICE` | S-VLAN has a readable name |
| 32 | Configure PE1 customer-facing tunnel port | PE1 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 33 | Add tunnel port description | PE1 | `description QINQ_TUNNEL_TO_CE1` | Interface purpose is visible |
| 34 | Force PE1 customer-facing port to Layer 2 if required | PE1 | `switchport` | Interface is a Layer 2 switchport |
| 35 | Assign PE1 tunnel port to provider S-VLAN | PE1 | `switchport access vlan 100` | Customer traffic is placed into S-VLAN 100 |
| 36 | Configure PE1 port as dot1q tunnel port | PE1 | `switchport mode dot1q-tunnel` | PE1 adds the outer S-VLAN tag to customer tagged frames |
| 37 | Disable DTP toward the customer if supported | PE1 | `switchport nonegotiate` | PE1 does not negotiate trunking with CE1 |
| 38 | Optionally tunnel customer STP | PE1 | `l2protocol-tunnel stp` | Customer STP BPDUs are tunneled instead of processed by provider STP |
| 39 | Optionally tunnel customer CDP | PE1 | `l2protocol-tunnel cdp` | Customer CDP frames are tunneled |
| 40 | Optionally tunnel customer VTP | PE1 | `l2protocol-tunnel vtp` | Customer VTP frames are tunneled |
| 41 | Bring up PE1 tunnel port | PE1 | `no shutdown` | Interface is administratively up |
| 42 | Configure PE1 provider-facing trunk | PE1 | `interface GigabitEthernet0/24` | Prompt changes to interface configuration mode |
| 43 | Add provider trunk description | PE1 | `description PROVIDER_TRUNK_TO_P` | Interface purpose is visible |
| 44 | Force PE1 provider-facing trunk mode | PE1 | `switchport mode trunk` | PE1 transports S-VLANs toward provider core |
| 45 | Allow S-VLAN 100 on PE1 provider trunk | PE1 | `switchport trunk allowed vlan 100` | Only the provider service VLAN is carried |
| 46 | Bring up PE1 provider trunk | PE1 | `no shutdown` | Interface is administratively up |
| 47 | Exit PE1 configuration mode | PE1 | `end` | Prompt returns to privileged EXEC mode |
| 48 | Enter configuration mode on provider core switch | P | `configure terminal` | Prompt changes to global configuration mode |
| 49 | Create provider service VLAN | P | `vlan 100` | S-VLAN 100 exists in the provider core |
| 50 | Name provider service VLAN | P | `name CUSTOMER_A_QINQ_SERVICE` | S-VLAN has a readable name |
| 51 | Configure P trunk toward PE1 | P | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 52 | Force trunk mode toward PE1 | P | `switchport mode trunk` | P transports S-VLANs toward PE1 |
| 53 | Allow S-VLAN 100 toward PE1 | P | `switchport trunk allowed vlan 100` | S-VLAN 100 is allowed |
| 54 | Bring up trunk toward PE1 | P | `no shutdown` | Interface is administratively up |
| 55 | Configure P trunk toward PE2 | P | `interface GigabitEthernet0/2` | Prompt changes to interface configuration mode |
| 56 | Force trunk mode toward PE2 | P | `switchport mode trunk` | P transports S-VLANs toward PE2 |
| 57 | Allow S-VLAN 100 toward PE2 | P | `switchport trunk allowed vlan 100` | S-VLAN 100 is allowed |
| 58 | Bring up trunk toward PE2 | P | `no shutdown` | Interface is administratively up |
| 59 | Exit P configuration mode | P | `end` | Prompt returns to privileged EXEC mode |
| 60 | Enter configuration mode on PE2 | PE2 | `configure terminal` | Prompt changes to global configuration mode |
| 61 | Create provider service VLAN | PE2 | `vlan 100` | S-VLAN 100 exists locally |
| 62 | Name provider service VLAN | PE2 | `name CUSTOMER_A_QINQ_SERVICE` | S-VLAN has a readable name |
| 63 | Configure PE2 provider-facing trunk | PE2 | `interface GigabitEthernet0/24` | Prompt changes to interface configuration mode |
| 64 | Add provider trunk description | PE2 | `description PROVIDER_TRUNK_TO_P` | Interface purpose is visible |
| 65 | Force PE2 provider-facing trunk mode | PE2 | `switchport mode trunk` | PE2 transports S-VLANs toward provider core |
| 66 | Allow S-VLAN 100 on PE2 provider trunk | PE2 | `switchport trunk allowed vlan 100` | Only the provider service VLAN is carried |
| 67 | Bring up PE2 provider trunk | PE2 | `no shutdown` | Interface is administratively up |
| 68 | Configure PE2 customer-facing tunnel port | PE2 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 69 | Add tunnel port description | PE2 | `description QINQ_TUNNEL_TO_CE2` | Interface purpose is visible |
| 70 | Force PE2 customer-facing port to Layer 2 if required | PE2 | `switchport` | Interface is a Layer 2 switchport |
| 71 | Assign PE2 tunnel port to provider S-VLAN | PE2 | `switchport access vlan 100` | Customer traffic is placed into S-VLAN 100 |
| 72 | Configure PE2 port as dot1q tunnel port | PE2 | `switchport mode dot1q-tunnel` | PE2 removes the outer S-VLAN tag toward CE2 and adds it toward provider core |
| 73 | Disable DTP toward the customer if supported | PE2 | `switchport nonegotiate` | PE2 does not negotiate trunking with CE2 |
| 74 | Optionally tunnel customer STP | PE2 | `l2protocol-tunnel stp` | Customer STP BPDUs are tunneled |
| 75 | Optionally tunnel customer CDP | PE2 | `l2protocol-tunnel cdp` | Customer CDP frames are tunneled |
| 76 | Optionally tunnel customer VTP | PE2 | `l2protocol-tunnel vtp` | Customer VTP frames are tunneled |
| 77 | Bring up PE2 tunnel port | PE2 | `no shutdown` | Interface is administratively up |
| 78 | Exit PE2 configuration mode | PE2 | `end` | Prompt returns to privileged EXEC mode |
| 79 | Verify CE trunks | CE1/CE2 | `show interfaces trunk` | CE trunks carry customer VLANs 10 and 20 |
| 80 | Verify PE tunnel port mode | PE1/PE2 | `show interfaces GigabitEthernet0/1 switchport` | Customer-facing ports show dot1q tunnel mode and access VLAN 100 |
| 81 | Verify provider trunks carry S-VLAN | PE1/P/PE2 | `show interfaces trunk` | S-VLAN 100 is allowed and forwarding across provider trunks |
| 82 | Verify S-VLAN exists on provider switches | PE1/P/PE2 | `show vlan brief` | VLAN 100 exists and is active |
| 83 | Verify provider STP state for S-VLAN | PE1/P/PE2 | `show spanning-tree vlan 100` | Provider STP has a safe forwarding path for S-VLAN 100 |
| 84 | Verify MAC learning in the provider S-VLAN | PE1/P/PE2 | `show mac address-table dynamic vlan 100` | Customer MACs are learned inside S-VLAN 100 |
| 85 | Verify customer VLAN 10 end-to-end reachability | CE1/HOST | `ping <remote-vlan-10-host-ip>` | VLAN 10 hosts communicate across QinQ service |
| 86 | Verify customer VLAN 20 end-to-end reachability | CE1/HOST | `ping <remote-vlan-20-host-ip>` | VLAN 20 hosts communicate across QinQ service |
| 87 | Verify provider does not learn customer C-VLANs as local VLANs | PE1/P/PE2 | `show vlan brief` | Provider carries S-VLAN 100, not separate local service treatment for C-VLANs 10 and 20 |
| 88 | Verify no MTU drops after double-tagging | PE1/P/PE2 | `show interfaces counters errors` | No rising giants, drops, or encapsulation errors |
| 89 | Save configuration | CE1/PE1/P/PE2/CE2 | `write memory` | QinQ configuration survives reload |

# QinQ_8021Q_Tunneling_Skeleton

CE1 customer trunk toward PE1:

configure terminal
!
vlan 10
 name CUSTOMER_USERS
vlan 20
 name CUSTOMER_SERVERS
!
interface GigabitEthernet0/1
 description TRUNK_TO_PE1_QINQ
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
!
end
write memory

PE1 customer tunnel port and provider trunk:

configure terminal
!
vlan 100
 name CUSTOMER_A_QINQ_SERVICE
!
interface GigabitEthernet0/1
 description QINQ_TUNNEL_TO_CE1
 switchport
 switchport access vlan 100
 switchport mode dot1q-tunnel
 switchport nonegotiate
 l2protocol-tunnel stp
 l2protocol-tunnel cdp
 l2protocol-tunnel vtp
 no shutdown
!
interface GigabitEthernet0/24
 description PROVIDER_TRUNK_TO_P
 switchport mode trunk
 switchport trunk allowed vlan 100
 no shutdown
!
end
write memory

Provider core switch:

configure terminal
!
vlan 100
 name CUSTOMER_A_QINQ_SERVICE
!
interface GigabitEthernet0/1
 description PROVIDER_TRUNK_TO_PE1
 switchport mode trunk
 switchport trunk allowed vlan 100
 no shutdown
!
interface GigabitEthernet0/2
 description PROVIDER_TRUNK_TO_PE2
 switchport mode trunk
 switchport trunk allowed vlan 100
 no shutdown
!
end
write memory

PE2 provider trunk and customer tunnel port:

configure terminal
!
vlan 100
 name CUSTOMER_A_QINQ_SERVICE
!
interface GigabitEthernet0/24
 description PROVIDER_TRUNK_TO_P
 switchport mode trunk
 switchport trunk allowed vlan 100
 no shutdown
!
interface GigabitEthernet0/1
 description QINQ_TUNNEL_TO_CE2
 switchport
 switchport access vlan 100
 switchport mode dot1q-tunnel
 switchport nonegotiate
 l2protocol-tunnel stp
 l2protocol-tunnel cdp
 l2protocol-tunnel vtp
 no shutdown
!
end
write memory

CE2 customer trunk toward PE2:

configure terminal
!
vlan 10
 name CUSTOMER_USERS
vlan 20
 name CUSTOMER_SERVERS
!
interface GigabitEthernet0/1
 description TRUNK_TO_PE2_QINQ
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
!
end
write memory

MTU baseline if required by platform:

configure terminal
!
system mtu 1504
!
end
write memory
reload

Optional native VLAN tagging hardening if supported:

configure terminal
!
vlan dot1q tag native
!
end
write memory

Optional provider edge hardening on customer-facing tunnel port:

configure terminal
!
interface GigabitEthernet0/1
 switchport nonegotiate
 no cdp enable
 no lldp transmit
 no lldp receive
 no shutdown
!
end
write memory

Expected tag behavior:

CE to PE:
Customer frame enters PE with C-VLAN tag 10 or 20

PE to provider core:
PE adds outer S-VLAN 100

Provider core:
Provider forwards only on S-VLAN 100

PE to remote CE:
Remote PE removes S-VLAN 100 and sends original C-VLAN tag 10 or 20 to CE2

# QinQ_8021Q_Tunneling_Verification_Commands

| Verification Goal | Device | Command | Expected Result |
|---|---|---|---|
| Confirm CE trunk state | CE1/CE2 | `show interfaces trunk` | Customer VLANs 10 and 20 are trunking toward provider edge |
| Confirm CE VLAN database | CE1/CE2 | `show vlan brief` | VLANs 10 and 20 exist and are active |
| Confirm PE S-VLAN database | PE1/PE2 | `show vlan brief` | Provider S-VLAN 100 exists and is active |
| Confirm provider core S-VLAN database | P | `show vlan brief` | Provider S-VLAN 100 exists and is active |
| Confirm PE tunnel port mode | PE1/PE2 | `show interfaces GigabitEthernet0/1 switchport` | Port is in dot1q tunnel mode and access VLAN is 100 |
| Confirm PE customer-facing interface state | PE1/PE2 | `show interfaces GigabitEthernet0/1 status` | Tunnel port is connected |
| Confirm provider trunk state | PE1/P/PE2 | `show interfaces trunk` | S-VLAN 100 is allowed and active across provider trunks |
| Confirm S-VLAN STP state | PE1/P/PE2 | `show spanning-tree vlan 100` | Provider STP forwards S-VLAN 100 along the intended path |
| Confirm no STP inconsistency | PE1/P/PE2 | `show spanning-tree inconsistentports` | No inconsistent ports are listed |
| Confirm MAC learning in S-VLAN | PE1/P/PE2 | `show mac address-table dynamic vlan 100` | Customer MACs are learned through provider S-VLAN 100 |
| Confirm customer VLAN 10 MAC learning | CE1/CE2 | `show mac address-table dynamic vlan 10` | Customer MACs are learned in VLAN 10 on CE switches |
| Confirm customer VLAN 20 MAC learning | CE1/CE2 | `show mac address-table dynamic vlan 20` | Customer MACs are learned in VLAN 20 on CE switches |
| Confirm customer VLAN 10 reachability | HOST/CE | `ping <remote-vlan-10-host-ip>` | VLAN 10 host traffic crosses QinQ service |
| Confirm customer VLAN 20 reachability | HOST/CE | `ping <remote-vlan-20-host-ip>` | VLAN 20 host traffic crosses QinQ service |
| Confirm L2 protocol tunneling config | PE1/PE2 | `show running-config interface GigabitEthernet0/1` | `l2protocol-tunnel` commands are present if intended |
| Confirm provider does not form customer STP adjacency locally | PE1/PE2 | `show spanning-tree vlan 100` | Provider STP operates on S-VLAN 100 |
| Confirm no provider trunk native mismatch | PE1/P/PE2 | `show logging | include NATIVE|native` | No native VLAN mismatch messages appear |
| Confirm no MTU or giants issue | PE1/P/PE2 | `show interfaces counters errors` | Giants, input errors, and drops are not increasing |
| Confirm configured MTU | PE1/P/PE2 | `show system mtu` | MTU supports expected QinQ overhead if required |
| Confirm no MAC flapping | PE1/P/PE2 | `show logging | include MACFLAP|LOOP|SPANTREE` | No loop or MAC flap symptoms appear |
| Confirm final PE tunnel config | PE1/PE2 | `show running-config interface GigabitEthernet0/1` | Tunnel mode, S-VLAN, nonegotiate, and optional protocol tunneling are present |
| Confirm final provider trunk config | PE1/P/PE2 | `show running-config interface GigabitEthernet0/24` | Trunk allows S-VLAN 100 |
| Confirm saved QinQ config | CE1/PE1/P/PE2/CE2 | `show startup-config | include dot1q-tunnel|l2protocol-tunnel|switchport trunk allowed vlan|switchport access vlan` | Startup config contains intended QinQ commands |

# QinQ_8021Q_Tunneling_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Record QinQ state before rollback | PE1/PE2 | `show running-config interface GigabitEthernet0/1` | Tunnel-port config is documented |
| 2 | Record provider trunk state before rollback | PE1/P/PE2 | `show interfaces trunk` | S-VLAN transport state is documented |
| 3 | Record customer trunk state before rollback | CE1/CE2 | `show interfaces trunk` | Customer trunk state is documented |
| 4 | Enter configuration mode on PE1 | PE1 | `configure terminal` | Prompt changes to global configuration mode |
| 5 | Reset PE1 customer-facing tunnel port | PE1 | `default interface GigabitEthernet0/1` | Dot1q tunnel and L2 protocol tunneling config is removed |
| 6 | Reset PE1 provider-facing trunk if lab is being removed | PE1 | `default interface GigabitEthernet0/24` | Provider trunk config is removed |
| 7 | Remove PE1 S-VLAN if no longer needed | PE1 | `no vlan 100` | S-VLAN 100 is deleted locally |
| 8 | Exit PE1 configuration mode | PE1 | `end` | Prompt returns to privileged EXEC mode |
| 9 | Enter configuration mode on provider core switch | P | `configure terminal` | Prompt changes to global configuration mode |
| 10 | Reset P trunk toward PE1 | P | `default interface GigabitEthernet0/1` | Trunk config is removed |
| 11 | Reset P trunk toward PE2 | P | `default interface GigabitEthernet0/2` | Trunk config is removed |
| 12 | Remove P S-VLAN if no longer needed | P | `no vlan 100` | S-VLAN 100 is deleted locally |
| 13 | Exit P configuration mode | P | `end` | Prompt returns to privileged EXEC mode |
| 14 | Enter configuration mode on PE2 | PE2 | `configure terminal` | Prompt changes to global configuration mode |
| 15 | Reset PE2 provider-facing trunk if lab is being removed | PE2 | `default interface GigabitEthernet0/24` | Provider trunk config is removed |
| 16 | Reset PE2 customer-facing tunnel port | PE2 | `default interface GigabitEthernet0/1` | Dot1q tunnel and L2 protocol tunneling config is removed |
| 17 | Remove PE2 S-VLAN if no longer needed | PE2 | `no vlan 100` | S-VLAN 100 is deleted locally |
| 18 | Exit PE2 configuration mode | PE2 | `end` | Prompt returns to privileged EXEC mode |
| 19 | Enter configuration mode on CE1 | CE1 | `configure terminal` | Prompt changes to global configuration mode |
| 20 | Reset CE1 trunk toward PE1 if lab is being removed | CE1 | `default interface GigabitEthernet0/1` | Customer trunk config is removed |
| 21 | Remove CE1 test VLAN 10 if no longer needed | CE1 | `no vlan 10` | VLAN 10 is deleted locally |
| 22 | Remove CE1 test VLAN 20 if no longer needed | CE1 | `no vlan 20` | VLAN 20 is deleted locally |
| 23 | Exit CE1 configuration mode | CE1 | `end` | Prompt returns to privileged EXEC mode |
| 24 | Enter configuration mode on CE2 | CE2 | `configure terminal` | Prompt changes to global configuration mode |
| 25 | Reset CE2 trunk toward PE2 if lab is being removed | CE2 | `default interface GigabitEthernet0/1` | Customer trunk config is removed |
| 26 | Remove CE2 test VLAN 10 if no longer needed | CE2 | `no vlan 10` | VLAN 10 is deleted locally |
| 27 | Remove CE2 test VLAN 20 if no longer needed | CE2 | `no vlan 20` | VLAN 20 is deleted locally |
| 28 | Exit CE2 configuration mode | CE2 | `end` | Prompt returns to privileged EXEC mode |
| 29 | Verify tunnel config is removed | PE1/PE2 | `show running-config interface GigabitEthernet0/1` | `switchport mode dot1q-tunnel` is gone |
| 30 | Verify S-VLAN cleanup | PE1/P/PE2 | `show vlan brief` | VLAN 100 is removed if full cleanup was intended |
| 31 | Verify customer VLAN cleanup | CE1/CE2 | `show vlan brief` | VLANs 10 and 20 are removed if full cleanup was intended |
| 32 | Save rollback | CE1/PE1/P/PE2/CE2 | `write memory` | Startup config reflects rollback |

# QinQ_8021Q_Tunneling_Failure_Checks

| Symptom                                      | Likely Cause                                                       | Device            | Command                                                                   | Fix                                                                       |                   |                                                       |                                                                        |
| -------------------------------------------- | ------------------------------------------------------------------ | ----------------- | ------------------------------------------------------------------------- | ------------------------------------------------------------------------- | ----------------- | ----------------------------------------------------- | ---------------------------------------------------------------------- |
| `switchport mode dot1q-tunnel` is rejected   | Platform or image does not support QinQ tunnel ports               | PE1/PE2           | `show running-config interface <interface>`                               | Use a platform that supports dot1q tunneling                              |                   |                                                       |                                                                        |
| Customer VLANs do not cross provider network | PE customer-facing port is trunk instead of tunnel port            | PE1/PE2           | `show interfaces GigabitEthernet0/1 switchport`                           | Configure `switchport mode dot1q-tunnel` and `switchport access vlan 100` |                   |                                                       |                                                                        |
| Customer VLANs do not cross provider network | Provider S-VLAN is missing on provider switches                    | PE1/P/PE2         | `show vlan brief`                                                         | Create S-VLAN 100 on all provider switches                                |                   |                                                       |                                                                        |
| Customer VLANs do not cross provider network | S-VLAN is not allowed on provider trunk                            | PE1/P/PE2         | `show interfaces trunk`                                                   | Allow S-VLAN 100 on all provider trunks                                   |                   |                                                       |                                                                        |
| CE trunk does not come up as trunk           | CE side is not configured as trunk                                 | CE1/CE2           | `show interfaces trunk`                                                   | Configure CE interface as `switchport mode trunk`                         |                   |                                                       |                                                                        |
| CE sends untagged traffic unexpectedly       | Native VLAN is being used on CE trunk                              | CE1/CE2           | `show interfaces trunk`                                                   | Avoid untagged customer traffic or tag native VLAN where supported        |                   |                                                       |                                                                        |
| Provider sees customer VLANs directly        | PE customer-facing port is configured as normal trunk              | PE1/PE2           | `show interfaces GigabitEthernet0/1 switchport`                           | Convert PE customer-facing port to dot1q tunnel mode                      |                   |                                                       |                                                                        |
| Provider learns no customer MACs in S-VLAN   | No traffic, wrong S-VLAN, or broken provider transport             | PE1/P/PE2         | `show mac address-table dynamic vlan 100`                                 | Generate traffic and verify S-VLAN transport                              |                   |                                                       |                                                                        |
| VLAN 10 works but VLAN 20 fails              | CE trunk allowed list missing VLAN 20                              | CE1/CE2           | `show interfaces trunk`                                                   | Add VLAN 20 to the CE trunk allowed list                                  |                   |                                                       |                                                                        |
| Both customer VLANs fail                     | S-VLAN transport is broken                                         | PE1/P/PE2         | `show interfaces trunk`                                                   | Fix provider trunks and S-VLAN 100 forwarding                             |                   |                                                       |                                                                        |
| Customer STP does not pass across QinQ       | L2 protocol tunneling is missing                                   | PE1/PE2           | `show running-config interface GigabitEthernet0/1`                        | Add `l2protocol-tunnel stp` on both PE tunnel ports                       |                   |                                                       |                                                                        |
| Customer CDP does not appear across service  | CDP tunneling is missing or CDP disabled                           | PE1/PE2/CE        | `show running-config interface GigabitEthernet0/1`                        | Add `l2protocol-tunnel cdp` if CDP tunneling is required                  |                   |                                                       |                                                                        |
| Customer VTP does not pass across service    | VTP tunneling is missing                                           | PE1/PE2           | `show running-config interface GigabitEthernet0/1`                        | Add `l2protocol-tunnel vtp` if VTP tunneling is required                  |                   |                                                       |                                                                        |
| Provider switch processes customer STP       | L2 protocol tunneling missing or CE connected to wrong port        | PE1/PE2           | `show spanning-tree vlan 100`                                             | Use tunnel port and `l2protocol-tunnel stp` where required                |                   |                                                       |                                                                        |
| Large frames drop                            | Provider MTU does not support extra tag overhead                   | PE1/P/PE2         | `show interfaces counters errors` and `show system mtu`                   | Increase MTU where required and reload if platform requires               |                   |                                                       |                                                                        |
| Giants or input errors increase              | QinQ overhead exceeds supported frame size                         | PE1/P/PE2         | `show interfaces counters errors`                                         | Increase MTU or reduce payload size                                       |                   |                                                       |                                                                        |
| Provider S-VLAN STP blocks path              | Provider STP topology blocks S-VLAN 100                            | PE1/P/PE2         | `show spanning-tree vlan 100`                                             | Verify intended provider root and forwarding path                         |                   |                                                       |                                                                        |
| MAC flapping appears in S-VLAN               | Provider loop or duplicate customer path                           | PE1/P/PE2         | `show logging                                                             | include MACFLAP                                                           | LOOP              | SPANTREE`                                             | Fix provider STP, remove duplicate attachment, or isolate service path |
| CE sees unexpected STP topology              | Customer STP is tunneled across service                            | CE1/CE2           | `show spanning-tree vlan 10`                                              | Confirm this is intended or disable L2 protocol tunneling                 |                   |                                                       |                                                                        |
| Provider trunk native VLAN mismatch appears  | Provider trunks have different native VLANs                        | PE1/P/PE2         | `show logging                                                             | include NATIVE                                                            | native`           | Match native VLANs or tag native VLAN where supported |                                                                        |
| DTP causes unexpected negotiation            | DTP not disabled toward customer                                   | PE1/PE2           | `show interfaces GigabitEthernet0/1 switchport`                           | Configure `switchport nonegotiate` on PE tunnel ports                     |                   |                                                       |                                                                        |
| CE cannot ping same VLAN remote host         | CE host port, CE trunk, S-VLAN, or provider STP issue              | CE/PE/P           | `show interfaces trunk`, `show vlan brief`, `show spanning-tree vlan 100` | Verify CE VLAN, CE trunk, PE tunnel, provider S-VLAN, and STP             |                   |                                                       |                                                                        |
| Remote CE receives wrong customer VLAN       | Customer VLAN allowed list or CE access mapping is wrong           | CE1/CE2           | `show vlan brief` and `show interfaces trunk`                             | Fix CE VLAN membership and trunk allowed VLANs                            |                   |                                                       |                                                                        |
| S-VLAN 100 exists but is inactive            | No active ports in VLAN 100                                        | PE1/P/PE2         | `show vlan brief`                                                         | Bring up tunnel or trunk ports assigned to S-VLAN 100                     |                   |                                                       |                                                                        |
| Interface is err-disabled                    | BPDU Guard, UDLD, port security, or link-flap protection triggered | PE1/PE2           | `show interfaces status err-disabled`                                     | Fix trigger, then shut/no shut the interface                              |                   |                                                       |                                                                        |
| Config disappears after reload               | Configuration was not saved                                        | CE1/PE1/P/PE2/CE2 | `show startup-config                                                      | include dot1q-tunnel                                                      | l2protocol-tunnel | switchport trunk allowed vlan`                        | Reapply configuration and run `write memory`                           |
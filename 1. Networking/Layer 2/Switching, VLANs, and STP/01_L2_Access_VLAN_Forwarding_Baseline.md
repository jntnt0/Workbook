L2_Access_VLAN_Forwarding_Baseline.md
# L2_Access_VLAN_Forwarding_Baseline

# L2_Access_VLAN_Forwarding_Baseline_Index

- L2_Access_VLAN_Forwarding_Baseline_Mental_Model
- L2_Access_VLAN_Forwarding_Baseline_Configuration_Checklist
- L2_Access_VLAN_Forwarding_Baseline_Skeleton
- L2_Access_VLAN_Forwarding_Baseline_Verification_Commands
- L2_Access_VLAN_Forwarding_Baseline_Rollback
- L2_Access_VLAN_Forwarding_Baseline_Failure_Checks

# L2_Access_VLAN_Forwarding_Baseline_Mental_Model

| Concept | Operational Meaning |
|---|---|
| Access VLAN | A switchport belongs to one untagged VLAN and forwards host traffic only inside that VLAN unless routed elsewhere |
| Broadcast domain | A VLAN is a Layer 2 broadcast domain; hosts in the same VLAN receive broadcasts from each other |
| Access port | A host-facing port that sends and receives untagged frames for one VLAN |
| VLAN database | The VLAN must exist on the switch before the access port assignment is useful |
| MAC learning | The switch learns source MAC addresses per VLAN and uses the MAC table to forward known unicast traffic |
| Unknown unicast flooding | If the destination MAC is unknown, the switch floods the frame inside that VLAN only |
| Same-subnet host test | Two hosts in the same VLAN and same IP subnet should ARP and ping without a router |
| STP forwarding state | The access port must be forwarding, not blocked, inconsistent, err-disabled, or administratively down |
| PortFast | Host-facing access ports should use PortFast so they skip unnecessary STP delay |
| Native VLAN is irrelevant here | Access ports do not tag frames, so native VLAN behavior belongs to trunking, not basic access-port forwarding |

# L2_Access_VLAN_Forwarding_Baseline_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the switch is reachable and interfaces exist | SW1 | `show ip interface brief` | Management or console access works and target interfaces are visible |
| 2 | Check physical link state before configuring VLAN logic | SW1 | `show interfaces status` | Host-facing ports show `connected` or become connected once hosts are attached |
| 3 | Enter global configuration mode | SW1 | `configure terminal` | Prompt changes to `SW1(config)#` |
| 4 | Create the access VLAN | SW1 | `vlan 10` | Switch enters VLAN configuration mode |
| 5 | Name the VLAN | SW1 | `name USERS` | VLAN has a useful operational label |
| 6 | Select the first host-facing port | SW1 | `interface GigabitEthernet0/1` | Prompt changes to interface configuration mode |
| 7 | Force the port to Layer 2 access mode | SW1 | `switchport mode access` | Port is not dynamically negotiating trunk mode |
| 8 | Assign the first host-facing port to the VLAN | SW1 | `switchport access vlan 10` | Untagged host traffic on the port is placed into VLAN 10 |
| 9 | Add a useful interface description | SW1 | `description H1_ACCESS_VLAN_10` | Interface purpose is obvious in show commands |
| 10 | Enable PortFast on the host-facing port | SW1 | `spanning-tree portfast` | Port transitions quickly to forwarding when the host link comes up |
| 11 | Administratively enable the first access port | SW1 | `no shutdown` | Port is not shut down by configuration |
| 12 | Select the second host-facing port | SW1 | `interface GigabitEthernet0/2` | Prompt changes to interface configuration mode |
| 13 | Force the second port to Layer 2 access mode | SW1 | `switchport mode access` | Port is locked as a host access port |
| 14 | Assign the second host-facing port to the same VLAN | SW1 | `switchport access vlan 10` | H1 and H2 are in the same Layer 2 broadcast domain |
| 15 | Add a useful interface description | SW1 | `description H2_ACCESS_VLAN_10` | Interface purpose is obvious in show commands |
| 16 | Enable PortFast on the second host-facing port | SW1 | `spanning-tree portfast` | Port transitions quickly to forwarding when the host link comes up |
| 17 | Administratively enable the second access port | SW1 | `no shutdown` | Port is not shut down by configuration |
| 18 | Exit configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 19 | Save the configuration | SW1 | `write memory` | Configuration survives reload |
| 20 | Verify VLAN creation and access-port membership | SW1 | `show vlan brief` | VLAN 10 exists and target ports appear under VLAN 10 |
| 21 | Verify actual switchport operating mode | SW1 | `show interfaces GigabitEthernet0/1 switchport` | Operational mode is access and access VLAN is 10 |
| 22 | Verify the second switchport operating mode | SW1 | `show interfaces GigabitEthernet0/2 switchport` | Operational mode is access and access VLAN is 10 |
| 23 | Verify access-port STP state | SW1 | `show spanning-tree vlan 10` | Host-facing ports are forwarding, ideally with PortFast edge behavior |
| 24 | Generate host traffic | H1 | `ping 192.0.2.12` | H1 can reach H2 when both hosts are in the same subnet |
| 25 | Confirm MAC learning in the correct VLAN | SW1 | `show mac address-table dynamic vlan 10` | H1 and H2 MAC addresses are learned on the correct access ports |
| 26 | Confirm no VLAN access-map is filtering traffic | SW1 | `show vlan access-map` | No unexpected VACL is blocking same-VLAN traffic |
| 27 | Confirm no port is err-disabled | SW1 | `show interfaces status err-disabled` | No access port is disabled by a protection feature |
| 28 | Confirm final running configuration | SW1 | `show running-config interface GigabitEthernet0/1` | Interface shows access mode, VLAN assignment, description, and PortFast if configured |

# L2_Access_VLAN_Forwarding_Baseline_Skeleton

SW1 baseline:

configure terminal
!
vlan 10
 name USERS
!
interface GigabitEthernet0/1
 description H1_ACCESS_VLAN_10
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 no shutdown
!
interface GigabitEthernet0/2
 description H2_ACCESS_VLAN_10
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 no shutdown
!
end
write memory

Linux host baseline, if using Linux endpoints:

H1:
ip addr flush dev eth0
ip addr add 192.0.2.11/24 dev eth0
ip link set eth0 up
ping 192.0.2.12

H2:
ip addr flush dev eth0
ip addr add 192.0.2.12/24 dev eth0
ip link set eth0 up
ping 192.0.2.11

Router endpoint baseline, if using Cisco router endpoints as hosts:

H1:
configure terminal
interface GigabitEthernet0/0
 ip address 192.0.2.11 255.255.255.0
 no shutdown
end
ping 192.0.2.12

H2:
configure terminal
interface GigabitEthernet0/0
 ip address 192.0.2.12 255.255.255.0
 no shutdown
end
ping 192.0.2.11

# L2_Access_VLAN_Forwarding_Baseline_Verification_Commands

| Verification Goal | Device | Command | Expected Result |
|---|---|---|---|
| Confirm access VLAN exists | SW1 | `show vlan brief` | VLAN 10 is active |
| Confirm access ports are assigned correctly | SW1 | `show vlan brief` | Gi0/1 and Gi0/2 appear under VLAN 10 |
| Confirm physical link state | SW1 | `show interfaces status` | Target ports show `connected` |
| Confirm switchport operational mode | SW1 | `show interfaces GigabitEthernet0/1 switchport` | `Operational Mode: static access` and `Access Mode VLAN: 10` |
| Confirm second switchport operational mode | SW1 | `show interfaces GigabitEthernet0/2 switchport` | `Operational Mode: static access` and `Access Mode VLAN: 10` |
| Confirm STP is not blocking the access ports | SW1 | `show spanning-tree vlan 10` | Access ports are forwarding |
| Confirm MAC learning | SW1 | `show mac address-table dynamic vlan 10` | Host MACs are learned on the expected ports |
| Confirm host ARP resolution | H1 | `show arp` or `ip neigh` | H2 IP resolves to a MAC address |
| Confirm same-VLAN reachability | H1 | `ping 192.0.2.12` | Ping succeeds |
| Confirm no VLAN ACL interference | SW1 | `show vlan access-map` | No unexpected VLAN access-map is active |
| Confirm no port security shutdown | SW1 | `show port-security interface GigabitEthernet0/1` | Port is not secure-shutdown or violation-disabled |
| Confirm no err-disable condition | SW1 | `show interfaces status err-disabled` | Target ports are not err-disabled |
| Confirm final interface config | SW1 | `show running-config interface GigabitEthernet0/1` | Interface is access mode in VLAN 10 |
| Confirm saved config | SW1 | `show startup-config interface GigabitEthernet0/1` | Startup config matches intended access-port settings |

# L2_Access_VLAN_Forwarding_Baseline_Rollback

| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter global configuration mode | SW1 | `configure terminal` | Prompt changes to `SW1(config)#` |
| 2 | Remove first access-port customization | SW1 | `default interface GigabitEthernet0/1` | Interface returns to platform default configuration |
| 3 | Remove second access-port customization | SW1 | `default interface GigabitEthernet0/2` | Interface returns to platform default configuration |
| 4 | Remove the test VLAN if no longer needed | SW1 | `no vlan 10` | VLAN 10 is deleted from the local VLAN database |
| 5 | Exit configuration mode | SW1 | `end` | Prompt returns to privileged EXEC mode |
| 6 | Save rollback | SW1 | `write memory` | Startup configuration reflects rollback |
| 7 | Verify ports returned to default VLAN behavior | SW1 | `show vlan brief` | Ports no longer appear under VLAN 10 |
| 8 | Verify target interfaces are clean | SW1 | `show running-config interface GigabitEthernet0/1` | Interface-specific access VLAN configuration is gone |

# L2_Access_VLAN_Forwarding_Baseline_Failure_Checks

| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| H1 cannot ping H2 | Hosts are in different VLANs | SW1 | `show vlan brief` | Put both access ports in the same VLAN |
| H1 cannot ARP for H2 | VLAN missing, wrong VLAN, or port down | SW1 | `show vlan brief` | Create the VLAN and assign both access ports correctly |
| Port shows `notconnect` | Cable, adapter, or endpoint is down | SW1 | `show interfaces status` | Bring up endpoint NIC, check cable, or use correct CML link |
| Port shows `disabled` | Interface administratively shut | SW1 | `show running-config interface GigabitEthernet0/1` | Apply `no shutdown` |
| Port shows `err-disabled` | Port security, BPDU Guard, link-flap, or protection trigger | SW1 | `show interfaces status err-disabled` | Fix the trigger, then shut/no shut the port |
| Port is trunking unexpectedly | Dynamic negotiation or wrong port mode | SW1 | `show interfaces GigabitEthernet0/1 switchport` | Apply `switchport mode access` |
| VLAN exists but no MACs appear | No traffic sourced from hosts or wrong port/VLAN | SW1 | `show mac address-table dynamic vlan 10` | Generate ping/ARP traffic and verify access VLAN assignment |
| MAC appears on the wrong port | Cabling or topology mismatch | SW1 | `show mac address-table dynamic vlan 10` | Trace cabling or CML links and correct port mapping |
| One host can ARP but ping fails | Host firewall or IP addressing issue | H1/H2 | `ip addr`, `ip neigh`, `ping` | Fix host IP/mask or endpoint firewall |
| Same VLAN works locally but not across another switch | Trunk VLAN not allowed or not forwarding | SW1/SW2 | `show interfaces trunk` | Allow VLAN 10 on trunk and verify STP forwarding |
| Port is blocking | STP state prevents forwarding | SW1 | `show spanning-tree vlan 10` | Check topology, STP guards, and port role |
| Traffic is filtered inside the VLAN | VACL is dropping traffic | SW1 | `show vlan access-map` | Remove or correct the VLAN access-map |
| Access port delay after link-up | PortFast missing on host-facing port | SW1 | `show spanning-tree interface GigabitEthernet0/1 detail` | Add `spanning-tree portfast` |
| VLAN deleted after reload | Config not saved or VTP behavior | SW1 | `show startup-config | section vlan` | Save config and verify VTP mode/design |

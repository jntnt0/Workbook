
IOS_XE_Management_Plane_Reachability_And_Local_Auth.md

IOS_XE_Management_Plane_Reachability_And_Local_Auth

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | NETCONF / RESTCONF / YANG / telemetry / gRPC source map | Places IOS XE programmability and management-plane topics under IOS XE, CiscoPress automation, and ENCOR/ENARSI references |
| `CCNP and CCIE Enterprise Core ENCOR 350-401 Official Cert.md` | Ch26, device access control, local usernames, vty lines, transport input, SSH setup | Supports local username authentication, `login local`, vty behavior, password types, SSH prerequisites, and timeout hardening |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Ch23, Telnet and SSH troubleshooting | Supports management access failure checks with `show line vty`, `show users`, `show ip ssh`, `show ssh`, TCP/22, and TCP/23 validation |
# IOS_XE_Management_Plane_Reachability_And_Local_Auth_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Management plane | The device must have a reachable IP path before Telnet, SSH, NETCONF, RESTCONF, or API tooling can work |
| Local authentication | The device can authenticate management users from its local username database without TACACS+ or RADIUS |
| VTY lines | Logical remote-access lines used by inbound Telnet and SSH sessions |
| Console safety | Create a known local user before applying `login local` to console or vty lines |
| Transport permission | `transport input` decides which protocols are allowed to terminate on the vty lines |
| Secure default | Use SSH for normal management; enable Telnet only for explicit Telnet labs |
| Reachability first | If ping, ARP, route, or gateway reachability fails, authentication troubleshooting is premature |
| Related labs | `telnet-final`, `telnet-server-client-final`, `ssh-server-client-final`, `netconf-final`, `ios-xe-netconf-restconf-final`, `ios-xe-rest-api-final`, `restconf-r1-bridge-final` |
# IOS_XE_Management_Plane_Reachability_And_Local_Auth_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the intended management interface | IOS XE | `show ip interface brief` | Target interface or SVI is visible and can be addressed |
| 2 | Enter global configuration mode | IOS XE | `configure terminal` | Device enters config mode |
| 3 | Set hostname for identity and SSH readiness | IOS XE | `hostname <DEVICE_NAME>` | Prompt changes to the configured hostname |
| 4 | Configure a local privilege user with strong hashing if supported | IOS XE | `username <USER> privilege 15 algorithm-type scrypt secret <SECRET>` | Local user exists with privilege 15 and type 9 secret |
| 5 | Configure fallback local user syntax if `algorithm-type scrypt` is unsupported | IOS XE | `username <USER> privilege 15 secret <SECRET>` | Local user exists with a hashed secret |
| 6 | Configure privileged EXEC protection | IOS XE | `enable secret <ENABLE_SECRET>` | Privileged EXEC mode requires an enable secret when needed |
| 7 | Configure routed management interface | IOS XE router | `interface <MGMT_INTERFACE>` | Device enters interface config mode |
| 8 | Add management interface description | IOS XE router | `description MGMT` | Interface purpose is documented |
| 9 | Assign management IP address | IOS XE router | `ip address <MGMT_IP> <MASK>` | Management interface has an IPv4 address |
| 10 | Enable the management interface | IOS XE router | `no shutdown` | Interface should move toward up/up |
| 11 | Exit interface config mode | IOS XE router | `exit` | Device returns to global config mode |
| 12 | Add default route if this is a routed IOS XE device | IOS XE router | `ip route 0.0.0.0 0.0.0.0 <NEXT_HOP>` | Device has an off-subnet management path |
| 13 | Configure SVI-based management instead if this is an L2 switch lab | IOS XE switch | `interface vlan <MGMT_VLAN>` | Device enters SVI config mode |
| 14 | Assign SVI management IP address | IOS XE switch | `ip address <MGMT_IP> <MASK>` | SVI has an IPv4 management address |
| 15 | Enable the SVI | IOS XE switch | `no shutdown` | SVI should move toward up/up once VLAN is active |
| 16 | Configure L2 switch default gateway if `ip routing` is not enabled | IOS XE switch | `ip default-gateway <GATEWAY_IP>` | Switch has an off-subnet management gateway |
| 17 | Protect console access with local authentication | IOS XE | `line con 0` | Device enters console line config mode |
| 18 | Require local username on console | IOS XE | `login local` | Console uses the local username database |
| 19 | Configure console idle timeout | IOS XE | `exec-timeout 10 0` | Idle console sessions time out after 10 minutes |
| 20 | Enter vty line configuration | IOS XE | `line vty 0 4` | Device enters vty line config mode |
| 21 | Require local username on vty lines | IOS XE | `login local` | Remote CLI sessions use the local username database |
| 22 | Set vty protocol access for secure default | IOS XE | `transport input ssh` | VTY lines accept SSH only |
| 23 | Set vty protocol access for Telnet-specific labs only | IOS XE | `transport input telnet` | VTY lines accept Telnet only |
| 24 | Set vty protocol access for mixed lab testing only | IOS XE | `transport input telnet ssh` | VTY lines accept both Telnet and SSH |
| 25 | Configure vty idle timeout | IOS XE | `exec-timeout 10 0` | Idle remote sessions time out after 10 minutes |
| 26 | End configuration mode | IOS XE | `end` | Device returns to privileged EXEC mode |
| 27 | Save the baseline | IOS XE | `write memory` | Configuration is saved to startup-config |
# IOS_XE_Management_Plane_Reachability_And_Local_Auth_Skeleton
configure terminal
!
hostname <DEVICE_NAME>
!
username <USER> privilege 15 algorithm-type scrypt secret <SECRET>
! If unsupported, use:
! username <USER> privilege 15 secret <SECRET>
!
enable secret <ENABLE_SECRET>
!
! Routed management interface option
interface <MGMT_INTERFACE>
 description MGMT
 ip address <MGMT_IP> <MASK>
 no shutdown
 exit
!
ip route 0.0.0.0 0.0.0.0 <NEXT_HOP>
!
! L2 switch management SVI option
! interface vlan <MGMT_VLAN>
!  description MGMT_SVI
!  ip address <MGMT_IP> <MASK>
!  no shutdown
!  exit
! ip default-gateway <GATEWAY_IP>
!
line con 0
 login local
 exec-timeout 10 0
!
line vty 0 4
 login local
 transport input ssh
 exec-timeout 10 0
!
end
write memory
# IOS_XE_Management_Plane_Reachability_And_Local_Auth_Verification_Commands
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Verify management interface state | IOS XE | `show ip interface brief` | Management interface shows correct IP and up/up status |
| 2 | Verify interface details | IOS XE | `show interfaces <MGMT_INTERFACE>` | Interface is not administratively down and has no obvious L1/L2 issue |
| 3 | Verify default route on routed device | IOS XE router | `show ip route 0.0.0.0` | Default route points toward expected next hop |
| 4 | Verify default gateway on L2 switch | IOS XE switch | `show running-config | include ip default-gateway` | Expected gateway is configured |
| 5 | Test gateway reachability | IOS XE | `ping <GATEWAY_IP> source <MGMT_IP>` | Ping succeeds |
| 6 | Test client-to-device reachability | Client | `ping <MGMT_IP>` | Client can reach the management IP |
| 7 | Verify local usernames | IOS XE | `show running-config | include ^username` | Expected local user exists |
| 8 | Verify enable secret exists | IOS XE | `show running-config | include ^enable secret` | Enable secret is configured |
| 9 | Verify console and vty authentication | IOS XE | `show running-config | section line` | Console and vty lines show `login local` |
| 10 | Verify allowed vty input protocols | IOS XE | `show line vty 0 | include Allowed` | Allowed input transport matches intended lab protocol |
| 11 | Verify active users | IOS XE | `show users` | Current console, Telnet, or SSH sessions are visible |
| 12 | Verify SSH only when SSH is enabled in the protocol note | IOS XE | `show ip ssh` | SSH version and key status are displayed |
| 13 | Verify active SSH sessions only when SSH is in use | IOS XE | `show ssh` | Active SSH sessions are listed, or no active sessions are reported |
# IOS_XE_Management_Plane_Reachability_And_Local_Auth_Rollback
configure terminal
!
line vty 0 4
 no login local
 transport input none
 no exec-timeout
 exit
!
line con 0
 no login local
 no exec-timeout
 exit
!
no username <USER>
no enable secret
!
interface <MGMT_INTERFACE>
 no ip address
 shutdown
 exit
!
no ip route 0.0.0.0 0.0.0.0 <NEXT_HOP>
!
! If SVI management was used:
! interface vlan <MGMT_VLAN>
!  no ip address
!  shutdown
!  exit
! no ip default-gateway <GATEWAY_IP>
!
end
write memory
# IOS_XE_Management_Plane_Reachability_And_Local_Auth_Failure_Checks
| Symptom | Device | Command | Likely Cause | Corrective Action |
|---|---|---|---|---|
| Client cannot ping management IP | IOS XE | `show ip interface brief` | Interface is down, wrong IP, wrong mask, or wrong interface selected | Correct IP addressing and issue `no shutdown` |
| Device can ping same subnet but not off-subnet | IOS XE router | `show ip route 0.0.0.0` | Missing or wrong default route | Add or correct `ip route 0.0.0.0 0.0.0.0 <NEXT_HOP>` |
| L2 switch cannot reach off-subnet manager | IOS XE switch | `show running-config | include ip default-gateway` | Missing `ip default-gateway` while `ip routing` is not enabled | Add `ip default-gateway <GATEWAY_IP>` |
| SVI stays down/down | IOS XE switch | `show vlan brief` | VLAN does not exist or no active port in that VLAN | Create VLAN and ensure at least one associated port is up |
| Login prompts for password only instead of username | IOS XE | `show running-config | section line vty` | VTY line uses `login` instead of `login local` | Configure `login local` under vty lines |
| SSH login fails after password entry | IOS XE | `show running-config | section line vty` | `login` used instead of `login local`, or no local username exists | Create local user and configure `login local` |
| Telnet or SSH connection refused | IOS XE | `show line vty 0 | include Allowed` | `transport input` does not allow the attempted protocol | Set `transport input ssh`, `transport input telnet`, or `transport input telnet ssh` as required |
| Telnet works but SSH fails | IOS XE | `show ip ssh` | SSH keys, domain name, username, SSH version, or vty transport is missing | Move SSH specifics to SSH note and configure hostname, domain, RSA keys, SSHv2, local user, and `transport input ssh` |
| SSH works locally but not from client | IOS XE and path device | `show access-lists` | ACL or CoPP blocks TCP/22 | Permit TCP/22 from the management source |
| Telnet works locally but not from client | IOS XE and path device | `show access-lists` | ACL or CoPP blocks TCP/23 | Permit TCP/23 only for lab scope |
| Login succeeds but user lands at `>` prompt | IOS XE | `show running-config | include ^username` | User lacks privilege 15 | Add `privilege 15` to the local username or use `enable` |
| Remote access fails after multiple sessions | IOS XE | `show users` | All vty lines are consumed | Clear stale sessions or expand/adjust available vty line range |
| User gets locked out after applying config | IOS XE | Console access | Local user missing or wrong secret | Recover through console and correct local user before reapplying vty restrictions |
##### Source_Basis
# IOS_XE_Management_Plane_Reachability_And_Local_Auth_Mental_Model
# IOS_XE_Management_Plane_Reachability_And_Local_Auth_Configuration_Checklist
# IOS_XE_Management_Plane_Reachability_And_Local_Auth_Skeleton
# IOS_XE_Management_Plane_Reachability_And_Local_Auth_Verification_Commands
# IOS_XE_Management_Plane_Reachability_And_Local_Auth_Rollback
# IOS_XE_Management_Plane_Reachability_And_Local_Auth_Failure_Checks
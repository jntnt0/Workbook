
MPP restricts supported management protocols to explicitly designated interfaces using control-plane host and management-interface <interface> allow <protocols>. Cisco lists SSH, SNMP, HTTP, HTTPS, Telnet, FTP, TFTP, and BEEP as affected MPP protocols.  

Management_Plane_Protection_MPP.md
# Management_Plane_Protection_MPP
# Management_Plane_Protection_MPP_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Management Plane Protection | Restricts which router interfaces may receive management traffic destined to the device |
| Management plane | Traffic used to manage the device itself, such as SSH, SNMP, HTTP, HTTPS, Telnet, FTP, TFTP, and BEEP |
| Not transit filtering | MPP does not filter normal data-plane traffic passing through the router |
| Not CoPP | CoPP rate-limits CPU-bound traffic; MPP controls which interfaces may accept management protocols |
| Not VTY ACL replacement | VTY ACLs restrict source addresses; MPP restricts allowed ingress interfaces for management protocols |
| Designated management interface | Interface explicitly allowed to receive selected management protocols |
| In-band management | Management traffic enters a normal routed interface that also forwards data traffic |
| Out-of-band caveat | Classic IOS MPP documentation focuses on in-band physical interfaces, not all dedicated OOB management ports |
| CEF dependency | IP CEF must be enabled before configuring MPP |
| Protocol-specific allow | Each management interface must list the protocols allowed on that interface |
| Default behavior | MPP is disabled by default |
| Enable behavior | Once an interface is configured as a management interface, nondesignated interfaces stop accepting the affected management protocols |
| Last-interface behavior | Removing the last MPP management interface disables MPP |
| Lab purpose | Prove SSH/SNMP/HTTP access works only through the designated management interface and fails through nondesignated interfaces |
# Management_Plane_Protection_MPP_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify candidate management interface | R1 | `show ip interface brief` | Management-facing interface is up/up and has the expected IP address |
| 2 | Identify nondesignated interfaces to block for management access | R1 | `show ip interface brief` | Data/WAN-facing interfaces are known |
| 3 | Confirm CEF is enabled | R1 | `show ip cef` | CEF table is present and populated |
| 4 | Enable CEF if missing | R1 | `configure terminal`<br>`ip cef`<br>`end` | CEF is enabled globally |
| 5 | Confirm baseline management reachability before MPP | Management Host | `ssh <management_interface_ip>`<br>`snmpwalk -v2c -c <community> <management_interface_ip>` | Management access works before MPP enforcement |
| 6 | Confirm nondesignated interface is currently reachable before MPP test | Management Host | `ssh <nondesignated_interface_ip>` | Access may work before MPP, proving later failure is caused by MPP |
| 7 | Check existing VTY transport settings | R1 | `show running-config \| section line vty` | VTY lines permit the intended transport, usually SSH |
| 8 | Check existing SSH configuration | R1 | `show ip ssh`<br>`show running-config \| include ip domain-name\|username\|crypto key` | SSH prerequisites are already present |
| 9 | Check existing SNMP configuration if SNMP will be allowed | R1 | `show running-config \| include snmp-server` | SNMP communities, users, groups, or hosts are known |
| 10 | Check existing HTTP/HTTPS configuration if web management will be allowed | R1 | `show running-config \| include ip http` | HTTP or HTTPS server state is known |
| 11 | Check existing MPP configuration | R1 | `show management-interface` | Existing MPP interfaces and protocol counters are known |
| 12 | Enter global configuration mode | R1 | `configure terminal` | CLI enters global configuration mode |
| 13 | Enter control-plane host configuration mode | R1 | `control-plane host` | CLI enters `config-cp-host` mode |
| 14 | Allow SSH and SNMP only on the management interface | R1 | `management-interface <management_interface> allow ssh snmp` | SSH and SNMP management packets are accepted only on the designated interface |
| 15 | Allow HTTPS if secure web management is required | R1 | `management-interface <management_interface> allow ssh snmp https` | SSH, SNMP, and HTTPS are accepted on the designated interface |
| 16 | Allow HTTP only for lab testing if required | R1 | `management-interface <management_interface> allow ssh snmp http https` | HTTP is allowed only where intentionally needed |
| 17 | Add a second management interface if the design requires it | R1 | `management-interface <second_management_interface> allow ssh snmp` | A second interface can accept the specified management protocols |
| 18 | Exit control-plane host mode | R1 | `exit` | CLI returns to global configuration mode |
| 19 | Harden VTY transport to SSH only | R1 | `line vty 0 4`<br>`transport input ssh` | Telnet is not accepted on VTY lines |
| 20 | Apply VTY source ACL if required | R1 | `access-class <vty_acl_name> in` | Only approved management source IPs can open VTY sessions |
| 21 | Exit VTY configuration mode | R1 | `exit` | CLI returns to global configuration mode |
| 22 | Disable HTTP if not required | R1 | `no ip http server` | Plain HTTP management is disabled |
| 23 | Enable HTTPS only if required | R1 | `ip http secure-server` | HTTPS server is enabled for secure web management |
| 24 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 25 | Verify MPP interface configuration | R1 | `show management-interface` | Designated interface and allowed protocols are displayed |
| 26 | Verify per-interface MPP state | R1 | `show management-interface <management_interface>` | Allowed protocols and packet counters are shown |
| 27 | Verify per-protocol MPP state | R1 | `show management-interface protocol ssh`<br>`show management-interface protocol snmp` | Protocols show the interfaces where they are allowed |
| 28 | Test SSH through allowed management interface | Management Host | `ssh <username>@<management_interface_ip>` | SSH succeeds through the designated management interface |
| 29 | Test SSH through nondesignated interface | Management Host | `ssh <username>@<nondesignated_interface_ip>` | SSH fails through nondesignated interface |
| 30 | Test SNMP through allowed management interface | Management Host | `snmpwalk -v2c -c <community> <management_interface_ip>` | SNMP succeeds through the designated management interface if SNMP is configured |
| 31 | Test SNMP through nondesignated interface | Management Host | `snmpwalk -v2c -c <community> <nondesignated_interface_ip>` | SNMP fails through nondesignated interface |
| 32 | Verify MPP counters after testing | R1 | `show management-interface` | Processed or dropped counters increment based on test direction |
| 33 | Verify logs for MPP events | R1 | `show logging` | MPP enable, disable, or interface failure messages appear if generated |
| 34 | Save configuration | R1 | `write memory` | MPP configuration is saved |
# Management_Plane_Protection_MPP_SSH_Only_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm SSH is working before MPP | Management Host | `ssh <username>@<management_interface_ip>` | SSH succeeds before MPP |
| 2 | Confirm SSH server state | R1 | `show ip ssh` | SSH is enabled |
| 3 | Enter MPP configuration mode | R1 | `configure terminal`<br>`control-plane host` | CLI enters control-plane host mode |
| 4 | Permit SSH only on the management interface | R1 | `management-interface <management_interface> allow ssh` | SSH is accepted only through the designated interface |
| 5 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 6 | Verify SSH MPP mapping | R1 | `show management-interface protocol ssh` | SSH is allowed only on the designated interface |
| 7 | Test SSH to allowed interface | Management Host | `ssh <username>@<management_interface_ip>` | SSH succeeds |
| 8 | Test SSH to nondesignated interface | Management Host | `ssh <username>@<nondesignated_interface_ip>` | SSH fails |
| 9 | Save configuration | R1 | `write memory` | SSH-only MPP policy is saved |
# Management_Plane_Protection_MPP_Skeleton
configure terminal
ip cef
control-plane host
 management-interface <management_interface> allow ssh snmp https
exit
line vty 0 4
 transport input ssh
 access-class <vty_acl_name> in
exit
no ip http server
ip http secure-server
end
write memory
# Management_Plane_Protection_MPP_SSH_Only_Skeleton
configure terminal
ip cef
control-plane host
 management-interface <management_interface> allow ssh
exit
line vty 0 4
 transport input ssh
 access-class <vty_acl_name> in
exit
end
write memory
# Management_Plane_Protection_MPP_SSH_SNMP_Skeleton
configure terminal
ip cef
control-plane host
 management-interface <management_interface> allow ssh snmp
exit
line vty 0 4
 transport input ssh
 access-class <vty_acl_name> in
exit
end
write memory
# Management_Plane_Protection_MPP_Multiple_Interfaces_Skeleton
configure terminal
ip cef
control-plane host
 management-interface <primary_management_interface> allow ssh snmp https
 management-interface <secondary_management_interface> allow ssh snmp
exit
line vty 0 4
 transport input ssh
 access-class <vty_acl_name> in
exit
end
write memory
# Management_Plane_Protection_MPP_Verification_Commands
| Check | Device | Command | Expected Result |
|---|---|---|---|
| CEF state | R1 | `show ip cef` | CEF table is present |
| Interface state | R1 | `show ip interface brief` | Management interface is up/up |
| MPP summary | R1 | `show management-interface` | Designated management interfaces and allowed protocols are displayed |
| MPP interface detail | R1 | `show management-interface <management_interface>` | Allowed protocols and processed counters are shown |
| MPP SSH protocol view | R1 | `show management-interface protocol ssh` | SSH is listed only on intended management interfaces |
| MPP SNMP protocol view | R1 | `show management-interface protocol snmp` | SNMP is listed only on intended management interfaces |
| MPP HTTPS protocol view | R1 | `show management-interface protocol https` | HTTPS is listed only if intentionally allowed |
| Running config MPP | R1 | `show running-config \| section control-plane` | `control-plane host` and `management-interface ... allow ...` are present |
| VTY transport | R1 | `show running-config \| section line vty` | VTY lines use `transport input ssh` |
| SSH server state | R1 | `show ip ssh` | SSH is enabled |
| HTTP server state | R1 | `show running-config \| include ip http` | HTTP/HTTPS state matches the design |
| SNMP state | R1 | `show running-config \| include snmp-server` | SNMP is configured only if needed |
| Allowed SSH test | Management Host | `ssh <username>@<management_interface_ip>` | SSH succeeds through designated management interface |
| Blocked SSH test | Management Host | `ssh <username>@<nondesignated_interface_ip>` | SSH fails through nondesignated interface |
| Allowed SNMP test | Management Host | `snmpwalk -v2c -c <community> <management_interface_ip>` | SNMP succeeds through designated management interface |
| Blocked SNMP test | Management Host | `snmpwalk -v2c -c <community> <nondesignated_interface_ip>` | SNMP fails through nondesignated interface |
| Allowed HTTPS test | Management Host | `curl -k https://<management_interface_ip>` | HTTPS responds only if allowed and enabled |
| Blocked HTTPS test | Management Host | `curl -k https://<nondesignated_interface_ip>` | HTTPS fails through nondesignated interface |
| Log check | R1 | `show logging` | MPP-related syslog messages appear if generated |
# Management_Plane_Protection_MPP_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter control-plane host mode | R1 | `configure terminal`<br>`control-plane host` | CLI enters MPP configuration mode |
| 2 | Remove MPP from the management interface | R1 | `no management-interface <management_interface> allow ssh snmp https` | MPP entry for that interface and protocol set is removed |
| 3 | Remove SSH-only MPP entry if configured that way | R1 | `no management-interface <management_interface> allow ssh` | SSH-only MPP entry is removed |
| 4 | Remove SNMP-only or protocol-specific MPP entry if configured | R1 | `no management-interface <management_interface> allow snmp` | Protocol-specific MPP entry is removed |
| 5 | Remove second management interface if configured | R1 | `no management-interface <second_management_interface> allow ssh snmp` | Second MPP entry is removed |
| 6 | Exit control-plane host mode | R1 | `exit` | CLI returns to global configuration mode |
| 7 | Remove VTY ACL only if it was added for this lab | R1 | `line vty 0 4`<br>`no access-class <vty_acl_name> in` | VTY source ACL is removed |
| 8 | Restore VTY transport only if required by the lab | R1 | `transport input ssh` | SSH-only transport remains the safer default |
| 9 | Exit configuration mode | R1 | `end` | CLI returns to privileged EXEC mode |
| 10 | Confirm MPP removal | R1 | `show management-interface` | Removed management interface entries are absent |
| 11 | Confirm control-plane config cleanup | R1 | `show running-config \| section control-plane` | Removed `management-interface` commands are absent |
| 12 | Test access according to rollback intent | Management Host | `ssh <username>@<nondesignated_interface_ip>` | Access behavior matches the post-rollback policy |
| 13 | Save rollback | R1 | `write memory` | Rollback is saved |
# Management_Plane_Protection_MPP_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Management access is lost after enabling MPP | Wrong interface was designated as the management interface | R1 | `show management-interface`<br>`show ip interface brief` | Console in and correct `management-interface <interface> allow <protocol>` |
| SSH fails through the intended management interface | SSH was not listed in the MPP allow list | R1 | `show management-interface protocol ssh` | Add `management-interface <management_interface> allow ssh` |
| SSH fails through the intended management interface | VTY transport does not allow SSH | R1 | `show running-config \| section line vty` | Configure `transport input ssh` |
| SSH fails through the intended management interface | SSH server prerequisites are missing | R1 | `show ip ssh`<br>`show running-config \| include ip domain-name\|crypto key\|username` | Configure hostname, domain name, local user, and RSA keys |
| SSH fails through the intended management interface | VTY ACL blocks the source | R1 | `show access-lists <vty_acl_name>`<br>`show running-config \| section line vty` | Add the management host or subnet to the VTY ACL |
| SSH still works through nondesignated interface | MPP is not configured or wrong protocol was tested | R1 | `show management-interface protocol ssh` | Configure SSH under MPP for the intended management interface |
| SNMP fails through the intended management interface | SNMP was not listed in the MPP allow list | R1 | `show management-interface protocol snmp` | Add SNMP to the MPP allow list |
| SNMP fails through the intended management interface | SNMP server configuration is missing or source is blocked | R1 | `show running-config \| include snmp-server` | Configure SNMP users, communities, ACLs, or hosts correctly |
| HTTPS fails through the intended management interface | HTTPS was not listed in MPP or HTTPS server is disabled | R1 | `show management-interface protocol https`<br>`show running-config \| include ip http` | Allow HTTPS in MPP and enable `ip http secure-server` |
| HTTP works when it should not | HTTP server is enabled and HTTP is allowed by MPP | R1 | `show running-config \| include ip http server`<br>`show management-interface protocol http` | Remove HTTP from MPP and configure `no ip http server` |
| `management-interface` command is rejected | Platform or IOS image does not support MPP | R1 | `show version` | Use supported IOS feature set or fall back to VTY ACLs, interface ACLs, CoPP, and ZBFW self-zone controls |
| `management-interface` command is rejected | Entered under wrong configuration mode | R1 | `show running-config \| section control-plane` | Enter `control-plane host` first |
| `management-interface` command is rejected | Interface name syntax is wrong for the platform | R1 | `show ip interface brief` | Use the exact interface name shown by the device |
| MPP counters do not increment | Test traffic is not destined to the router itself | R1 | `show management-interface`<br>`show ip cef <destination>` | Test SSH/SNMP/HTTPS to the router interface IP, not through the router |
| MPP counters do not increment | Traffic is blocked before reaching MPP | R1 | `show access-lists`<br>`show running-config interface <interface>` | Check interface ACLs, upstream ACLs, or host firewall |
| Routing protocols break after MPP | Issue is probably unrelated to MPP because routing protocols are not MPP management protocols | R1 | `show ip protocols`<br>`show ip ospf neighbor`<br>`show bgp ipv4 unicast summary` | Troubleshoot CoPP, ACLs, ZBFW self-zone, or routing configuration |
| CoPP counters show drops during management testing | CoPP is policing management traffic before or alongside MPP | R1 | `show policy-map control-plane input` | Adjust CoPP management class rates or matches |
| ZBFW self-zone blocks management | Self-zone policy does not permit management traffic | R1 | `show zone-pair security`<br>`show policy-map type inspect zone-pair` | Add self-zone permit/pass for required management traffic |
| Access still fails after MPP rollback | VTY ACL, AAA, SSH, SNMP, HTTP, CoPP, ZBFW, or upstream filtering still blocks it | R1 | `show running-config \| section line vty`<br>`show aaa servers`<br>`show policy-map control-plane input`<br>`show access-lists` | Troubleshoot the remaining management-plane dependency |
| Access succeeds from wrong source but correct interface | MPP restricts ingress interface, not source IP | R1 | `show management-interface`<br>`show access-lists <vty_acl_name>` | Add VTY ACL, SNMP ACL, HTTP ACL, or upstream ACL to restrict source IP |
# Management_Plane_Protection_MPP_Related_Labs
- management-plane-protection-final
# Management_Plane_Protection_MPP_Index
Management_Plane_Protection_MPP.md
# Management_Plane_Protection_MPP
# Management_Plane_Protection_MPP_Mental_Model
# Management_Plane_Protection_MPP_Configuration_Checklist
# Management_Plane_Protection_MPP_SSH_Only_Checklist
# Management_Plane_Protection_MPP_Skeleton
# Management_Plane_Protection_MPP_SSH_Only_Skeleton
# Management_Plane_Protection_MPP_SSH_SNMP_Skeleton
# Management_Plane_Protection_MPP_Multiple_Interfaces_Skeleton
# Management_Plane_Protection_MPP_Verification_Commands
# Management_Plane_Protection_MPP_Rollback
# Management_Plane_Protection_MPP_Failure_Checks
# Management_Plane_Protection_MPP_Related_Labs
# Management_Plane_Protection_MPP_Index

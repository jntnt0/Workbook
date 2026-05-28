
IP_Source_Guard_DHCP_Snooping_Binding_Enforcement.md
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_Mental_Model
| Concept | Operational Meaning |
|---|---|
| IP Source Guard | Access-layer enforcement feature that blocks packets whose source IP does not match an approved binding |
| Binding table | The trusted source of truth for allowed host IP, MAC, VLAN, and interface mappings |
| DHCP snooping dependency | Dynamic bindings are usually learned from DHCP snooping |
| Static binding | Manually configured binding used when the host has a static IP address |
| Untrusted access port | Host-facing port where DHCP snooping watches client DHCP messages and builds bindings |
| Trusted uplink | Port toward the DHCP server, router, distribution switch, or relay path |
| `ip verify source` | Enforces source IP validation against the binding table |
| `ip verify source port-security` | Enforces source IP and source MAC validation when port security is also used |
| Not a firewall | IP Source Guard does not classify applications or inspect sessions; it validates whether the source identity belongs on that port |
| Common failure point | DHCP snooping is missing, VLAN is not enabled for snooping, uplink is not trusted, or static hosts lack static bindings |
| Lab purpose | Prove DHCP-learned or statically bound hosts pass traffic, while spoofed or unbound source IP traffic is dropped |
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify client access ports, uplinks, and protected VLANs | SW1 | `show interfaces status`<br>`show vlan brief` | Client-facing ports, uplink ports, and VLAN IDs are known |
| 2 | Confirm DHCP path before enabling enforcement | SW1 | `show cdp neighbors`<br>`show lldp neighbors` | Uplink toward router, DHCP server, relay, or distribution switch is identified |
| 3 | Check existing DHCP snooping state | SW1 | `show ip dhcp snooping` | Current global, VLAN, trusted port, and option-82 state is known |
| 4 | Check current binding table | SW1 | `show ip dhcp snooping binding` | Existing dynamic DHCP snooping bindings are visible, if present |
| 5 | Check current IP Source Guard state | SW1 | `show ip verify source` | Current IP Source Guard interface enforcement state is known |
| 6 | Enter global configuration mode | SW1 | `configure terminal` | CLI enters global configuration mode |
| 7 | Enable DHCP snooping globally | SW1 | `ip dhcp snooping` | DHCP snooping feature is enabled globally |
| 8 | Enable DHCP snooping for the protected VLAN | SW1 | `ip dhcp snooping vlan <vlan_id>` | DHCP snooping is active for the target VLAN |
| 9 | Disable DHCP option insertion if the lab topology or DHCP server rejects option 82 | SW1 | `no ip dhcp snooping information option` | Switch does not insert DHCP snooping option 82 |
| 10 | Enter uplink interface configuration mode | SW1 | `interface <uplink_interface>` | CLI enters uplink interface configuration mode |
| 11 | Trust the uplink toward DHCP server or relay path | SW1 | `ip dhcp snooping trust` | DHCP server replies are allowed through the uplink |
| 12 | Exit uplink interface | SW1 | `exit` | CLI returns to global configuration mode |
| 13 | Enter client access interface configuration mode | SW1 | `interface <access_interface>` | CLI enters client-facing interface configuration mode |
| 14 | Force static access mode | SW1 | `switchport mode access` | Interface is no longer negotiating trunking |
| 15 | Assign the protected access VLAN | SW1 | `switchport access vlan <vlan_id>` | Client port belongs to the protected VLAN |
| 16 | Ensure the client port is untrusted for DHCP snooping | SW1 | `no ip dhcp snooping trust` | Client port remains untrusted |
| 17 | Optionally rate-limit DHCP messages on the client port | SW1 | `ip dhcp snooping limit rate <pps>` | DHCP control traffic is rate-limited on the untrusted port |
| 18 | Enable IP Source Guard using DHCP snooping bindings | SW1 | `ip verify source` | Source IP validation is enabled on the access port |
| 19 | Add port-security-backed MAC validation if required | SW1 | `switchport port-security`<br>`switchport port-security maximum 1`<br>`switchport port-security violation restrict`<br>`switchport port-security mac-address sticky`<br>`ip verify source port-security` | Source IP and MAC validation are both enforced on the access port |
| 20 | Bring the access interface up | SW1 | `no shutdown` | Access interface is administratively up |
| 21 | Exit configuration mode | SW1 | `end` | CLI returns to privileged EXEC mode |
| 22 | Renew DHCP lease from the client | Client | `ipconfig /renew`<br>or<br>`dhclient -r && dhclient` | Client receives DHCP address and generates snooping binding |
| 23 | Verify DHCP snooping binding was learned | SW1 | `show ip dhcp snooping binding` | Binding includes client MAC, IP address, VLAN, and interface |
| 24 | Verify IP Source Guard enforcement state | SW1 | `show ip verify source` | Target interface shows IP Source Guard enabled with an active binding |
| 25 | Test valid client traffic | Client | `ping <default_gateway>` | Traffic succeeds when source IP matches binding |
| 26 | Test spoofed source behavior if required | Client | Change client IP to an unbound address in the same VLAN | Spoofed or unbound source IP traffic is dropped |
| 27 | Verify drop or enforcement evidence | SW1 | `show ip verify source`<br>`show ip dhcp snooping binding` | Binding and enforcement state explain why valid traffic passes and spoofed traffic fails |
| 28 | Save configuration | SW1 | `write memory` | DHCP snooping and IP Source Guard configuration is saved |
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_Static_Host_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Identify static host details | SW1 | `show mac address-table interface <access_interface>` | Static host MAC address is known |
| 2 | Confirm the static host interface and VLAN | SW1 | `show interfaces <access_interface> switchport` | Access VLAN and interface are confirmed |
| 3 | Enter global configuration mode | SW1 | `configure terminal` | CLI enters global configuration mode |
| 4 | Configure a static IP Source Guard binding | SW1 | `ip source binding <mac_address> vlan <vlan_id> <ip_address> interface <access_interface>` | Static host is added to the source binding table |
| 5 | Enable IP Source Guard on the static host port | SW1 | `interface <access_interface>`<br>`ip verify source` | Static host source IP is validated against the configured binding |
| 6 | Exit configuration mode | SW1 | `end` | CLI returns to privileged EXEC mode |
| 7 | Verify static source binding | SW1 | `show ip source binding` | Static binding appears with MAC, IP, VLAN, and interface |
| 8 | Test static host traffic | Static Host | `ping <default_gateway>` | Static host passes traffic when source IP matches configured binding |
| 9 | Save configuration | SW1 | `write memory` | Static binding and enforcement config are saved |
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_Skeleton
configure terminal
ip dhcp snooping
ip dhcp snooping vlan <vlan_id>
no ip dhcp snooping information option
interface <uplink_interface>
 ip dhcp snooping trust
exit
interface <access_interface>
 switchport mode access
 switchport access vlan <vlan_id>
 no ip dhcp snooping trust
 ip dhcp snooping limit rate <pps>
 ip verify source
 no shutdown
exit
end
write memory
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_With_Port_Security_Skeleton
configure terminal
ip dhcp snooping
ip dhcp snooping vlan <vlan_id>
no ip dhcp snooping information option
interface <uplink_interface>
 ip dhcp snooping trust
exit
interface <access_interface>
 switchport mode access
 switchport access vlan <vlan_id>
 no ip dhcp snooping trust
 ip dhcp snooping limit rate <pps>
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 ip verify source port-security
 no shutdown
exit
end
write memory
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_Static_Binding_Skeleton
configure terminal
ip dhcp snooping
ip dhcp snooping vlan <vlan_id>
no ip dhcp snooping information option
ip source binding <mac_address> vlan <vlan_id> <ip_address> interface <access_interface>
interface <access_interface>
 switchport mode access
 switchport access vlan <vlan_id>
 no ip dhcp snooping trust
 ip verify source
 no shutdown
exit
end
write memory
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_Verification_Commands
| Check | Device | Command | Expected Result |
|---|---|---|---|
| VLAN exists | SW1 | `show vlan brief` | Protected VLAN exists and contains expected access ports |
| Interface mode | SW1 | `show interfaces <access_interface> switchport` | Access interface is in the expected VLAN |
| DHCP snooping global state | SW1 | `show ip dhcp snooping` | DHCP snooping is enabled globally |
| DHCP snooping VLAN state | SW1 | `show ip dhcp snooping` | Protected VLAN is listed as enabled for DHCP snooping |
| Trusted uplink | SW1 | `show ip dhcp snooping` | Uplink interface is trusted |
| DHCP snooping bindings | SW1 | `show ip dhcp snooping binding` | Dynamic binding exists for client MAC, IP, VLAN, and interface |
| Static source bindings | SW1 | `show ip source binding` | Static binding exists when static hosts are used |
| IP Source Guard state | SW1 | `show ip verify source` | Access interface shows source validation enabled |
| MAC table correlation | SW1 | `show mac address-table interface <access_interface>` | Learned MAC matches DHCP snooping or static binding |
| Port security state | SW1 | `show port-security interface <access_interface>` | Port security is enabled if `ip verify source port-security` is used |
| DHCP snooping statistics | SW1 | `show ip dhcp snooping statistics` | Drops or forwarded DHCP packet counters reflect test traffic |
| Running config check | SW1 | `show running-config interface <access_interface>` | `ip verify source` or `ip verify source port-security` is present |
| Valid source test | Client | `ping <default_gateway>` | Traffic succeeds when source IP matches binding |
| Spoofed source test | Client | Change IP to unbound address and ping gateway | Traffic fails because source IP does not match binding |
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_Rollback
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Enter access interface configuration mode | SW1 | `configure terminal`<br>`interface <access_interface>` | CLI enters target interface configuration mode |
| 2 | Remove IP Source Guard from the access port | SW1 | `no ip verify source` | Source IP validation is disabled on the interface |
| 3 | Remove DHCP snooping rate limit if configured | SW1 | `no ip dhcp snooping limit rate` | DHCP rate limit is removed from the access port |
| 4 | Remove port security if it was only used for IP Source Guard MAC validation | SW1 | `no switchport port-security` | Port security is disabled on the access port |
| 5 | Exit access interface | SW1 | `exit` | CLI returns to global configuration mode |
| 6 | Remove static source binding if configured | SW1 | `no ip source binding <mac_address> vlan <vlan_id> <ip_address> interface <access_interface>` | Static binding is removed |
| 7 | Enter uplink interface configuration mode | SW1 | `interface <uplink_interface>` | CLI enters uplink interface configuration mode |
| 8 | Remove DHCP snooping trust if DHCP snooping is being fully removed | SW1 | `no ip dhcp snooping trust` | Uplink is no longer trusted for DHCP snooping |
| 9 | Exit uplink interface | SW1 | `exit` | CLI returns to global configuration mode |
| 10 | Disable DHCP snooping for the VLAN if no longer needed | SW1 | `no ip dhcp snooping vlan <vlan_id>` | DHCP snooping is removed from the protected VLAN |
| 11 | Disable DHCP snooping globally if no longer used anywhere | SW1 | `no ip dhcp snooping` | DHCP snooping feature is disabled globally |
| 12 | Exit configuration mode | SW1 | `end` | CLI returns to privileged EXEC mode |
| 13 | Clear dynamic DHCP snooping bindings if needed | SW1 | `clear ip dhcp snooping binding *` | Dynamic DHCP snooping bindings are cleared |
| 14 | Save rollback | SW1 | `write memory` | Rollback is saved |
| 15 | Confirm rollback | SW1 | `show ip verify source`<br>`show ip dhcp snooping`<br>`show ip source binding` | IP Source Guard and unwanted DHCP snooping configuration are removed |
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_Failure_Checks
| Symptom | Likely Cause | Device | Command | Fix |
|---|---|---|---|---|
| Client does not get DHCP address | Uplink toward DHCP server is not trusted | SW1 | `show ip dhcp snooping` | Configure `ip dhcp snooping trust` on DHCP server or relay-facing uplink |
| Client does not get DHCP address | DHCP snooping enabled globally but not for the VLAN | SW1 | `show ip dhcp snooping` | Configure `ip dhcp snooping vlan <vlan_id>` |
| Client does not get DHCP address | DHCP option 82 is inserted and upstream DHCP path rejects it | SW1 | `show ip dhcp snooping`<br>`show ip dhcp snooping statistics` | Configure `no ip dhcp snooping information option` if appropriate for the lab |
| IP Source Guard blocks valid client | No DHCP snooping binding exists yet | SW1 | `show ip dhcp snooping binding` | Renew DHCP lease or fix DHCP snooping trust/VLAN configuration |
| Static host loses connectivity | Static host has no static source binding | SW1 | `show ip source binding` | Configure `ip source binding <mac_address> vlan <vlan_id> <ip_address> interface <access_interface>` |
| Static host still fails after binding | Binding has wrong MAC, IP, VLAN, or interface | SW1 | `show ip source binding`<br>`show mac address-table interface <access_interface>` | Correct the static binding |
| `ip verify source port-security` does not behave as expected | Port security is not configured or has wrong secure MAC state | SW1 | `show port-security interface <access_interface>`<br>`show port-security address` | Configure or correct port security before using MAC validation |
| Spoofed IP still passes | IP Source Guard is not enabled on the access interface | SW1 | `show running-config interface <access_interface>`<br>`show ip verify source` | Add `ip verify source` to the client-facing port |
| Spoofed IP still passes | Test traffic is not entering through the protected port | SW1 | `show mac address-table address <client_mac>` | Test from the actual protected interface |
| Valid DHCP client works until reload | DHCP snooping binding database was not persistent and host has not renewed DHCP yet | SW1 | `show ip dhcp snooping binding` | Renew DHCP lease or configure binding database persistence if required |
| DHCP snooping bindings are empty | Client has not sent DHCP traffic through the switch | SW1 | `show ip dhcp snooping binding`<br>`show ip dhcp snooping statistics` | Renew DHCP lease from client |
| DHCP packets are being dropped | Client-facing rate limit too low | SW1 | `show ip dhcp snooping statistics` | Raise or remove `ip dhcp snooping limit rate <pps>` |
| Interface command is rejected | Platform, image, or interface mode does not support IP Source Guard | SW1 | `show version`<br>`show interfaces <access_interface> switchport` | Use supported switch platform and Layer 2 access interface |
| Binding exists but traffic still fails | VLAN/interface mismatch between binding and actual forwarding path | SW1 | `show ip dhcp snooping binding`<br>`show interfaces <access_interface> switchport` | Correct access VLAN or recreate binding |
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_Related_Labs
- ip-source-guard-final
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_Index
IP_Source_Guard_DHCP_Snooping_Binding_Enforcement.md
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_Mental_Model
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_Configuration_Checklist
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_Static_Host_Checklist
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_Skeleton
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_With_Port_Security_Skeleton
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_Static_Binding_Skeleton
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_Verification_Commands
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_Rollback
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_Failure_Checks
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_Related_Labs
# IP_Source_Guard_DHCP_Snooping_Binding_Enforcement_Index
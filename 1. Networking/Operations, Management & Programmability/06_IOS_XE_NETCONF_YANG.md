

IOS_XE_NETCONF_YANG.md

IOS_XE_NETCONF_YANG

# IOS_XE_NETCONF_YANG_Mental_Model
| Concept | Operational Meaning |
|---|---|
| NETCONF | Model-driven management protocol used to retrieve, edit, lock, commit, and validate YANG-modeled device data |
| YANG | The schema. It defines the data tree that NETCONF reads or changes |
| NETCONF is not CLI scraping | NETCONF exchanges structured XML RPC messages instead of screen-scraping command output |
| Transport | IOS XE NETCONF normally uses SSH on TCP/830 |
| Encoding | NETCONF uses XML |
| Capabilities | When the client connects, the device advertises what NETCONF features and YANG modules it supports |
| Datastore | Logical config/state target, commonly running configuration and operational state |
| RPC | The client sends XML requests such as `<get>`, `<get-config>`, `<edit-config>`, `<lock>`, `<unlock>`, and save/config operations |
| `get` | Retrieves configuration and operational state |
| `get-config` | Retrieves configuration from a datastore |
| `edit-config` | Changes configuration through a YANG-modeled XML payload |
| Transaction model | NETCONF is built for structured, transactional configuration workflows |
| AAA dependency | IOS XE NETCONF uses default AAA methods. For local lab auth, use default local login and exec authorization |
| `netconf-yang` | IOS XE CLI command that enables the NETCONF/YANG management process |
| Port 830 test | A raw SSH subsystem test to TCP/830 should return a NETCONF `<hello>` and capabilities dump after authentication |
| VTY distinction | `line vty` controls normal remote CLI. NETCONF over TCP/830 is a NETCONF SSH subsystem, not an interactive CLI session |
| Security restriction | NETCONF can be restricted by source with `netconf-yang ssh ipv4 access-list name <ACL_NAME>` |
| Common mistake | Enabling SSH on vty lines and assuming NETCONF is enabled. SSH readiness is required, but NETCONF still needs `netconf-yang` |
| Related labs | `netconf-final`, `ios-xe-netconf-restconf-final` |
# IOS_XE_NETCONF_YANG_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the IOS XE device has a reachable management IP | IOS XE | `show ip interface brief` | Management interface or SVI has the expected IP and is up/up |
| 2 | Confirm return routing to the NETCONF client | IOS XE | `show ip route <CLIENT_IP>` | Route exists back toward the automation client |
| 3 | Confirm client-to-device reachability | NETCONF client | `ping <DEVICE_IP>` | Client can reach the IOS XE management IP |
| 4 | Enter global configuration mode | IOS XE | `configure terminal` | Device enters global config mode |
| 5 | Configure a stable hostname for SSH identity | IOS XE | `hostname <DEVICE_NAME>` | Prompt changes to the configured hostname |
| 6 | Configure a domain name for RSA key generation | IOS XE | `ip domain-name <DOMAIN_NAME>` | Domain name exists for SSH key identity |
| 7 | Create a local privilege user with strong hashing if supported | IOS XE | `username <USER> privilege 15 algorithm-type scrypt secret <SECRET>` | Local privilege 15 user exists with type 9 secret if supported |
| 8 | Use fallback local user syntax if scrypt is unsupported | IOS XE | `username <USER> privilege 15 secret <SECRET>` | Local privilege 15 user exists with a hashed secret |
| 9 | Enable AAA if the lab uses AAA method lists for NETCONF | IOS XE | `aaa new-model` | AAA method-list configuration becomes available |
| 10 | Set default login authentication to local for lab NETCONF access | IOS XE | `aaa authentication login default local` | NETCONF SSH authentication can use the local username database |
| 11 | Set default exec authorization to local for lab NETCONF access | IOS XE | `aaa authorization exec default local` | NETCONF access has default exec authorization |
| 12 | Generate RSA keys for SSH if missing | IOS XE | `crypto key generate rsa modulus 2048` | RSA key pair exists for SSH-based services |
| 13 | Force SSH version 2 | IOS XE | `ip ssh version 2` | SSHv2 is used |
| 14 | Enable the NETCONF/YANG service | IOS XE | `netconf-yang` | IOS XE starts NETCONF/YANG management support |
| 15 | Optional: create a standard ACL for NETCONF client sources | IOS XE | `ip access-list standard <NETCONF_ACL_NAME>` | Device enters standard ACL config mode |
| 16 | Optional: permit only the trusted NETCONF client or subnet | IOS XE | `permit <CLIENT_OR_SUBNET> <WILDCARD>` | Trusted NETCONF source is permitted |
| 17 | Optional: deny all other NETCONF sources | IOS XE | `deny any` | All other NETCONF sources are blocked |
| 18 | Optional: apply the ACL to NETCONF over SSH | IOS XE | `netconf-yang ssh ipv4 access-list name <NETCONF_ACL_NAME>` | NETCONF SSH access is restricted to permitted sources |
| 19 | Exit configuration mode | IOS XE | `end` | Device returns to privileged EXEC mode |
| 20 | Save the NETCONF baseline | IOS XE | `write memory` | Startup-config keeps NETCONF and AAA baseline |
| 21 | Verify YANG management process state | IOS XE | `show platform software yang-management process` | YANG management processes are present and running |
| 22 | Verify NETCONF SSH algorithm support | IOS XE | `show netconf-yang ssh server` | NETCONF SSH server algorithms and ciphers are displayed |
| 23 | Test raw NETCONF SSH subsystem access | NETCONF client | `ssh -p 830 -s <USER>@<DEVICE_IP> netconf` | Device authenticates and returns NETCONF `<hello>` capabilities |
| 24 | Confirm the capabilities output includes supported models | NETCONF client | Review `<capability>` lines | Client can see supported NETCONF features and YANG modules |
| 25 | Test a basic NETCONF read operation | NETCONF client | Send `<get-config>` RPC against `<running/>` | Device returns XML configuration data |
| 26 | Compare modeled data to CLI truth | IOS XE | `show running-config` or `show <FEATURE>` | CLI output matches the modeled data being retrieved |
# IOS_XE_NETCONF_YANG_Skeleton
! =========================
! IOS XE NETCONF/YANG server baseline
! =========================
configure terminal
!
hostname <DEVICE_NAME>
ip domain-name <DOMAIN_NAME>
!
username <USER> privilege 15 algorithm-type scrypt secret <SECRET>
! If unsupported:
! username <USER> privilege 15 secret <SECRET>
!
aaa new-model
aaa authentication login default local
aaa authorization exec default local
!
crypto key generate rsa modulus 2048
ip ssh version 2
!
netconf-yang
!
end
write memory
! =========================
! Optional NETCONF source restriction
! =========================
configure terminal
!
ip access-list standard <NETCONF_ACL_NAME>
 permit <CLIENT_OR_SUBNET> <WILDCARD>
 deny any
 exit
!
netconf-yang ssh ipv4 access-list name <NETCONF_ACL_NAME>
!
end
write memory
! =========================
! IOS XE verification
! =========================
show platform software yang-management process
show netconf-yang ssh server
show running-config | include netconf-yang|aaa authentication login default|aaa authorization exec default
show access-lists <NETCONF_ACL_NAME>
! =========================
! Client raw NETCONF subsystem test
! Expected result: XML <hello> capabilities from the device
! =========================
ssh -p 830 -s <USER>@<DEVICE_IP> netconf
! =========================
! Basic get-config RPC after the NETCONF subsystem opens
! Paste after the XML hello exchange
! =========================
<rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <get-config>
    <source>
      <running/>
    </source>
  </get-config>
</rpc>
]]>]]>
! =========================
! Cisco save-config RPC pattern
! =========================
<rpc message-id="102" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <cisco-ia:save-config xmlns:cisco-ia="http://cisco.com/yang/cisco-ia"/>
</rpc>
]]>]]>
# IOS_XE_NETCONF_YANG_Verification_Commands
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Verify management IP state | IOS XE | `show ip interface brief` | Management IP is correct and interface is up/up |
| 2 | Verify route back to NETCONF client | IOS XE | `show ip route <CLIENT_IP>` | Device has a return path |
| 3 | Verify local user exists | IOS XE | `show running-config | include ^username` | Expected local NETCONF user exists |
| 4 | Verify AAA default login method | IOS XE | `show running-config | include ^aaa authentication login default` | Default login method points to local for lab access |
| 5 | Verify AAA default exec authorization | IOS XE | `show running-config | include ^aaa authorization exec default` | Default exec authorization points to local for lab access |
| 6 | Verify SSH prerequisites | IOS XE | `show running-config | include ^hostname|^ip domain-name|^ip ssh version` | Hostname, domain name, and SSHv2 are configured |
| 7 | Verify RSA key exists | IOS XE | `show crypto key mypubkey rsa` | RSA public key is displayed |
| 8 | Verify NETCONF is configured | IOS XE | `show running-config | include ^netconf-yang` | `netconf-yang` is present |
| 9 | Verify YANG management process state | IOS XE | `show platform software yang-management process` | YANG management processes are running |
| 10 | Verify NETCONF SSH server algorithms | IOS XE | `show netconf-yang ssh server` | NETCONF SSH server algorithm table is displayed |
| 11 | Verify NETCONF source ACL if configured | IOS XE | `show running-config | include netconf-yang ssh ipv4 access-list` | NETCONF ACL is attached |
| 12 | Verify NETCONF ACL hit counters if configured | IOS XE | `show access-lists <NETCONF_ACL_NAME>` | Permitted client entries show hits after testing |
| 13 | Verify TCP/830 reachability from client | NETCONF client | `ssh -p 830 -s <USER>@<DEVICE_IP> netconf` | Client authenticates and receives NETCONF `<hello>` capabilities |
| 14 | Verify the device advertises capabilities | NETCONF client | Review `<capability>` output | Capabilities include NETCONF base capability and supported YANG modules |
| 15 | Verify read access to running config | NETCONF client | Send `<get-config>` RPC against `<running/>` | XML config data is returned |
| 16 | Verify modeled data against CLI | IOS XE | `show running-config` | CLI and returned XML represent the same configuration |
| 17 | Verify active users if SSH login behavior is unclear | IOS XE | `show users` | Interactive CLI sessions are visible, but NETCONF should be tested on TCP/830 |
| 18 | Verify standard SSH separately if needed | IOS XE | `show ip ssh` | SSHv2 is enabled and available |
# IOS_XE_NETCONF_YANG_Rollback
! =========================
! Remove NETCONF source restriction only
! =========================
configure terminal
!
no netconf-yang ssh ipv4 access-list name <NETCONF_ACL_NAME>
no ip access-list standard <NETCONF_ACL_NAME>
!
end
write memory
! =========================
! Disable NETCONF/YANG service
! Use console or known alternate access before doing this
! =========================
configure terminal
!
no netconf-yang
!
end
write memory
! =========================
! Lab-only AAA rollback
! Do not remove AAA defaults if other services depend on them
! =========================
configure terminal
!
no aaa authorization exec default local
no aaa authentication login default local
!
end
write memory
! =========================
! Full lab teardown only if this lab created the SSH and local-auth baseline
! =========================
configure terminal
!
no username <USER>
no ip ssh version 2
no ip domain-name <DOMAIN_NAME>
crypto key zeroize rsa
!
end
write memory
# IOS_XE_NETCONF_YANG_Failure_Checks
| Symptom | Device | Command | Likely Cause | Corrective Action |
|---|---|---|---|---|
| Client cannot reach device | NETCONF client | `ping <DEVICE_IP>` | IP path, interface, SVI, VLAN, route, or gateway problem | Fix management reachability first |
| SSH to TCP/830 times out | IOS XE | `show running-config | include ^netconf-yang` | NETCONF/YANG service is not enabled or TCP/830 is filtered | Configure `netconf-yang` and permit TCP/830 |
| SSH to TCP/830 is refused | IOS XE | `show platform software yang-management process` | NETCONF process is not running or failed to initialize | Re-enable `netconf-yang` or restart during lab maintenance |
| Authentication fails on TCP/830 | IOS XE | `show running-config | include ^username|^aaa authentication login default` | Missing local user or default AAA login method is wrong | Create local user and set `aaa authentication login default local` |
| Authorization fails after login | IOS XE | `show running-config | include ^aaa authorization exec default` | Default exec authorization missing or points to unavailable AAA | Configure `aaa authorization exec default local` for lab access |
| Client connects but no capabilities appear | NETCONF client | `ssh -p 830 -s <USER>@<DEVICE_IP> netconf` | Wrong SSH subsystem syntax or client did not request NETCONF subsystem | Use `ssh -p 830 -s <USER>@<DEVICE_IP> netconf` |
| Standard SSH works but NETCONF fails | IOS XE | `show running-config | include ^netconf-yang` | SSH is enabled, but NETCONF/YANG service is not enabled | Configure `netconf-yang` |
| NETCONF works from one client but not another | IOS XE | `show access-lists <NETCONF_ACL_NAME>` | NETCONF source ACL blocks the second client | Correct `netconf-yang ssh ipv4 access-list name <ACL_NAME>` or ACL entries |
| NETCONF ACL causes negotiation reset | NETCONF client | Client error output | Source is blocked by NETCONF service-level restriction | Permit the client source or remove the NETCONF ACL |
| RPC returns unknown element | NETCONF client | RPC reply error | Wrong namespace, wrong module, or unsupported YANG path | Recheck module name, namespace, revision, and path |
| RPC returns operation not supported | NETCONF client | RPC reply error | Device capability set does not support requested operation | Check capabilities and use an advertised operation |
| `edit-config` fails on selected leaf | NETCONF client | RPC reply error | Target leaf is operational state or `config false` | Use a configurable path only |
| Returned XML does not match intended feature | IOS XE | `show running-config | section <FEATURE>` | Wrong model family or wrong subtree selected | Validate Cisco native, IETF, or OpenConfig model choice |
| YANG process appears stuck or unhealthy | IOS XE | `show platform software yang-management process` | NETCONF/YANG management process issue | Disable and re-enable `netconf-yang` during lab maintenance |
| SSH cipher or algorithm negotiation fails | IOS XE | `show netconf-yang ssh server` | Client and server do not share acceptable SSH algorithms | Use a compatible client or adjust SSH hardening policy |
| User expects vty `transport input ssh` to enable NETCONF | IOS XE | `show running-config | section line vty` | Confusion between interactive SSH and NETCONF SSH subsystem | Keep SSH baseline, but enable NETCONF with `netconf-yang` and test TCP/830 |
| Lab tool cannot build RPCs | YANG Suite or client | Device profile and repository view | Missing YANG repository, missing module dependencies, or wrong module set | Build device profile, repository, module set, then retest NETCONF RPC |
##### Source_Basis
# IOS_XE_NETCONF_YANG_Mental_Model
# IOS_XE_NETCONF_YANG_Configuration_Checklist
# IOS_XE_NETCONF_YANG_Skeleton
# IOS_XE_NETCONF_YANG_Verification_Commands
# IOS_XE_NETCONF_YANG_Rollback
# IOS_XE_NETCONF_YANG_Failure_Checks
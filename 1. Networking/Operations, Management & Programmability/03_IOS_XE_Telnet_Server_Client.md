IOS_XE_Telnet_Server_Client.md

IOS_XE_Telnet_Server_Client

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | CiscoPress / ENCOR / ENARSI source map | Points IOS XE management access and remote management troubleshooting topics to CiscoPress ENCOR and ENARSI sources |
| `CCNP and CCIE Enterprise Core ENCOR 350-401 Official Cert.md` | Ch26, Terminal Lines and Password Protection | Supports the model that vty lines are logical remote-access lines used for Telnet and SSH |
| `CCNP and CCIE Enterprise Core ENCOR 350-401 Official Cert.md` | Ch26, Username and Password Authentication | Supports local username authentication using `username`, `secret`, privilege levels, and `login local` |
| `CCNP and CCIE Enterprise Core ENCOR 350-401 Official Cert.md` | Ch26, Controlling Access to vty Lines Using Transport Input | Supports `transport input telnet`, `transport input telnet ssh`, `transport input none`, and vty line behavior |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Ch23, Device Management Troubleshooting, Telnet | Supports Telnet failure checks for reachability, `transport input`, `login local`, missing passwords, ACL filtering, busy vty lines, `no exec`, TCP/23 blocking, and `transport output telnet` |
# IOS_XE_Telnet_Server_Client_Mental_Model
| Concept | Operational Meaning |
|---|---|
| Telnet | Plaintext remote CLI access over TCP/23 |
| Telnet server | The IOS XE device accepting inbound Telnet sessions through its vty lines |
| Telnet client | The host or IOS XE device initiating `telnet <ip-address>` toward the server |
| VTY dependency | Telnet does not bind to a physical interface; it lands on logical `line vty` sessions after IP reachability works |
| Authentication dependency | Telnet needs the vty line to require credentials with `login`, `login local`, or AAA |
| Local-auth lab model | For CML labs, use `username <USER> privilege 15 secret <SECRET>` plus `login local` |
| Transport dependency | Inbound Telnet requires `transport input telnet` or `transport input telnet ssh` under the vty lines |
| TCP/23 dependency | ACLs, CoPP, firewalls, or path filters blocking TCP/23 will break Telnet while ping may still work |
| VTY exhaustion | If every Telnet-allowed vty line is already in use, new Telnet attempts can fail |
| Outbound Telnet | If an IOS XE device is used as the Telnet client and outbound Telnet is blocked on the active line, configure `transport output telnet` |
| Security posture | Telnet is legacy and lab-only. Use SSH for normal device administration |
| Related labs | `telnet-final`, `telnet-server-client-final` |
# IOS_XE_Telnet_Server_Client_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the Telnet server has a reachable management IP | Telnet server | `show ip interface brief` | Target interface or SVI has the expected IP and is up/up |
| 2 | Confirm the Telnet server has a return route to the client | Telnet server | `show ip route <CLIENT_IP>` | Route exists back toward the Telnet client |
| 3 | Confirm the Telnet client can reach the server IP | Telnet client | `ping <SERVER_IP>` | Ping succeeds before Telnet is tested |
| 4 | Enter global configuration mode on the Telnet server | Telnet server | `configure terminal` | Device enters global config mode |
| 5 | Create a local privilege 15 user using type 9 hashing if supported | Telnet server | `username <USER> privilege 15 algorithm-type scrypt secret <SECRET>` | Local admin user exists with a strong hashed secret |
| 6 | Use fallback local user syntax if scrypt is unsupported | Telnet server | `username <USER> privilege 15 secret <SECRET>` | Local admin user exists with a hashed secret |
| 7 | Enter the vty line range | Telnet server | `line vty 0 4` | Device enters vty line config mode |
| 8 | Require local username authentication on vty lines | Telnet server | `login local` | Telnet sessions prompt for username and password |
| 9 | Permit inbound Telnet only for the Telnet lab | Telnet server | `transport input telnet` | VTY lines accept inbound Telnet |
| 10 | Permit both Telnet and SSH only if the lab compares both protocols | Telnet server | `transport input telnet ssh` | VTY lines accept inbound Telnet and SSH |
| 11 | Configure a sane idle timeout | Telnet server | `exec-timeout 10 0` | Idle Telnet sessions disconnect after 10 minutes |
| 12 | Ensure EXEC is enabled on the vty line | Telnet server | `exec` | Remote Telnet sessions can start an EXEC shell |
| 13 | Optional: create a standard ACL for allowed Telnet management source | Telnet server | `access-list <ACL_NUM> permit <CLIENT_IP>` | ACL permits the Telnet client |
| 14 | Optional: explicitly deny and log all other Telnet management sources | Telnet server | `access-list <ACL_NUM> deny any log` | Denied source attempts can increment/log |
| 15 | Optional: apply source restriction to vty lines | Telnet server | `access-class <ACL_NUM> in` | Only permitted source IPs can access the vty lines |
| 16 | Exit configuration mode | Telnet server | `end` | Device returns to privileged EXEC mode |
| 17 | Save the Telnet server configuration | Telnet server | `write memory` | Telnet server config is saved |
| 18 | If the IOS XE client blocks outbound Telnet, enter client config mode | Telnet client | `configure terminal` | Client enters global config mode |
| 19 | Permit outbound Telnet from the console line if initiating from console | Telnet client | `line con 0` then `transport output telnet` | Console session can initiate outbound Telnet |
| 20 | Permit outbound Telnet from vty lines if initiating from a remote session | Telnet client | `line vty 0 4` then `transport output telnet` | VTY session can initiate outbound Telnet |
| 21 | Save the Telnet client configuration if outbound transport was changed | Telnet client | `end` then `write memory` | Client-side transport setting is saved |
| 22 | Initiate Telnet from the client | Telnet client | `telnet <SERVER_IP>` | Session opens and prompts for username and password |
| 23 | Authenticate with the local user | Telnet client | `<USER>` and `<SECRET>` | User lands on the Telnet server CLI |
| 24 | Confirm privilege level after login | Telnet server session | `show privilege` | Privilege level is 15 if the username was configured with privilege 15 |
# IOS_XE_Telnet_Server_Client_Skeleton
! =========================
! Telnet server
! =========================
configure terminal
!
username <USER> privilege 15 algorithm-type scrypt secret <SECRET>
! If unsupported:
! username <USER> privilege 15 secret <SECRET>
!
line vty 0 4
 login local
 transport input telnet
 exec-timeout 10 0
 exec
!
end
write memory
!
! =========================
! Optional vty source restriction
! =========================
configure terminal
!
access-list <ACL_NUM> permit <CLIENT_IP>
access-list <ACL_NUM> deny any log
!
line vty 0 4
 access-class <ACL_NUM> in
!
end
write memory
!
! =========================
! Telnet client test
! =========================
ping <SERVER_IP>
telnet <SERVER_IP>
!
! Login with:
! Username: <USER>
! Password: <SECRET>
# IOS_XE_Telnet_Server_Client_Verification_Commands
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Verify server management IP state | Telnet server | `show ip interface brief` | Server IP is correct and interface is up/up |
| 2 | Verify server has route back to client | Telnet server | `show ip route <CLIENT_IP>` | Route exists toward the client |
| 3 | Verify client can reach server | Telnet client | `ping <SERVER_IP>` | Ping succeeds |
| 4 | Verify vty config block | Telnet server | `show running-config | section line vty` | VTY lines show `login local` and `transport input telnet` or `transport input telnet ssh` |
| 5 | Verify allowed vty input protocols | Telnet server | `show line vty 0 | include Allowed` | Allowed input transports include `telnet` |
| 6 | Verify local username exists | Telnet server | `show running-config | include ^username` | Expected local user is present |
| 7 | Verify source ACL if used | Telnet server | `show access-lists <ACL_NUM>` | ACL permits the client and shows expected hit counters |
| 8 | Verify vty ACL attachment if used | Telnet server | `show running-config | section line vty` | VTY block includes `access-class <ACL_NUM> in` |
| 9 | Open Telnet session | Telnet client | `telnet <SERVER_IP>` | Session opens and prompts for username/password |
| 10 | Verify active Telnet user | Telnet server | `show users` | Remote Telnet session appears on a vty line |
| 11 | Verify line usage | Telnet server | `show line` | Active vty line is marked in use |
| 12 | Verify user privilege after login | Telnet session | `show privilege` | User has expected privilege level |
| 13 | Verify syslog visibility in Telnet session if needed | Telnet session | `terminal monitor` | Syslog messages display in the remote Telnet session |
| 14 | Verify outbound Telnet permission from IOS XE client if blocked | Telnet client | `show line con 0 | include Allowed` or `show line vty 0 | include Allowed` | Allowed output transports include `telnet` |
# IOS_XE_Telnet_Server_Client_Rollback
! =========================
! Remove Telnet server access
! =========================
configure terminal
!
line vty 0 4
 no access-class <ACL_NUM> in
 transport input none
 no exec-timeout
 exit
!
no access-list <ACL_NUM>
no username <USER>
!
end
write memory
!
! =========================
! Safer rollback to SSH-only baseline
! Use this instead of transport input none when SSH is already configured
! =========================
configure terminal
!
line vty 0 4
 login local
 transport input ssh
 exec-timeout 10 0
 exit
!
end
write memory
!
! =========================
! Remove outbound Telnet permission from client if configured
! =========================
configure terminal
!
line con 0
 no transport output telnet
 exit
!
line vty 0 4
 no transport output telnet
 exit
!
end
write memory
# IOS_XE_Telnet_Server_Client_Failure_Checks
| Symptom | Device | Command | Likely Cause | Corrective Action |
|---|---|---|---|---|
| Client cannot ping Telnet server | Telnet client | `ping <SERVER_IP>` | IP reachability, interface, VLAN, route, or gateway problem | Fix basic IP path before troubleshooting Telnet |
| Server has no return path to client | Telnet server | `show ip route <CLIENT_IP>` | Missing route or wrong default route | Add or correct route toward the client |
| Telnet connection is refused | Telnet server | `show line vty 0 | include Allowed` | VTY lines do not allow Telnet | Configure `transport input telnet` or `transport input telnet ssh` |
| Telnet opens then immediately closes | Telnet server | `show running-config | section line vty` | `no exec` is configured under vty | Configure `exec` under the vty lines |
| Telnet prompts for password only | Telnet server | `show running-config | section line vty` | VTY is using `login` with a line password model | Use `login local` for username/password authentication |
| Telnet says password required but none set | Telnet server | `show running-config | section line vty` | `login` is configured but no line password exists, or all vty lines are busy | Configure `login local` with a local user, or check `show users` for line exhaustion |
| Username/password fails | Telnet server | `show running-config | include ^username` | Missing user, wrong secret, or wrong privilege/user syntax | Recreate the local user with the expected secret |
| Login succeeds but user lands at `>` | Telnet server | `show running-config | include ^username` | Username lacks privilege 15 | Configure `username <USER> privilege 15 secret <SECRET>` |
| Ping works but Telnet fails | Path device or Telnet server | `show access-lists` | ACL, vty `access-class`, firewall, or CoPP blocks TCP/23 | Permit TCP/23 from the lab client to the server |
| Telnet source restriction blocks valid client | Telnet server | `show access-lists <ACL_NUM>` | ACL source address or wildcard is wrong | Correct the ACL and verify hit counters |
| New Telnet sessions fail while existing sessions work | Telnet server | `show users` | All Telnet-allowed vty lines are busy | Clear stale sessions with `clear line <LINE_NUMBER>` or free existing sessions |
| Telnet fails only from IOS XE client | Telnet client | `show line con 0 | include Allowed` or `show line vty 0 | include Allowed` | Outbound Telnet not permitted from the active terminal line | Configure `transport output telnet` under the active line |
| Telnet session does not show syslog messages | Telnet session | `terminal monitor` | Remote sessions do not display syslog by default | Run `terminal monitor` in the Telnet session |
| Commands are hard to read during logging bursts | Telnet server | `show running-config | section line` | `logging synchronous` missing on active line | Add `logging synchronous` under console or vty as needed |
| Telnet works but this is not a lab | Telnet server | `show running-config | section line vty` | Plaintext management is enabled in production-style config | Replace Telnet with SSH and set `transport input ssh` |
##### Source_Basis
# IOS_XE_Telnet_Server_Client_Mental_Model
# IOS_XE_Telnet_Server_Client_Configuration_Checklist
# IOS_XE_Telnet_Server_Client_Skeleton
# IOS_XE_Telnet_Server_Client_Verification_Commands
# IOS_XE_Telnet_Server_Client_Rollback
# IOS_XE_Telnet_Server_Client_Failure_Checks
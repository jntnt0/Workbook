
IOS_XE_SSH_Server_Client.md

IOS_XE_SSH_Server_Client

# Source_Basis
| Source | Relevant Section | What It Supports |
|---|---|---|
| `_KB_INDEX.md` | CiscoPress / ENCOR / ENARSI source map | Points IOS XE management access and remote management troubleshooting topics to CiscoPress ENCOR and ENARSI sources |
| `CCNP and CCIE Enterprise Core ENCOR 350-401 Official Cert.md` | Ch26, Enabling SSH vty Access | Supports SSH prerequisites: hostname, domain name, RSA key generation, local username, vty login, and SSHv2 |
| `CCNP and CCIE Enterprise Core ENCOR 350-401 Official Cert.md` | Ch26, Terminal Lines and Password Protection | Supports vty line behavior and the difference between console, aux, and vty access |
| `CCNP and CCIE Enterprise Core ENCOR 350-401 Official Cert.md` | Ch26, Username and Password Authentication | Supports local username authentication with `username`, `secret`, privilege levels, and `login local` |
| `CCNP and CCIE Enterprise Core ENCOR 350-401 Official Cert.md` | Ch26, Controlling Access to vty Lines Using Transport Input | Supports `transport input ssh` and vty transport restriction |
| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Ch23, SSH troubleshooting | Supports `show ip ssh`, `show ssh`, SSH version checks, key-size checks, TCP/22 filtering, `login local`, and outbound `transport output ssh` |
# IOS_XE_SSH_Server_Client_Mental_Model
| Concept | Operational Meaning |
|---|---|
| SSH | Encrypted remote CLI access over TCP/22 |
| SSH server | The IOS XE device accepting inbound SSH sessions through its vty lines |
| SSH client | The host or IOS XE device initiating an SSH session toward the server |
| VTY dependency | SSH does not attach to a physical interface. It terminates on logical `line vty` sessions after IP reachability works |
| Local-auth dependency | SSH requires username-based authentication. `login local` points the vty lines at the local username database |
| Identity dependency | SSH key generation depends on the device hostname and domain name |
| Crypto key dependency | SSH needs local RSA keys before the SSH server can operate |
| SSHv2 | Normal secure target. SSHv1 should not be used |
| Transport dependency | Inbound SSH requires `transport input ssh` or `transport input telnet ssh` under the vty lines |
| TCP/22 dependency | ACLs, CoPP, firewalls, or path filters blocking TCP/22 will break SSH even if ping works |
| Outbound SSH | If an IOS XE device is used as the SSH client, the active line may need `transport output ssh` |
| Security posture | SSH is the normal management protocol. Telnet is legacy and lab-only |
| Related labs | `ssh-server-client-final` |
# IOS_XE_SSH_Server_Client_Configuration_Checklist
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Confirm the SSH server has a reachable management IP | SSH server | `show ip interface brief` | Target interface or SVI has the expected IP and is up/up |
| 2 | Confirm the SSH server has a route back to the client | SSH server | `show ip route <CLIENT_IP>` | Route exists back toward the SSH client |
| 3 | Confirm the SSH client can reach the server IP | SSH client | `ping <SERVER_IP>` | Ping succeeds before SSH is tested |
| 4 | Enter global configuration mode on the SSH server | SSH server | `configure terminal` | Device enters global config mode |
| 5 | Configure a non-default hostname | SSH server | `hostname <DEVICE_NAME>` | Prompt changes to the configured hostname |
| 6 | Configure a domain name for RSA key identity | SSH server | `ip domain-name <DOMAIN_NAME>` | Device has the domain value needed for key generation |
| 7 | Create a local privilege 15 user using strong hashing if supported | SSH server | `username <USER> privilege 15 algorithm-type scrypt secret <SECRET>` | Local admin user exists with type 9 secret if supported |
| 8 | Use fallback local user syntax if scrypt is unsupported | SSH server | `username <USER> privilege 15 secret <SECRET>` | Local admin user exists with a hashed secret |
| 9 | Generate RSA keys with a modern modulus | SSH server | `crypto key generate rsa modulus 2048` | RSA key pair is generated and SSH can be enabled |
| 10 | Force SSH version 2 | SSH server | `ip ssh version 2` | Device accepts SSHv2 sessions only |
| 11 | Optional: set SSH authentication retry limit | SSH server | `ip ssh authentication-retries 2` | Failed login attempts are limited |
| 12 | Optional: set SSH timeout | SSH server | `ip ssh time-out 60` | SSH authentication must complete within 60 seconds |
| 13 | Enter vty line configuration | SSH server | `line vty 0 4` | Device enters vty line config mode |
| 14 | Require local username authentication on vty lines | SSH server | `login local` | SSH sessions use the local username database |
| 15 | Permit inbound SSH only | SSH server | `transport input ssh` | VTY lines accept SSH and reject Telnet |
| 16 | Configure vty idle timeout | SSH server | `exec-timeout 10 0` | Idle SSH sessions disconnect after 10 minutes |
| 17 | Ensure EXEC is enabled on the vty line | SSH server | `exec` | Remote SSH sessions can start an EXEC shell |
| 18 | Optional: create a standard ACL for allowed SSH source | SSH server | `access-list <ACL_NUM> permit <CLIENT_IP>` | ACL permits the SSH client |
| 19 | Optional: explicitly deny and log other SSH sources | SSH server | `access-list <ACL_NUM> deny any log` | Denied source attempts can increment/log |
| 20 | Optional: apply source restriction to vty lines | SSH server | `access-class <ACL_NUM> in` | Only permitted source IPs can access the vty lines |
| 21 | Exit configuration mode | SSH server | `end` | Device returns to privileged EXEC mode |
| 22 | Save the SSH server configuration | SSH server | `write memory` | SSH server config is saved |
| 23 | If an IOS XE client blocks outbound SSH, enter client config mode | SSH client | `configure terminal` | Client enters global config mode |
| 24 | Permit outbound SSH from the console line if initiating from console | SSH client | `line con 0` then `transport output ssh` | Console session can initiate outbound SSH |
| 25 | Permit outbound SSH from vty lines if initiating from a remote session | SSH client | `line vty 0 4` then `transport output ssh` | VTY session can initiate outbound SSH |
| 26 | Save the SSH client configuration if outbound transport was changed | SSH client | `end` then `write memory` | Client-side transport setting is saved |
| 27 | Initiate SSH from an IOS XE client | SSH client | `ssh -l <USER> <SERVER_IP>` | SSH session opens and prompts for password |
| 28 | Initiate SSH from a Linux/macOS client | SSH client | `ssh <USER>@<SERVER_IP>` | SSH session opens and prompts for password |
| 29 | Confirm privilege level after login | SSH server session | `show privilege` | Privilege level is 15 if the username was configured with privilege 15 |
# IOS_XE_SSH_Server_Client_Skeleton
! =========================
! SSH server
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
crypto key generate rsa modulus 2048
!
ip ssh version 2
ip ssh authentication-retries 2
ip ssh time-out 60
!
line vty 0 4
 login local
 transport input ssh
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
! SSH client test from IOS XE
! =========================
ping <SERVER_IP>
ssh -l <USER> <SERVER_IP>
!
! =========================
! SSH client test from Linux/macOS
! =========================
ssh <USER>@<SERVER_IP>
# IOS_XE_SSH_Server_Client_Verification_Commands
| Step | Task | Device | Command | Expected Result |
|---:|---|---|---|---|
| 1 | Verify server management IP state | SSH server | `show ip interface brief` | Server IP is correct and interface is up/up |
| 2 | Verify server has route back to client | SSH server | `show ip route <CLIENT_IP>` | Route exists toward the client |
| 3 | Verify client can reach server | SSH client | `ping <SERVER_IP>` | Ping succeeds |
| 4 | Verify hostname and domain name | SSH server | `show running-config | include ^hostname|^ip domain-name` | Hostname and domain name are configured |
| 5 | Verify local username exists | SSH server | `show running-config | include ^username` | Expected local user is present |
| 6 | Verify RSA key exists | SSH server | `show crypto key mypubkey rsa` | RSA public key information is displayed |
| 7 | Verify SSH server status and version | SSH server | `show ip ssh` | SSH is enabled and version 2 is active |
| 8 | Verify vty config block | SSH server | `show running-config | section line vty` | VTY lines show `login local` and `transport input ssh` |
| 9 | Verify allowed vty input protocols | SSH server | `show line vty 0 | include Allowed` | Allowed input transports include `ssh` |
| 10 | Verify source ACL if used | SSH server | `show access-lists <ACL_NUM>` | ACL permits the client and shows expected hit counters |
| 11 | Verify vty ACL attachment if used | SSH server | `show running-config | section line vty` | VTY block includes `access-class <ACL_NUM> in` |
| 12 | Open SSH session from IOS XE client | SSH client | `ssh -l <USER> <SERVER_IP>` | Session opens and authenticates |
| 13 | Open SSH session from Linux/macOS client | SSH client | `ssh <USER>@<SERVER_IP>` | Session opens and authenticates |
| 14 | Verify active SSH sessions | SSH server | `show ssh` | Active SSHv2 sessions are listed |
| 15 | Verify active remote users | SSH server | `show users` | SSH session appears on a vty line |
| 16 | Verify line usage | SSH server | `show line` | Active vty line is marked in use |
| 17 | Verify user privilege after login | SSH server session | `show privilege` | User has expected privilege level |
| 18 | Verify syslog visibility in SSH session if needed | SSH server session | `terminal monitor` | Syslog messages display in the remote SSH session |
# IOS_XE_SSH_Server_Client_Rollback
! =========================
! Close inbound SSH access without deleting keys or users
! Safe when console access is available
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
!
end
write memory
!
! =========================
! Remove SSH-specific hardening but keep basic management user
! =========================
configure terminal
!
no ip ssh authentication-retries
no ip ssh time-out
no ip ssh version
!
end
write memory
!
! =========================
! Full SSH teardown
! Only do this from console or known alternate management access
! =========================
configure terminal
!
line vty 0 4
 transport input none
 no login local
 exit
!
no username <USER>
no ip domain-name <DOMAIN_NAME>
crypto key zeroize rsa
!
end
write memory
!
! =========================
! Remove outbound SSH permission from IOS XE client if configured
! =========================
configure terminal
!
line con 0
 no transport output ssh
 exit
!
line vty 0 4
 no transport output ssh
 exit
!
end
write memory
# IOS_XE_SSH_Server_Client_Failure_Checks
| Symptom | Device | Command | Likely Cause | Corrective Action |
|---|---|---|---|---|
| Client cannot ping SSH server | SSH client | `ping <SERVER_IP>` | IP reachability, interface, VLAN, route, or gateway problem | Fix basic IP path before troubleshooting SSH |
| Server has no return path to client | SSH server | `show ip route <CLIENT_IP>` | Missing route or wrong default route | Add or correct the route toward the client |
| SSH is not enabled | SSH server | `show ip ssh` | Missing hostname, domain name, RSA keys, or crypto support | Configure hostname, domain name, RSA keys, and SSHv2 |
| RSA key generation fails | SSH server | `show running-config | include ^hostname|^ip domain-name` | Hostname or domain name is missing | Configure both values, then regenerate keys |
| SSHv2 fails with weak key | SSH server | `show ip ssh` | RSA key size is too small for SSHv2 or modern IOS XE crypto policy | Regenerate RSA keys with `crypto key generate rsa modulus 2048` |
| SSH connection is refused | SSH server | `show line vty 0 | include Allowed` | VTY lines do not allow SSH | Configure `transport input ssh` |
| SSH prompts fail after password entry | SSH server | `show running-config | section line vty` | VTY line uses `login` instead of username-based authentication | Configure `login local` |
| SSH asks for username but authentication fails | SSH server | `show running-config | include ^username` | Missing user, wrong secret, or wrong username | Recreate or correct the local user |
| Login succeeds but user lands at `>` | SSH server | `show running-config | include ^username` | Username lacks privilege 15 | Configure `username <USER> privilege 15 secret <SECRET>` |
| Ping works but SSH fails | Path device or SSH server | `show access-lists` | ACL, vty `access-class`, firewall, or CoPP blocks TCP/22 | Permit TCP/22 from the management client to the SSH server |
| SSH source restriction blocks valid client | SSH server | `show access-lists <ACL_NUM>` | ACL source address or wildcard is wrong | Correct the ACL and verify hit counters |
| New SSH sessions fail while existing sessions work | SSH server | `show users` | All SSH-allowed vty lines are busy | Clear stale sessions with `clear line <LINE_NUMBER>` or free existing sessions |
| SSH client command fails with no user specified | SSH client | `ssh <SERVER_IP>` | IOS XE SSH client requires a username when none is available | Use `ssh -l <USER> <SERVER_IP>` |
| SSH fails only from IOS XE client | SSH client | `show line con 0 | include Allowed` or `show line vty 0 | include Allowed` | Outbound SSH not permitted from the active terminal line | Configure `transport output ssh` under the active line |
| SSH version mismatch | SSH server | `show ip ssh` | Server and client are trying incompatible SSH versions | Use `ip ssh version 2` and connect with an SSHv2-capable client |
| SSH session does not show syslog messages | SSH server session | `terminal monitor` | Remote sessions do not display syslog by default | Run `terminal monitor` in the SSH session |
| Commands are hard to read during logging bursts | SSH server | `show running-config | section line` | `logging synchronous` missing on active line | Add `logging synchronous` under console or vty as needed |
| Telnet still works after SSH configuration | SSH server | `show running-config | section line vty` | VTY line permits Telnet with `transport input telnet ssh` or `all` | Change vty transport to `transport input ssh` |
##### Source_Basis
# IOS_XE_SSH_Server_Client_Mental_Model
# IOS_XE_SSH_Server_Client_Configuration_Checklist
# IOS_XE_SSH_Server_Client_Skeleton
# IOS_XE_SSH_Server_Client_Verification_Commands
# IOS_XE_SSH_Server_Client_Rollback
# IOS_XE_SSH_Server_Client_Failure_Checks
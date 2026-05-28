
IOS_XE_VTY_Line_Transport_Input.md

IOS_XE_VTY_Line_Transport_Input

# Source_Basis

| Source | Relevant Section | What It Supports |

|---|---|---|

| `_KB_INDEX.md` | CiscoPress and ENCOR/ENARSI source map | Points IOS XE management access, SSH, Telnet, and troubleshooting topics to the CiscoPress ENCOR and ENARSI sources |

| `CCNP and CCIE Enterprise Core ENCOR 350-401 Official Cert.md` | Ch26, Terminal Lines and Password Protection | Defines console, aux, and vty lines and explains that vty lines are logical remote-access lines for Telnet and SSH |

| `CCNP and CCIE Enterprise Core ENCOR 350-401 Official Cert.md` | Ch26, Username and Password Authentication | Supports local username creation, `login local`, and privilege assignment |

| `CCNP and CCIE Enterprise Core ENCOR 350-401 Official Cert.md` | Ch26, Controlling Access to vty Lines Using Transport Input | Provides `transport input all`, `transport input none`, `transport input telnet`, `transport input ssh`, and `transport input telnet ssh` behavior |

| `CCNP and CCIE Enterprise Core ENCOR 350-401 Official Cert.md` | Ch26, Verifying Access to vty Lines Using Transport Input | Supports `show line` verification and confirms vty lines are consumed from the lowest available matching line upward |

| `CCNP Enterprise Advanced Routing ENARSI 300-410 Official.md` | Ch23, Device Management Troubleshooting | Supports troubleshooting with `show line vty <line> | include Allowed`, `show users`, TCP/22, TCP/23, busy vty lines, and missing `login local` |

# IOS_XE_VTY_Line_Transport_Input_Mental_Model

| Concept | Operational Meaning |

|---|---|

| VTY lines | Logical remote-access lines used for inbound CLI management sessions |

| VTY is not an interface | VTY lines do not carry routed traffic; they terminate management sessions after IP reachability already works |

| `transport input` | Controls which inbound protocols are allowed to connect to the vty lines |

| `login local` | Forces username and password authentication against the local username database |

| Telnet | Plaintext CLI access over TCP/23; useful for explicit labs but bad as a normal default |

| SSH | Encrypted CLI access over TCP/22; normal secure choice for device administration |

| `transport input none` | Blocks inbound remote CLI sessions on the selected vty line or line range |

| `transport input all` | Allows all supported inbound protocols; useful for demonstration but loose for real operations |

| VTY line selection | IOS XE uses the first available vty line that permits the requested protocol |

| Line exhaustion | If all matching vty lines are busy, new sessions can fail even when credentials and reachability are correct |

| Separate protocol setup | `transport input ssh` allows SSH on the vty line, but SSH still needs hostname, domain name, RSA keys, local user, and SSH version setup |

| Related labs | `telnet-final`, `telnet-server-client-final`, `ssh-server-client-final` |

# IOS_XE_VTY_Line_Transport_Input_Configuration_Checklist

| Step | Task | Device | Command | Expected Result |

|---:|---|---|---|---|

| 1 | Confirm management IP reachability exists before touching vty lines | IOS XE | `show ip interface brief` | Management-facing interface has the expected IP and is up/up |

| 2 | Confirm the device has a reachable route back to the management client | IOS XE | `show ip route <CLIENT_IP>` | Route exists toward the management client |

| 3 | Enter global configuration mode | IOS XE | `configure terminal` | Device enters global config mode |

| 4 | Create a local admin user for vty authentication | IOS XE | `username <USER> privilege 15 algorithm-type scrypt secret <SECRET>` | Local privilege 15 user exists with type 9 secret if supported |

| 5 | Use fallback local user syntax if scrypt is unsupported | IOS XE | `username <USER> privilege 15 secret <SECRET>` | Local privilege 15 user exists with hashed secret |

| 6 | Enter the standard vty line range | IOS XE | `line vty 0 4` | Device enters vty line configuration mode |

| 7 | Require local username authentication on the vty lines | IOS XE | `login local` | Telnet or SSH sessions prompt for username and password |

| 8 | Configure secure default inbound access | IOS XE | `transport input ssh` | VTY lines accept inbound SSH only |

| 9 | Configure Telnet-only access for Telnet lab validation | IOS XE | `transport input telnet` | VTY lines accept inbound Telnet only |

| 10 | Configure mixed Telnet and SSH access for comparison labs | IOS XE | `transport input telnet ssh` | VTY lines accept both inbound Telnet and SSH |

| 11 | Block inbound access on selected vty lines if testing line behavior | IOS XE | `transport input none` | Selected vty lines refuse inbound remote access |

| 12 | Avoid loose lab behavior unless specifically needed | IOS XE | `transport input all` | All supported inbound transports are allowed on the selected vty line |

| 13 | Configure idle timeout for remote sessions | IOS XE | `exec-timeout 10 0` | Idle vty sessions disconnect after 10 minutes |

| 14 | Optional: prevent DNS lookup delays from mistyped CLI commands | IOS XE | `exit` then `no ip domain-lookup` | Mistyped commands do not trigger DNS lookup delays |

| 15 | Optional: restrict vty access by source IP | IOS XE | `access-list <ACL_NUM> permit <MGMT_SOURCE> <WILDCARD>` | Source filter ACL exists |

| 16 | Optional: apply source restriction inbound to vty lines | IOS XE | `line vty 0 4` then `access-class <ACL_NUM> in` | Only permitted management sources can reach the vty lines |

| 17 | Exit line configuration mode | IOS XE | `end` | Device returns to privileged EXEC mode |

| 18 | Save the vty transport baseline | IOS XE | `write memory` | Startup-config matches the working vty baseline |

# IOS_XE_VTY_Line_Transport_Input_Skeleton

  

configure terminal

!

username <USER> privilege 15 algorithm-type scrypt secret <SECRET>

! If unsupported:

! username <USER> privilege 15 secret <SECRET>

!

line vty 0 4

 login local

 transport input ssh

 exec-timeout 10 0

!

end

write memory

# IOS_XE_VTY_Line_Transport_Input_Skeleton_Telnet_Lab_Only

  

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

!

end

write memory

# IOS_XE_VTY_Line_Transport_Input_Skeleton_Mixed_Telnet_And_SSH_Lab

  

configure terminal

!

username <USER> privilege 15 algorithm-type scrypt secret <SECRET>

! If unsupported:

! username <USER> privilege 15 secret <SECRET>

!

line vty 0 4

 login local

 transport input telnet ssh

 exec-timeout 10 0

!

end

write memory

# IOS_XE_VTY_Line_Transport_Input_Skeleton_Per_Line_Behavior_Lab

  

configure terminal

!

username <USER> privilege 15 algorithm-type scrypt secret <SECRET>

!

line vty 0

 login local

 transport input all

!

line vty 1

 login local

 transport input none

!

line vty 2

 login local

 transport input telnet

!

line vty 3

 login local

 transport input ssh

!

line vty 4

 login local

 transport input telnet ssh

!

end

write memory

# IOS_XE_VTY_Line_Transport_Input_Verification_Commands

| Step | Task | Device | Command | Expected Result |

|---:|---|---|---|---|

| 1 | Verify the vty configuration block | IOS XE | `show running-config | section line vty` | VTY lines show `login local` and the intended `transport input` setting |

| 2 | Verify allowed inbound transports on vty 0 | IOS XE | `show line vty 0 | include Allowed` | Output lists the expected allowed input transports |

| 3 | Verify all line usage | IOS XE | `show line` | Active lines show an asterisk and unused vty lines remain available |

| 4 | Verify active remote users | IOS XE | `show users` | Current console, Telnet, or SSH sessions are listed |

| 5 | Test Telnet when Telnet is allowed | Client or peer IOS XE | `telnet <DEVICE_IP>` | Session opens and prompts for username and password |

| 6 | Confirm Telnet is refused when Telnet is not allowed | Client or peer IOS XE | `telnet <DEVICE_IP>` | Connection fails when vty lines use `transport input ssh` or `none` |

| 7 | Test SSH when SSH is allowed | Client or peer IOS XE | `ssh -l <USER> <DEVICE_IP>` | Session opens and authenticates with the local username |

| 8 | Confirm SSH is refused when SSH is not allowed | Client or peer IOS XE | `ssh -l <USER> <DEVICE_IP>` | Connection fails when vty lines use `transport input telnet` or `none` |

| 9 | Verify SSH process details when SSH is part of the lab | IOS XE | `show ip ssh` | SSH version and key status are displayed |

| 10 | Verify active SSH sessions when SSH is connected | IOS XE | `show ssh` | Active SSH sessions are listed |

| 11 | Verify vty source ACL if used | IOS XE | `show access-lists <ACL_NUM>` | ACL has expected permit or deny hit counters |

| 12 | Verify vty source ACL attachment if used | IOS XE | `show running-config | section line vty` | VTY block shows `access-class <ACL_NUM> in` |

# IOS_XE_VTY_Line_Transport_Input_Rollback

  

configure terminal

!

line vty 0 4

 no access-class <ACL_NUM> in

 no exec-timeout

 transport input none

 no login local

 exit

!

no access-list <ACL_NUM>

!

end

write memory

# IOS_XE_VTY_Line_Transport_Input_Rollback_To_SSH_Only

  

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

# IOS_XE_VTY_Line_Transport_Input_Failure_Checks

| Symptom | Device | Command | Likely Cause | Corrective Action |

|---|---|---|---|---|

| Telnet connection is refused | IOS XE | `show line vty 0 | include Allowed` | VTY lines do not allow Telnet | Configure `transport input telnet` or `transport input telnet ssh` for Telnet labs |

| SSH connection is refused | IOS XE | `show line vty 0 | include Allowed` | VTY lines do not allow SSH | Configure `transport input ssh` or `transport input telnet ssh` |

| Both Telnet and SSH are refused | IOS XE | `show running-config | section line vty` | VTY lines use `transport input none` | Change to the intended allowed protocol |

| SSH prompts fail after password entry | IOS XE | `show running-config | section line vty` | VTY line uses `login` instead of `login local` | Configure `login local` and verify a local username exists |

| Telnet prompts for password only | IOS XE | `show running-config | section line vty` | VTY line uses line password authentication instead of local username authentication | Configure `login local` |

| Login succeeds but user lands at `>` prompt | IOS XE | `show running-config | include ^username` | Username lacks privilege 15 | Configure `username <USER> privilege 15 secret <SECRET>` |

| New sessions fail while old sessions work | IOS XE | `show users` | All matching vty lines are busy | Clear stale sessions or expand available vty lines if supported |

| Per-line lab behaves inconsistently | IOS XE | `show line` | IOS XE selects first available vty line that allows the requested protocol | Check which vty lines are available and which transports each line permits |

| Telnet fails but ping works | IOS XE or path device | `show access-lists` | ACL, vty `access-class`, or path filter blocks TCP/23 | Permit TCP/23 from the lab client only if Telnet is required |

| SSH fails but ping works | IOS XE or path device | `show access-lists` | ACL, vty `access-class`, or path filter blocks TCP/22 | Permit TCP/22 from the management client |

| SSH still fails after `transport input ssh` | IOS XE | `show ip ssh` | SSH service prerequisites are missing | Configure SSH-specific requirements in the SSH note: hostname, domain name, RSA keys, SSHv2, and local user |

| Outbound Telnet from the device fails | IOS XE | `show line vty 0 | include Allowed` | `transport output telnet` is not enabled for outbound use from that terminal line | Configure outbound transport only if the lab requires device-initiated Telnet |

| Outbound SSH from the device fails | IOS XE | `show line vty 0 | include Allowed` | `transport output ssh` is not enabled for outbound use from that terminal line | Configure outbound transport only if the lab requires device-initiated SSH |

| Remote access works from wrong source | IOS XE | `show running-config | section line vty` | No vty `access-class` source restriction exists | Add and apply a management-source ACL with `access-class <ACL_NUM> in` |

##### Source_Basis

# IOS_XE_VTY_Line_Transport_Input_Mental_Model

# IOS_XE_VTY_Line_Transport_Input_Configuration_Checklist

# IOS_XE_VTY_Line_Transport_Input_Skeleton

# IOS_XE_VTY_Line_Transport_Input_Skeleton_Telnet_Lab_Only

# IOS_XE_VTY_Line_Transport_Input_Skeleton_Mixed_Telnet_And_SSH_Lab

# IOS_XE_VTY_Line_Transport_Input_Skeleton_Per_Line_Behavior_Lab

# IOS_XE_VTY_Line_Transport_Input_Verification_Commands

# IOS_XE_VTY_Line_Transport_Input_Rollback

# IOS_XE_VTY_Line_Transport_Input_Rollback_To_SSH_Only

# IOS_XE_VTY_Line_Transport_Input_Failure_Checks
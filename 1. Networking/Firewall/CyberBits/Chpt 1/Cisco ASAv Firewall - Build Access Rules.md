```
=============================================================
CISCO ASAv FIREWALL LAB - FULL WALKTHROUGH
=============================================================

ENVIRONMENT:
- Windows 2019 Workstation (Win_Station)
- Ubuntu Workstation
- Cisco ASAv Firewall (mgmt.asav.internal | admin / P@ssw0rd)
- IIS Web Server (IIS_Server | admin / P@ssw0rd)

=============================================================
QUESTION 1 - Network Objects & Access Rules
=============================================================

OBJECTIVE:
Resolve IP addresses of Win_Station and IIS_Server, create
network objects, update access rule, then browse to IIS_Server.

STEPS:
1. Open CMD and run:
   ping Win_Station
   ping IIS_Server
   Note the third octet of each IP address.

2. Navigate to:
   Configuration → Firewall → Objects → Network Objects/Groups

3. Add Network Object:
   Name: All_Workstations
   Type: Network
   IP Address: 10.233.XX.0 (XX = third octet of Win_Station IP)
   Netmask: 255.255.254.0
   Click Ok

4. Add Network Object:
   Name: All_Servers
   Type: Network
   IP Address: 10.233.XX.0 (XX = third octet of IIS_Server IP)
   Netmask: 255.255.254.0
   Click Ok → Apply

5. Navigate to Access Rules (top left)
   Double-click the Workstations network first rule
   Change Source: any → All_Workstations
   Change Destination: any → All_Servers
   Click Ok → Apply

6. Open Chrome and browse to http://IIS_Server

ANSWER:
The site can't be reached (ERR_CONNECTION_TIMED_OUT)

REASON:
Restricting from "any/any" to specific network objects broke
connectivity because the ASAv had no complete matching permit
rule for the traffic — the service/protocol was not yet defined.

=============================================================
QUESTION 2 - Adding HTTP Service to Access Rule
=============================================================

OBJECTIVE:
Add HTTP (TCP/80) to the Workstations first rule and browse
to IIS_Server again.

STEPS:
1. Double-click the Workstations network first rule
2. In the Service section, add HTTP (TCP/80)
3. Click Ok → Apply
4. Open Chrome and browse to http://IIS_Server
   (Wait up to 1 minute)

ANSWER:
"cyberbit"

REASON:
All four required rule components now defined:
- Source:      All_Workstations
- Destination: All_Servers
- Service:     HTTP (TCP/80)
- Action:      Allow
Firewall passes traffic, IIS responds, page loads.

=============================================================
QUESTION 3 - Creating RDP Service Object & Testing RDP
=============================================================

OBJECTIVE:
Create an RDP service object, add it to the Workstations
first rule, and test RDP connectivity to IIS_Server.

STEPS:
1. On the Access Rules page, go to the Services tab (top right)
2. Click Add → Service Object:
   Name: RDP
   Service Type: TCP
   Destination Port: 3389
   Click Ok → Apply

3. Double-click the Workstations first rule
4. Add RDP to the Service section alongside HTTP
5. Click Ok → Apply

6. Open CMD/Run/PowerShell and run:
   mstsc
   Connect to: IIS_Server
   Username: admin
   Password: P@ssw0rd

ANSWER:
Yes, I'm connected to the IIS_Server via RDP

REASON:
Rule now permits TCP/3389 from All_Workstations to All_Servers.
IIS_Server has RDP enabled. Credentials are valid.

=============================================================
QUESTION 4 - SMB Rule & File Share Access
=============================================================

OBJECTIVE:
Create a rule to allow SMB from Servers to Workstations,
then browse \\Win_Station\c$\Program Files from IIS_Server.

STEPS:
1. In Access Rules, click Add (top left)
2. Choose the Servers interface
3. Set:
   Source:      All_Servers
   Destination: All_Workstations
   Service:     SMB (remove "ip", select "SMB")
   Action:      Allow
4. Click Ok → Apply

5. From inside the RDP session on IIS_Server:
   Open File Explorer or Run
   Navigate to: \\Win_Station\c$\Program Files

ANSWER:
Amazon and Cisco (folders present in the SMB path)

REASON:
The SMB rule (TCP/445) from Servers → Workstations allowed
the share to be accessed. Amazon and Cisco are the installed
application folders present under Program Files on Win_Station.

=============================================================
QUESTION 5 - Deny ICMP Rule & Ping Test
=============================================================

OBJECTIVE:
Create a deny ICMP rule above existing rules on the Servers
interface, then ping IIS_Server from the workstation.

STEPS:
1. Right-click the top rule in the Servers network section
2. Click Insert...
3. Set:
   Action:      Deny
   Source:      All_Servers
   Destination: All_Workstations
   Service:     ICMP (remove "ip", select "icmp")
4. Click Ok → Apply
5. If rule appears as #2, use Move Up arrow to make it #1

6. Open CMD and run:
   ping IIS_Server

ANSWER:
Yes - It fails because Ping needs ICMP to be allowed for both sides

REASON:
ICMP is stateless — no connection tracking.
- Echo Request:  Workstation → IIS_Server (permitted)
- Echo Reply:    IIS_Server → Workstation (DENIED by new rule)
The reply is blocked. Ping fails regardless of origin.

=============================================================
QUESTION 6 - Add TCP to Deny Rule & RDP Test
=============================================================

OBJECTIVE:
Add TCP to the existing ICMP deny rule, then attempt RDP
to IIS_Server via MSTSC.

STEPS:
1. Double-click the deny rule created in Question 5
2. Click "..." next to the Service section
3. Locate "tcp" and double-click to add it
4. Click Ok → Ok → Apply

5. Open CMD/Run and launch:
   mstsc
   Connect to: IIS_Server
   Username: admin
   Password: P@ssw0rd

ANSWER:
Success - Because denying TCP from the "Servers" network does
not affect TCP from the "Workstations" network

REASON:
TCP is stateful. RDP originates FROM the workstation.
The firewall tracks the session — return traffic from IIS_Server
is recognized as part of an established connection, not as new
traffic originating from the Servers network.
The deny rule only hits NEW connections initiated from Servers.

=============================================================
QUESTION 7 - Simulated Reboot (No Save)
=============================================================

OBJECTIVE:
Simulate an unexpected reboot and observe the result.

STEPS:
1. Navigate to Tools → System Reload...
2. Change Configuration State to:
   "Reload without saving the running configuration"
3. Click Schedule Reload → Yes
4. Do NOT exit ASDM — close the pop-up only
5. Wait up to 2 minutes for ASDM to return
6. Navigate back to Access Rules

ANSWER:
Yes! All the changes made in the lab are gone

REASON:
Apply writes changes to RAM (running config) only.
Rebooting without saving wipes RAM.
Nothing was written to flash (permanent storage).
Device boots from last saved startup config — original state.

Save properly using Ctrl+S or the floppy disk icon to write
running config to flash and survive reboots.

=============================================================
KEY CONCEPTS SUMMARY
=============================================================

Access Rule requires 4 components:
  1. Source
  2. Destination
  3. Service/Protocol
  4. Action (Allow/Deny)

Service Types:
  ICMP     → Ping, Traceroute
  TCP      → RDP (3389), HTTP (80), SMB (445), FTP, Telnet
  UDP      → NTP, Syslog, RADIUS, SNMP
  TCP+UDP  → All TCP and UDP
  IP       → All protocols

ICMP vs TCP on a stateful firewall:
  ICMP → Stateless. Both directions must be explicitly permitted.
  TCP  → Stateful. Return traffic tracked via session table.

Memory:
  Apply   → Running config (RAM)  — lost on reboot
  Ctrl+S  → Startup config (Flash) — survives reboot

=============================================================
```
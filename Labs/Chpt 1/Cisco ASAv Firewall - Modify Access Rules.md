1. RDP question
Change Workstations first incoming rule from ICMP-only to ICMP + TCP, then retry RDP to IIS_Server.
Expected answer:
Success, because allowing all TCP protocol-based traffic includes RDP.

2. Disable Servers ICMP rule question
Disable the first Servers incoming ICMP rule, then ping IIS_Server.
Cyberbit marked:
Yes, it became "Request timed out."
Reason:
Their lab treats ICMP as needing both directions explicitly allowed.

3. Enable Global ICMP rule question
Enable first Global ICMP permit rule, then ping IIS_Server.
Expected answer:
Success, the destination replies.
Reason:
Global ICMP permit catches ICMP when no earlier relevant deny blocks it.

4. Servers ICMP deny while Global ICMP permit exists
Change Servers first ICMP rule to Deny and enable it, then ping IIS_Server.
Cyberbit marked:
Failure, "Destination host unreachable."
Your actual VM showed:
Failure, "Request timed out."
Concept:
Specific Servers deny beats later Global permit. Your screenshot proved this with deny hit counts.

5. Saturdays Only time range on RDP/TCP rule
Add "Saturdays Only" time range to the Workstations rule that allowed TCP/RDP, then retry RDP.
Expected answer:
Failure, the connection does not open/fails.
Reason:
The permit rule is inactive unless current firewall time matches Saturday.

6. Create Win_Station and IIS_Server objects, permit HTTP
Resolve both IPs, create host objects, edit Workstations first incoming rule:
Source Win_Station
Destination IIS_Server
Service HTTP tcp/80
No time range
Then browse http://IIS_Server.
Expected answer:
Likely "Welcome to the homepage."
Caveat:
The exact page text depends on what Cyberbit put on IIS. Firewall logic says it should load, not timeout.

7. SMB question
Allow SMB from Workstations to Servers, browse \\IIS_Server\c$, identify folders.
Likely expected folders:
Program Data
inetpub
Possible extra:
temp, only if visibly present in the share.
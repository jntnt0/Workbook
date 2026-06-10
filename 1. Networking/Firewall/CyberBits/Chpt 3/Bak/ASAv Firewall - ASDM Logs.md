================================================================================
CISCO ASDM SYSLOG LAB - QUESTIONS, STEPS & ANSWERS
================================================================================

NETWORK TOPOLOGY:
- Windows 2019 Workstation (connected to Cisco ASAv Firewall)
- Cisco ASAv Firewall (central device)
- Ubuntu Workstation
- IIS Web Server

CREDENTIALS:
- ASAv Management: admin / P@ssw0rd
- IIS Server:      admin / P@ssw0rd
- Trainee Windows: admin / P@ssw0rd

================================================================================
Q1: What is the first event's description after enabling ASDM logging?
================================================================================
STEPS:
1. Log into the ASDM console
2. On the Home screen, scroll to the bottom
3. Click "Enable ASDM Logging"
4. Examine the Latest ASDM Syslog Messages

ANSWER: "User 'admin' executed the 'logging asdm informational' command."
REASON: Clicking "Enable ASDM Logging" runs the 'logging asdm informational'
        command, which is recorded as the first syslog entry.

================================================================================
Q2: What are the severities of the successful ping and unsuccessful TCP test?
================================================================================
STEPS:
1. Open PowerShell
2. Run: Test-NetConnection -ComputerName iis_server -Port 80
3. Go back to ASDM Home screen
4. Examine the top Syslog messages and check the first column (severity)

ANSWER: The ping severity is 6, and the TCP test is 4.
REASON:
- Ping (ICMP) success = %ASA-6-302020 = Severity 6 (Informational)
- TCP denied       = %ASA-4-106023 = Severity 4 (Warning)

SEVERITY SCALE:
1 = Alert
2 = Critical
3 = Error
4 = Warning
5 = Notification
6 = Informational
7 = Debugging

================================================================================
Q3: What is the default foreground color of the most severe (0 - Emergencies) 
    log level?
================================================================================
STEPS:
1. Inside "Latest ASDM Syslog Messages", right-click any log
2. Click "Color Settings"
3. Find the Emergencies (level 0) foreground color at the top

ANSWER: Dark red

================================================================================
Q4: What is the explanation for the log when accessing IIS_Server via SMB 
    (destination port 445)?
================================================================================
STEPS:
1. Open File Explorer
2. Navigate to \\IIS_Server\c$
3. Enter credentials: admin / P@ssw0rd
4. In ASDM, hover over a log with destination port 445

ANSWER: "A TCP connection slot between two hosts was created."
REASON: Corresponds to %ASA-6-302013, logged when a successful TCP connection
        is established (SMB uses port 445).

================================================================================
Q5: What logs appear in the exported text file after using "Save Content"?
================================================================================
STEPS:
1. Right-click a specific log
2. Click "Save Content"
3. Name it "logs.txt" and save to desktop
4. Open the file from the desktop

ANSWER: The same "latest" logs that appeared on the home screen.
REASON: "Save Content" exports the entire contents of the Latest ASDM Syslog
        Messages panel as displayed — not just the one right-clicked entry.

================================================================================
Q6: What is the explanation for the first two severity 7 logs (Syslog ID 111009)?
================================================================================
STEPS:
1. Right-click a log → "Clear Content"
2. Click "Configure ASDM Syslog Filters" (gear icon)
3. Double-click "ASDM" logging
4. Change "Filter on severity" to Debugging
5. Click Ok → Apply
6. Go to Home screen, hover over the two Syslog ID 111009 logs

ANSWER: "The user ran a command, but no actual changes were made to the 
        configuration."
REASON: Syslog ID 111009 (Severity 7/Debugging) logs read-only commands such
        as 'show' commands — actions that retrieve info without modifying config.

================================================================================
Q7: What is the severity and description of all events when filtering for 
    Syslog ID 106023 after running Test-NetConnection to port 80?
================================================================================
STEPS:
1. Right-click → "Clear Content"
2. Click "Configure ASDM Syslog Filters" (gear icon)
3. Double-click "ASDM" logging
4. Change mode from "Filter on severity" to "Use event list"
5. Click New → Name it "Only106023"
6. Click Add (under Message ID Filters)
7. In "Message IDs" field, type: 106023
8. Click Ok (three times) → Apply
9. Open PowerShell, run: Test-NetConnection -ComputerName iis_server -Port 80
10. Examine Latest ASDM Syslog Messages on Home screen

ANSWER: Severity 4, "Deny tcp src Workstations dst Servers"
REASON: %ASA-4-106023 is exclusively a Severity 4 (Warning) message, generated
        when the ASA denies traffic via ACL. Since port 80 is blocked and the
        filter only shows 106023, every log will be this same denied TCP event.

================================================================================
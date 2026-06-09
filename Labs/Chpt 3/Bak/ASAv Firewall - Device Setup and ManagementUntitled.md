```
=======================================================================
CISCO ASAv LAB - QUESTIONS, STEPS & ANSWERS (CHRONOLOGICAL)
=======================================================================

-----------------------------------------------------------------------
Q1: Ping IIS_Server and run pathping. What is the last stop of the ICMP?
-----------------------------------------------------------------------
STEPS:
  1. Open CMD/PowerShell
  2. Run: ping IIS_Server (will time out)
  3. Run: pathping IIS_Server
  4. Look for the last hop that is NOT only "* * *"

ANSWER: Win_Station
  - Every hop beyond Win_Station returns only "* * *"
  - The ICMP never successfully leaves the source machine

-----------------------------------------------------------------------
Q2: Log in to Cisco ASDM and check interfaces. Is there a problem?
-----------------------------------------------------------------------
STEPS:
  1. Open ASDM (desktop shortcut)
  2. Navigate to: Configuration → Device Setup → Interface Setup → Interfaces
  3. Compare TenGigabitEthernet interfaces for missing information

ANSWER: Yes, TenGigabitEthernet0/0 has no name or IP address.
  - Without a nameif and IP, the interface cannot pass traffic
  - DHCP identifier formula requires: <MAC><interface_name>-<host>
  - Missing interface name = DHCP cannot assign an IP

-----------------------------------------------------------------------
Q3: Name the interface "Workstations", apply, then ping IIS_Server again.
    What is the result?
-----------------------------------------------------------------------
STEPS:
  1. In ASDM: Configuration → Device Setup → Interface Setup → Interfaces
  2. Double-click TenGigabitEthernet0/0 to edit
  3. In "Interface Name" field, type: Workstations
  4. Save and apply changes
  5. Open CMD/PowerShell
  6. Run: ping IIS_Server

ANSWER: The ping succeeds now.
  - Naming the interface completes the DHCP identifier formula
  - Interface obtains an IP from DHCP server
  - Routing between Win_Station and IIS_Server now works end-to-end

-----------------------------------------------------------------------
Q4: Change the enable password to "10203040", then re-open ASDM with
    original credentials. What is the result?
-----------------------------------------------------------------------
STEPS:
  1. In ASDM: Configuration → Device Setup → Device Name/Password
  2. Under "Enable Password", check "Change the privileged mode password"
  3. Enter "10203040" in both password fields
  4. Apply changes
  5. Close ASDM
  6. Reopen ASDM using:
       Device Name: mgmt.asav.internal
       Username:    admin
       Password:    P@ssw0rd

ANSWER: Success - using "admin" and "P@ssw0rd"
  - The enable password and user login credentials are separate
  - Enable password = CLI privileged mode only (not ASDM login)
  - User login credentials (admin/P@ssw0rd) were unchanged

-----------------------------------------------------------------------
Q5: Edit the "admin" user account. Which access restriction options
    are available?
-----------------------------------------------------------------------
STEPS:
  1. In ASDM: Configuration → Device Management → Users/AAA → User Accounts
  2. Double-click the "admin" user
  3. Check "Change user password" box
  4. Enter new password in both fields
  5. Examine the "Access Restriction" section below the password fields

ANSWER: (Multiple correct)
  - Full access (ASDM, SSH, Telnet and Console)
  - CLI login prompt for SSH, Telnet and Console
  - No ASDM, SSH, Telnet or Console access

  NOTE: "ASDM access only" is NOT an available option.

-----------------------------------------------------------------------
Q6: Add SSH management access via Startup Wizard, then SSH into ASAv.
    What is the result?
-----------------------------------------------------------------------
STEPS:
  1. Open CMD/PowerShell
  2. Run: ssh admin@mgmt.asav.internal (will fail initially)
  3. In ASDM: Configuration → Device Setup → Startup Wizard
  4. Click "Launch Startup Wizard"
  5. Click Next until Step 8 (Management Access)
  6. Click Add and configure:
       Access Type:    SSH
       Interface Name: management
       IP Address:     0.0.0.0
       Subnet Mask:    0.0.0.0
  7. Click Finish and apply
  8. Run: ssh admin@mgmt.asav.internal again

ANSWER: Success, I was able to log in successfully.
  - SSH is now permitted on the management interface
  - 0.0.0.0/0.0.0.0 permits SSH from any source IP
  - Admin credentials authenticate successfully

-----------------------------------------------------------------------
Q7: Check available protocols for a new AAA Server Group. Which are
    available?
-----------------------------------------------------------------------
STEPS:
  1. In ASDM: Configuration → Device Management → Users/AAA → AAA Server Groups
  2. Click "Add"
  3. View the "Protocol" drop-down menu options

ANSWER: (Multiple correct)
  - TACACS+
  - HTTP Form
  - RADIUS
  - LDAP

  NOT available: OAUTH, SAML

-----------------------------------------------------------------------
Q8: Change SSL cipher to "High", then browse https://mgmt.asav.internal
    in Chrome. What is the result?
-----------------------------------------------------------------------
STEPS:
  1. In ASDM: Configuration → Device Management → Advanced → SSL Settings
  2. Double-click "Default" cipher
  3. Change "SSL cipher security level" to "High"
  4. Click OK and apply
  5. Open Chrome
  6. Navigate to: https://mgmt.asav.internal

ANSWER: "Your connection is not private", but I'm able to proceed.
  - Modern Chrome/Windows supports high ciphers natively - no issue there
  - Warning is caused by ASA's self-signed certificate (not trusted by Chrome)
  - Can bypass via: Advanced → Proceed to mgmt.asav.internal

=======================================================================
END OF LAB
=======================================================================
```
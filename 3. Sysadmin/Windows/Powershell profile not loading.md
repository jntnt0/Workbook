###### PowerShell Startup Environment Not Loading (Profile Skipped)

## Overview

Use this when a PowerShell session opens without the expected environment — aliases, functions, modules, or startup messages are missing. The root cause is almost always that the profile was skipped, not that the profile itself is broken.

---

## Primary Symptom

Things expected from the PowerShell environment are missing immediately after opening a terminal:

- Aliases or functions not present
- Startup confirmation message did not print
- Modules or paths feel inconsistent between sessions

---

## First Triage — Run These First

**Confirm you are in pwsh and what host launched it:**

```powershell
$PSVersionTable.PSVersion
$host.Name
```

**Confirm which profile file this host should load:**

```powershell
$PROFILE
Test-Path $PROFILE
```

**Confirm whether the profile ran automatically:**

- Did a startup message print on open?
- If not, the profile was skipped — work through the decision tree below.

---

## Root Cause Decision Tree

### A — pwsh Launched with -NoProfile (Most Common)

**Check:**

```powershell
Get-Process -Id $PID | Select-Object -ExpandProperty CommandLine
```

If the output includes `-NoProfile`, that is the cause.

**Temporary fix (current session only):**

```powershell
. $PROFILE
```

**Permanent fix:**

Remove `-NoProfile` from whatever launches pwsh — Windows Terminal profile settings, a desktop shortcut, VS Code terminal settings, or a task scheduler entry. `-NoLogo` is safe to keep.

Correct command line:

```
"C:\Program Files\PowerShell\7\pwsh.exe" -NoLogo
```

**Verify:**

```powershell
Get-Process -Id $PID | Select-Object -ExpandProperty CommandLine
```

The startup message should now appear automatically on next open, and profile-defined functions should be available without manual dot-sourcing.

---

### B — Execution Policy Blocked the Profile (Less Common)

**Check:**

```powershell
Get-ExecutionPolicy -List
```

If `CurrentUser` is `Restricted` or `AllSigned` and the profile is unsigned, it will not run.

**Fix:**

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

---

### C — Wrong Profile File Edited

Different hosts use different profile paths. ConsoleHost, VS Code, and ISE each load a different `$PROFILE`.

**Check all profile locations:**

```powershell
$PROFILE | Format-List *
```

Edit the profile path that corresponds to the host you are actually using.

---

### D — PSModulePath Overwritten (Profile Runs but Modules Missing)

**Symptom:** Profile loads correctly but modules cannot be found.

**Check:**

```powershell
($env:PSModulePath -split ';') | Select-Object -Unique
Get-Module -ListAvailable <module-name> | Select-Object Name, Version, Path
```

**Fix:**

Ensure `PSModulePath` is appended or preserved in the profile, not overwritten. The user modules path must be present:

```
$HOME\Documents\PowerShell\Modules
```

Correct pattern in profile:

```powershell
$env:PSModulePath = "$HOME\Documents\PowerShell\Modules;" + $env:PSModulePath
```

---

## Known-Good Incident Record

|Field|Detail|
|---|---|
|Problem|pwsh launched with `-NoProfile` — profile never ran|
|Evidence|`Get-Process -Id $PID \| Select-Object -ExpandProperty CommandLine` showed `-NoProfile`|
|Fix|Removed `-NoProfile` from Windows Terminal profile command line|
|Verification|Startup message appeared automatically; profile-defined commands available immediately|

---

## Verification Checklist

Run after applying a fix to confirm the environment is in a known-good state:

```powershell
# Confirm profile is not being skipped
Get-Process -Id $PID | Select-Object -ExpandProperty CommandLine

# Confirm profile path exists
Test-Path $PROFILE

# Confirm a profile-dependent command is available
Get-Command <your-profile-function> -ErrorAction SilentlyContinue

# Confirm module availability
Get-Module -ListAvailable <module-name> | Select-Object Name, Version
```

On a correctly configured session:

- Command line does **not** include `-NoProfile`
- Startup message prints automatically on open
- Profile-defined aliases and functions are available without manual dot-sourcing
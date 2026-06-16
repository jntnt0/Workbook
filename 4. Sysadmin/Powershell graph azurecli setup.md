###### PowerShell 7 + Microsoft Graph + Azure CLI Setup and Troubleshooting

## Overview

Reference for getting a Windows PowerShell 7 environment into a known-good state for Microsoft Graph and Azure CLI work. Covers module installation, profile loading issues, Graph authentication failures, and Azure CLI path problems.

---

## 1 — PSGallery Install Prompts

### Symptom

Repeated confirmation prompts when installing modules from PSGallery.

### Fix

```powershell
Set-PSRepository -Name PSGallery -InstallationPolicy Trusted
```

### Verify

```powershell
Get-PSRepository | Select-Object Name, InstallationPolicy
```

---

## 2 — Microsoft Graph PowerShell Module Install

### Install

```powershell
Install-Module Microsoft.Graph -Scope CurrentUser
```

### Verify

```powershell
Get-Module -ListAvailable Microsoft.Graph.Authentication | Select-Object Name, Version, Path
Get-InstalledModule Microsoft.Graph* | Select-Object Name, Version | Sort-Object Name
```

---

## 3 — Profile Not Loading

### Symptom

- Profile-dependent functions or aliases missing on startup
- Having to run `. $PROFILE` manually each session

### Root Cause

PowerShell is being launched with `-NoProfile`. Confirm with:

```powershell
Get-Process -Id $PID | Select-Object -ExpandProperty CommandLine
```

**Problem — profile disabled:**

```
"C:\Program Files\PowerShell\7\pwsh.exe" -NoLogo -NoProfile
```

**Correct — profile enabled:**

```
"C:\Program Files\PowerShell\7\pwsh.exe" -NoLogo
```

### Fix

Remove `-NoProfile` from the launcher — Windows Terminal profile settings, shortcut target, or whatever is invoking pwsh.

### Verify

```powershell
Get-Command <your-profile-function>
```

Profile-defined functions should be available immediately on shell open without manual dot-sourcing.

---

## 4 — Graph Authentication Failures

### 4A — InteractiveBrowser / WAM Broker Failure

#### Symptom

`Connect-MgGraph` fails with:

- `InteractiveBrowserCredential authentication failed`
- MSAL broker or WAM-related error messages

#### Debug

```powershell
try {
    Connect-MgGraph -Scopes "AuditLog.Read.All" -ErrorAction Stop
} catch {
    $_ | Format-List * -Force
}
```

#### Resolution

Fix profile loading first (see Section 3). Once the profile loads correctly, interactive sign-in works:

```powershell
Connect-MgGraph -Scopes "AuditLog.Read.All"
Get-MgContext
```

---

### 4B — Device Code Connects but API Calls Fail

#### Symptom

Device code flow completes successfully ("Welcome to Microsoft Graph"), but the first API call fails with:

```
DeviceCodeCredential authentication failed: Object reference not set to an instance of an object.
```

This affects both:

- `Get-MgAuditLogSignIn`
- `Invoke-MgGraphRequest`

#### Resolution

Do not use device code flow for audit log calls in this environment. Use interactive login instead:

```powershell
Connect-MgGraph -Scopes "AuditLog.Read.All"
```

#### Verify Graph Context and Scopes

```powershell
Get-MgContext | Select-Object Account, TenantId, Scopes, TokenCredentialType, WamEnabled
```

---

## 5 — Audit Log Query Test

```powershell
Get-MgAuditLogSignIn -Top 1 | Select-Object CreatedDateTime, UserDisplayName, AppDisplayName, Status
```

### Interpreting Failures

|Error|Meaning|
|---|---|
|403 / Authorization_RequestDenied|Permissions, role assignment, or admin consent issue|
|Null reference on API call|Local Graph PowerShell auth/client bug — not a permissions problem|

---

## 6 — Azure CLI Not Recognized

### Symptom

```powershell
az login
# az: The term 'az' is not recognized...
```

### Cause

Azure CLI not installed, or PATH not updated in the current session.

### Install

```powershell
winget install -e --id Microsoft.AzureCLI
```

### Fix PATH Without Reopening Shell

```powershell
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + `
            [System.Environment]::GetEnvironmentVariable("Path","User")
```

### Verify

```powershell
az version
az login
az account show
```

---

## 7 — Evidence Directory Convention

When running scripts that produce output, use a consistent directory structure to separate raw output from committed summaries:

```powershell
$slug = "<scenario-name>"
$out  = Join-Path $PWD "evidence\$slug\raw"
New-Item -ItemType Directory -Force -Path $out | Out-Null
```

Add the following to `.gitignore` to prevent raw logs from being committed accidentally:

```
evidence/**/raw/*
```

Commit only sanitized summaries under `evidence/<slug>/summary/`.

---

## 8 — Recovering Command History After Closing the Shell

PowerShell command history is stored by PSReadLine and persists across sessions.

### Find the History File

```powershell
(Get-PSReadLineOption).HistorySavePath
```

### View Last N Commands

```powershell
Get-Content (Get-PSReadLineOption).HistorySavePath -Tail 300
```

---

## Known-Good Baseline Sequence

Run this after opening a new shell to confirm the environment is in a working state:

```powershell
# 1. Confirm profile loaded
Get-Command <your-profile-function>

# 2. Connect to Graph and verify
Connect-MgGraph -Scopes "AuditLog.Read.All"
Get-MgContext
Get-MgAuditLogSignIn -Top 1

# 3. Confirm Azure CLI
az version
az login
az account show
```

---

## Notes

- Launching pwsh from inside a repo folder only sets the starting working directory for that session — it does not persist
- If your profile contains a `Set-Location` call, it will override the working directory on startup — adjust or remove it when working from a specific repo path
- Device code flow (`-UseDeviceCode`) is unreliable for audit log calls in environments where WAM is active — use interactive login
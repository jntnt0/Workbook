# =============================================================================
# cleanup_old_user_profiles.ps1
# Remove stale user profile directories and user-owned files based on a
# configurable last-write/last-access cutoff date.
#
# Overview:
#   On shared or managed Windows endpoints, user profile directories under
#   C:\Users accumulate over time as staff turn over or machines are
#   reassigned. This script walks the base user directory, identifies
#   subdirectories and common user files that have not been accessed since
#   the cutoff date, and removes them. Exclusion patterns protect system and
#   administrative profiles from accidental deletion.
#
# Usage:
#   Run as a local administrator or via a privileged scheduled task.
#
#       .\cleanup_old_user_profiles.ps1
#       .\cleanup_old_user_profiles.ps1 -CutoffDate "2024-01-01" -WhatIf
#
# Parameters:
#   -BaseDirectory      Root path to scan. Default: C:\Users
#   -CutoffDate         Profiles and files last modified before this date are
#                       targeted for removal. Default: 2023-01-01
#   -ExcludePatterns    Wildcard patterns for profile folders to skip.
#                       Default: *TEMP*, *admin*, *Public*, *OneDrive*
#   -WhatIf             Report what would be removed without deleting anything.
#
# Notes:
#   Test with -WhatIf before running in production. The script does not
#   remove the Windows registry hive (NTUSER.DAT) — use the built-in
#   User Profile control panel or MDM policy for full profile deprovisioning
#   where registry cleanup is also required.
# =============================================================================

[CmdletBinding(SupportsShouldProcess)]
param (
    [string]$BaseDirectory   = "C:\Users",
    [datetime]$CutoffDate    = (Get-Date "2023-01-01"),
    [string[]]$ExcludePatterns = @("*TEMP*", "*admin*", "*Public*", "*OneDrive*")
)

# ---------------------------------------------------------------------------
# Tracking counters for the summary report
# ---------------------------------------------------------------------------
$RemovedCount = 0
$SkippedCount = 0
$ErrorCount   = 0

# ---------------------------------------------------------------------------
# Helper: test whether a directory name matches any exclusion pattern
# ---------------------------------------------------------------------------
function Test-Excluded {
    param ([string]$Name)
    foreach ($Pattern in $ExcludePatterns) {
        if ($Name -like $Pattern) { return $true }
    }
    return $false
}

# ---------------------------------------------------------------------------
# Helper: write a timestamped console entry
# ---------------------------------------------------------------------------
function Write-Log {
    param (
        [string]$Message,
        [ValidateSet("Info", "Warn", "Skip", "Error")]
        [string]$Level = "Info"
    )
    $Prefix = switch ($Level) {
        "Info"  { "[INFO ]" }
        "Warn"  { "[WARN ]" }
        "Skip"  { "[SKIP ]" }
        "Error" { "[ERROR]" }
    }
    Write-Host "$(Get-Date -Format 'HH:mm:ss') $Prefix $Message"
}

# ---------------------------------------------------------------------------
# Phase 1 — Remove stale profile directories
# Targets top-level subdirectories of BaseDirectory whose LastWriteTime
# predates CutoffDate and whose name does not match an exclusion pattern.
# ---------------------------------------------------------------------------
function Remove-StaleProfileDirectories {
    param ([string]$Directory)

    $Subdirectories = Get-ChildItem -Path $Directory -Directory -ErrorAction SilentlyContinue
    if (-not $Subdirectories) { return }

    foreach ($Dir in $Subdirectories) {
        if (Test-Excluded -Name $Dir.Name) {
            Write-Log "Excluded by pattern: $($Dir.FullName)" -Level Skip
            $script:SkippedCount++
            continue
        }

        if ($Dir.LastWriteTime -lt $CutoffDate) {
            if ($PSCmdlet.ShouldProcess($Dir.FullName, "Remove stale profile directory")) {
                try {
                    Remove-Item -Path $Dir.FullName -Recurse -Force -ErrorAction Stop
                    Write-Log "Removed directory: $($Dir.FullName)"
                    $script:RemovedCount++
                }
                catch {
                    Write-Log "Failed to remove '$($Dir.FullName)': $($_.Exception.Message)" -Level Error
                    $script:ErrorCount++
                }
            }
        } else {
            # Directory is within retention window — recurse into it
            Remove-StaleProfileDirectories -Directory $Dir.FullName
        }
    }
}

# ---------------------------------------------------------------------------
# Phase 2 — Remove stale files from common user data folders
# Targets files in Downloads, Documents, Videos, and Pictures whose
# LastAccessTime predates CutoffDate.
# ---------------------------------------------------------------------------
function Remove-StaleUserFiles {
    param ([string]$ProfileDirectory)

    $TargetFolders = @("Downloads", "Documents", "Videos", "Pictures")

    foreach ($Folder in $TargetFolders) {
        $FolderPath = Join-Path -Path $ProfileDirectory -ChildPath $Folder
        $Files = Get-ChildItem -Path $FolderPath -File -ErrorAction SilentlyContinue
        if (-not $Files) { continue }

        foreach ($File in $Files) {
            if ($File.LastAccessTime -lt $CutoffDate) {
                if ($PSCmdlet.ShouldProcess($File.FullName, "Remove stale user file")) {
                    try {
                        Remove-Item -Path $File.FullName -Force -ErrorAction Stop
                        Write-Log "Removed file: $($File.FullName)"
                        $script:RemovedCount++
                    }
                    catch {
                        Write-Log "Failed to remove '$($File.FullName)': $($_.Exception.Message)" -Level Error
                        $script:ErrorCount++
                    }
                }
            }
        }
    }
}

# ---------------------------------------------------------------------------
# Entry point
# ---------------------------------------------------------------------------
if (-not (Test-Path -Path $BaseDirectory)) {
    Write-Log "Base directory '$BaseDirectory' not found. Exiting." -Level Error
    exit 1
}

Write-Log "Scan started. Base: $BaseDirectory | Cutoff: $($CutoffDate.ToString('yyyy-MM-dd')) | WhatIf: $($WhatIfPreference)"

# Phase 1 — profile directories
$UserDirectories = Get-ChildItem -Path $BaseDirectory -Directory -ErrorAction SilentlyContinue
foreach ($UserDir in $UserDirectories) {
    Remove-StaleProfileDirectories -Directory $UserDir.FullName
}

# Phase 2 — files within surviving profile directories
$RemainingDirectories = Get-ChildItem -Path $BaseDirectory -Directory -Recurse -ErrorAction SilentlyContinue
foreach ($Dir in $RemainingDirectories) {
    Remove-StaleUserFiles -ProfileDirectory $Dir.FullName
}

# ---------------------------------------------------------------------------
# Summary
# ---------------------------------------------------------------------------
Write-Log "-----------------------------------------------------------"
Write-Log "Scan complete."
Write-Log "Removed : $RemovedCount items"
Write-Log "Skipped : $SkippedCount items (exclusion patterns)"
Write-Log "Errors  : $ErrorCount items"
if ($WhatIfPreference) {
    Write-Log "WhatIf mode was active — no changes were made." -Level Warn
}

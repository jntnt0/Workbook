# =============================================================================
# disk_cleanup.ps1
# Remove temporary files, cached data, and delivery optimization artifacts
# from a Windows endpoint to reclaim disk space.
#
# Overview:
#   This script targets well-known Windows temporary and cache locations that
#   accumulate over time and are safe to remove without impacting system
#   stability. It is intended for use on managed endpoints where scheduled or
#   on-demand cleanup is required outside of the built-in Disk Cleanup utility
#   (cleanmgr.exe), which cannot be easily scripted or integrated into
#   automation pipelines.
#
# Usage:
#   Run as a local administrator or via a privileged scheduled task.
#
#       .\disk_cleanup.ps1
#       .\disk_cleanup.ps1 -WhatIf
#       .\disk_cleanup.ps1 -LogPath "D:\Logs\disk_cleanup.log"
#
# Parameters:
#   -LogPath    Path to the output log file.
#               Default: C:\Logs\disk_cleanup.log
#   -WhatIf     Report what would be removed without deleting anything.
#
# Targets:
#   - User temp directory (%TEMP%)
#   - Windows temp directory (%SystemRoot%\Temp)
#   - Windows Update download cache (SoftwareDistribution\Download)
#   - Delivery Optimization cache (SoftwareDistribution\DeliveryOptimization)
#   - Windows Error Reporting files (%SystemRoot%\Windows\WER)
#   - DirectX Shader Cache
#   - Windows Explorer thumbnail cache
#   - Recycle Bin
#
# Notes:
#   The script removes files only — directory structures are preserved.
#   Windows Update and Delivery Optimization targets are safe to clear;
#   Windows will re-download any updates that have not yet been applied.
#   The Recycle Bin is emptied system-wide, not per-user.
# =============================================================================

[CmdletBinding(SupportsShouldProcess)]
param (
    [string]$LogPath = "C:\Logs\disk_cleanup.log"
)

# ---------------------------------------------------------------------------
# Tracking counters
# ---------------------------------------------------------------------------
$RemovedCount = 0
$FailedCount  = 0
$BytesFreed   = 0

# ---------------------------------------------------------------------------
# Helper: write a timestamped entry to both the log file and the console
# ---------------------------------------------------------------------------
function Write-Log {
    param (
        [string]$Message,
        [ValidateSet("Info", "Warn", "Error")]
        [string]$Level = "Info"
    )

    $Prefix = switch ($Level) {
        "Info"  { "[INFO ]" }
        "Warn"  { "[WARN ]" }
        "Error" { "[ERROR]" }
    }

    $Entry = "$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss') $Prefix $Message"
    Add-Content -Path $LogPath -Value $Entry -ErrorAction SilentlyContinue
    Write-Host $Entry
}

# ---------------------------------------------------------------------------
# Ensure log directory exists
# ---------------------------------------------------------------------------
$LogDirectory = Split-Path $LogPath -Parent
if (-not (Test-Path -Path $LogDirectory)) {
    try {
        New-Item -ItemType Directory -Path $LogDirectory -Force | Out-Null
    }
    catch {
        Write-Error "Failed to create log directory '$LogDirectory': $($_.Exception.Message)"
        exit 1
    }
}

# ---------------------------------------------------------------------------
# Core cleanup function — removes all files in a given directory
# ---------------------------------------------------------------------------
function Remove-DirectoryContents {
    param (
        [string]$Path,
        [string]$Label
    )

    if (-not (Test-Path -Path $Path)) {
        Write-Log "Path not found, skipping: $Path ($Label)" -Level Warn
        return
    }

    $Files = Get-ChildItem -Path $Path -File -ErrorAction SilentlyContinue

    if (-not $Files) {
        Write-Log "No files found in: $Path ($Label)"
        return
    }

    Write-Log "Cleaning $Label ($($Files.Count) files)..."

    foreach ($File in $Files) {
        if ($PSCmdlet.ShouldProcess($File.FullName, "Remove file")) {
            try {
                $Size = $File.Length
                Remove-Item -Path $File.FullName -Force -ErrorAction Stop
                $script:RemovedCount++
                $script:BytesFreed += $Size
            }
            catch {
                Write-Log "Failed to remove '$($File.FullName)': $($_.Exception.Message)" -Level Error
                $script:FailedCount++
            }
        }
    }
}

# ---------------------------------------------------------------------------
# Recycle Bin cleanup
# ---------------------------------------------------------------------------
function Clear-RecycleBinSafe {
    if ($PSCmdlet.ShouldProcess("Recycle Bin", "Empty")) {
        try {
            Clear-RecycleBin -Force -ErrorAction Stop
            Write-Log "Recycle Bin emptied."
        }
        catch {
            Write-Log "Failed to empty Recycle Bin: $($_.Exception.Message)" -Level Error
            $script:FailedCount++
        }
    }
}

# ---------------------------------------------------------------------------
# Entry point
# ---------------------------------------------------------------------------
Write-Log "Disk cleanup started. WhatIf: $($WhatIfPreference)"

$CleanupTargets = @(
    @{ Path = $env:TEMP;                                                                          Label = "User Temp" },
    @{ Path = "$env:SystemRoot\Temp";                                                             Label = "System Temp" },
    @{ Path = "$env:SystemRoot\SoftwareDistribution\Download";                                   Label = "Windows Update Cache" },
    @{ Path = "$env:SystemRoot\SoftwareDistribution\DeliveryOptimization";                       Label = "Delivery Optimization Cache" },
    @{ Path = "$env:SystemRoot\Windows\WER";                                                     Label = "Windows Error Reporting" },
    @{ Path = "$env:USERPROFILE\AppData\Local\Microsoft\Windows\INetCache\ShaderCache";          Label = "DirectX Shader Cache" },
    @{ Path = "$env:USERPROFILE\AppData\Local\Microsoft\Windows\Explorer";                       Label = "Thumbnail Cache" }
)

foreach ($Target in $CleanupTargets) {
    Remove-DirectoryContents -Path $Target.Path -Label $Target.Label
}

Clear-RecycleBinSafe

# ---------------------------------------------------------------------------
# Summary
# ---------------------------------------------------------------------------
$FreedMB = [math]::Round($BytesFreed / 1MB, 2)

Write-Log "-----------------------------------------------------------"
Write-Log "Cleanup complete."
Write-Log "Files removed : $RemovedCount"
Write-Log "Failures      : $FailedCount"
Write-Log "Space freed   : $FreedMB MB"
if ($WhatIfPreference) {
    Write-Log "WhatIf mode was active — no changes were made." -Level Warn
}

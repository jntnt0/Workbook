# =============================================================================
# restartrdpclip.ps1
# Restart rdpclip.exe on a loop to restore RDP clipboard functionality
#
# Overview:
#   rdpclip.exe is the Windows process responsible for clipboard redirection
#   in Remote Desktop sessions. It occasionally stops responding, causing
#   copy-paste to fail between the RDP client and the remote host. This script
#   kills and restarts rdpclip.exe on a timed loop, restoring clipboard
#   functionality without requiring a full session disconnect.
#
# Usage:
#   Run as the RDP session user — does not require elevation.
#   Launch from Task Scheduler or manually from a PowerShell session.
#   Stop the loop with Ctrl+C or by terminating the process.
#
# Log output:
#   All events are written to $LogFilePath with timestamps.
#   Default log location: C:\Logs\rdpclip_restart.log
# =============================================================================

param (
    [string]$LogFilePath = "C:\Logs\rdpclip_restart.log",
    [int]$IntervalSeconds = 10
)

# ---------------------------------------------------------------------------
# Ensure the log directory exists before attempting to write
# ---------------------------------------------------------------------------
$LogDirectory = Split-Path $LogFilePath -Parent

if (-not (Test-Path -Path $LogDirectory)) {
    try {
        New-Item -ItemType Directory -Path $LogDirectory | Out-Null
    }
    catch {
        Write-Error "Failed to create log directory '$LogDirectory': $($_.Exception.Message)"
        exit 1
    }
}

# ---------------------------------------------------------------------------
# Helper: write a timestamped entry to the log file and to the console
# ---------------------------------------------------------------------------
function Write-Log {
    param ([string]$Message)
    $Entry = "$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss') - $Message"
    Add-Content -Path $LogFilePath -Value $Entry
    Write-Host $Entry
}

# ---------------------------------------------------------------------------
# Main loop — kill and restart rdpclip.exe on each iteration
# ---------------------------------------------------------------------------
Write-Log "rdpclip restart loop started. Interval: ${IntervalSeconds}s. Log: $LogFilePath"

while ($true) {
    try {
        $Process = Get-Process rdpclip -ErrorAction SilentlyContinue

        if ($Process) {
            Stop-Process -Name rdpclip -Force -ErrorAction Stop
            Write-Log "rdpclip.exe stopped (PID $($Process.Id))."
        }

        Start-Process -FilePath "rdpclip.exe" -ErrorAction Stop
        Write-Log "rdpclip.exe started successfully."
    }
    catch {
        Write-Log "ERROR: $($_.Exception.Message)"
    }

    Start-Sleep -Seconds $IntervalSeconds
}

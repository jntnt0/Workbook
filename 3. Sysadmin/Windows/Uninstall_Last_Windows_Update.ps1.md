# Uninstall the most recently installed Windows update
# Requires administrative privileges — will self-elevate if needed
# Requires PSWindowsUpdate module: Install-Module PSWindowsUpdate -Scope CurrentUser


# Define the progress bar function
function Show-ProgressBar {
    param(
        [int]$PercentComplete
    )
    $ProgressWidth = 50
    $CompletedWidth = [math]::Floor($ProgressWidth * ($PercentComplete / 100))
    $RemainingWidth = $ProgressWidth - $CompletedWidth
    $ProgressBar = "[" + "-" * $CompletedWidth + " " * $RemainingWidth + "]"
    Write-Progress -Activity "Uninstalling Update" -Status "Progress: $PercentComplete %" -PercentComplete $PercentComplete
}

# Check if the script is running with administrative privileges
if (-not ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")) {
    # If not, restart the script with administrative privileges
    Write-Host "Script is not running with administrative privileges. Restarting with elevated permissions..."
    Start-Process powershell.exe -ArgumentList "-NoProfile -ExecutionPolicy Bypass -File `"$PSCommandPath`" -Force" -Verb RunAs
    Exit
}

# Import the module
Import-Module PSWindowsUpdate

# Define the code to be run with elevated privileges
$elevatedCode = {
    # Import the module
    Import-Module PSWindowsUpdate

    # Uninstall last installed update
    $lastUpdate = Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 1
    if ($lastUpdate) {
        Write-Host "Uninstalling last update $($lastUpdate.HotFixID)..."
        $updateSession = New-Object -ComObject Microsoft.Update.Session
        $updateSearcher = $updateSession.CreateUpdateSearcher()
        $updates = $updateSearcher.Search("IsInstalled=1 AND IsHidden=0").Updates | Where-Object {$_.KBArticleIDs -match $lastUpdate.HotFixID}
        if ($updates) {
            $update = $updates[0]
            Write-Host "Found update $($update.Title)"
            $updateResult = $updateSession.CreateUpdateInstaller().Uninstall($update)
            if ($updateResult.ResultCode -eq 2) {
                Write-Host "Update uninstalled successfully."
            } else {
                Write-Host "Failed to uninstall update."
            }
        } else {
            Write-Host "Update not found."
        }
    } else {
        Write-Host "No updates found."
    }
}

# Check if the script is already running with elevated privileges
if (-not ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")) {
    # If not, run the code with elevated privileges
    Start-Process powershell.exe -ArgumentList "-NoProfile -ExecutionPolicy Bypass -Command `'$($elevatedCode | Out-String)'"
} else {
    # If already elevated, execute the code directly
    Invoke-Command -ScriptBlock $elevatedCode
}
